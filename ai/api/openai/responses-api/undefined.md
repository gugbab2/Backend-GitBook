# 시나리오 : 챗봇으로 과거 법무검토 이력 조회

### 시나리오 개요

**전제 조건**

* 자체 PostgreSQL + pgvector에 과거 법무검토 이력이 이미 저장되어 있음
* OpenAI의 Vector Store / file\_search는 사용하지 않음
* 임베딩 생성, 유사도 검색 모두 자체 인프라에서 수행
* Responses API는 **대화 오케스트레이션 + 도구 호출 엔진**으로만 활용

```
사용자: "A사와 SaaS 서비스 이용계약을 체결하려고 하는데,
        이전에 비슷한 SaaS 관련 법무검토 이력이 있을까?"
```

***

### 아키텍처

```
┌───────────────────────────────────────────────────────────────┐
│                     법무관리 솔루션 (프론트엔드)                    │
│                          채팅 UI                               │
└──────────────────────────┬────────────────────────────────────┘
                           │
                           ▼
┌───────────────────────────────────────────────────────────────┐
│                     백엔드 서버 (Node.js / Python)              │
│                                                               │
│  ┌──────────────┐  ┌───────────────┐  ┌────────────────────┐  │
│  │ Responses    │  │ Function Call │  │ Embedding          │  │
│  │ API 호출      │  │ 라우터        │  │ Service            │  │
│  │              │  │ (도구 실행)    │  │ (OpenAI Embedding) │  │
│  └──────┬───────┘  └───────┬───────┘  └────────┬───────────┘  │
│         │                  │                    │              │
└─────────┼──────────────────┼────────────────────┼──────────────┘
          │                  │                    │
          ▼                  ▼                    ▼
   ┌─────────────┐   ┌─────────────────────────────────────┐
   │ OpenAI      │   │         PostgreSQL + pgvector        │
   │ Responses   │   │                                     │
   │ API         │   │  ┌─────────────┐ ┌───────────────┐  │
   │             │   │  │ legal_      │ │ review_       │  │
   │ (LLM 추론 +  │   │  │ documents   │ │ history       │  │
   │  도구 호출   │   │  │ (벡터 검색)  │ │ (구조화 데이터) │  │
   │  판단만 담당)│   │  │             │ │               │  │
   │             │   │  │ - embedding │ │ - review_id   │  │
   │             │   │  │ - content   │ │ - status      │  │
   │             │   │  │ - metadata  │ │ - risk_level  │  │
   │             │   │  └─────────────┘ └───────────────┘  │
   └─────────────┘   └─────────────────────────────────────┘

  OpenAI에 보내는 것:          자체 인프라에 남는 것:
  - 사용자 질문 텍스트           - 계약서 원문
  - Function 스키마 정의         - 검토의견서
  - Function 실행 결과 (요약본)   - 벡터 임베딩
  - 대화 히스토리                - 법무검토 메타데이터
```

**핵심 차이**: OpenAI에는 법무 문서 원문이 절대 올라가지 않습니다. 모델에게는 function call의 결과(요약/발췌)만 전달되므로 데이터 주권을 유지할 수 있습니다.

***

### DB 스키마 (pgvector)

<pre class="language-sql"><code class="lang-sql">-- 1) 법무검토 이력 (구조화 데이터)
<strong>CREATE TABLE review_history (
</strong>    review_id       VARCHAR(20) PRIMARY KEY,  -- 'REV-2024-0315'
    contract_type   VARCHAR(50) NOT NULL,      -- 'SaaS', 'NDA', '라이선스', '용역'
    counterparty    VARCHAR(100) NOT NULL,     -- 상대방 회사명
    title           VARCHAR(200) NOT NULL,     -- 계약서 제목
    reviewer        VARCHAR(50),               -- 검토자
    review_date     DATE NOT NULL,
    status          VARCHAR(20),               -- '검토완료', '검토중', '반려', '승인'
    risk_level      VARCHAR(10),               -- 'high', 'medium', 'low'
    summary         TEXT,                      -- 검토 의견 요약
    key_issues      JSONB,                     -- 주요 리스크 포인트 배열
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- 2) 법무 문서 청크 (벡터 검색용)
--    계약서 원문 + 검토의견서를 청킹하여 임베딩 저장
CREATE TABLE legal_documents (
    id              SERIAL PRIMARY KEY,
    review_id       VARCHAR(20) REFERENCES review_history(review_id),
    doc_type        VARCHAR(30),               -- 'contract', 'review_opinion', 'memo'
    chunk_index     INTEGER,                   -- 청크 순서
    content         TEXT NOT NULL,             -- 원문 텍스트 청크
    embedding       vector(1536),              -- text-embedding-3-small 기준
    metadata        JSONB,                     -- 조항 번호, 섹션명 등
    created_at      TIMESTAMPTZ DEFAULT NOW()
);

-- 벡터 검색용 인덱스
CREATE INDEX idx_legal_docs_embedding
    ON legal_documents USING ivfflat (embedding vector_cosine_ops)
    WITH (lists = 100);

-- 메타데이터 필터용 인덱스
CREATE INDEX idx_review_history_type ON review_history(contract_type);
CREATE INDEX idx_review_history_date ON review_history(review_date);
CREATE INDEX idx_legal_docs_review_id ON legal_documents(review_id);
</code></pre>

***

### 구현 코드

#### 1. 임베딩 + pgvector 시맨틱 검색 함수 (백엔드)

```python
import asyncpg
from openai import OpenAI

client = OpenAI()

async def semantic_search_legal_docs(
    query: str,
    contract_type: str = None,
    top_k: int = 5,
    pool: asyncpg.Pool = None
) -> list[dict]:
    """
    사용자 질문을 임베딩하여 pgvector에서 유사 문서 청크를 검색합니다.
    """
    # 1) 쿼리 임베딩 생성
    embedding_response = client.embeddings.create(
        model="text-embedding-3-small",
        input=query
    )
    query_embedding = embedding_response.data[0].embedding

    # 2) pgvector 코사인 유사도 검색 + 메타데이터 필터
    sql = """
        SELECT
            ld.content,
            ld.doc_type,
            ld.metadata,
            rh.review_id,
            rh.contract_type,
            rh.counterparty,
            rh.title,
            rh.reviewer,
            rh.review_date,
            rh.risk_level,
            rh.summary,
            rh.key_issues,
            1 - (ld.embedding <=> $1::vector) AS similarity
        FROM legal_documents ld
        JOIN review_history rh ON ld.review_id = rh.review_id
        WHERE 1=1
    """
    params = [str(query_embedding)]
    param_idx = 2

    if contract_type:
        sql += f" AND rh.contract_type = ${param_idx}"
        params.append(contract_type)
        param_idx += 1

    sql += f"""
        ORDER BY ld.embedding <=> $1::vector
        LIMIT ${param_idx}
    """
    params.append(top_k)

    rows = await pool.fetch(sql, *params)

    return [
        {
            "review_id": row["review_id"],
            "counterparty": row["counterparty"],
            "title": row["title"],
            "contract_type": row["contract_type"],
            "reviewer": row["reviewer"],
            "review_date": str(row["review_date"]),
            "risk_level": row["risk_level"],
            "doc_type": row["doc_type"],
            "content_snippet": row["content"][:500],   # ← 원문 전체가 아닌 발췌만 전달
            "summary": row["summary"],
            "key_issues": row["key_issues"],
            "similarity": round(row["similarity"], 4)
        }
        for row in rows
    ]


async def search_review_metadata(
    contract_type: str = None,
    counterparty: str = None,
    date_from: str = None,
    date_to: str = None,
    status: str = None,
    pool: asyncpg.Pool = None
) -> list[dict]:
    """
    구조화된 조건으로 법무검토 이력을 조회합니다.
    """
    sql = "SELECT * FROM review_history WHERE 1=1"
    params = []
    param_idx = 1

    if contract_type:
        sql += f" AND contract_type = ${param_idx}"
        params.append(contract_type)
        param_idx += 1
    if counterparty:
        sql += f" AND counterparty ILIKE ${param_idx}"
        params.append(f"%{counterparty}%")
        param_idx += 1
    if date_from:
        sql += f" AND review_date >= ${param_idx}"
        params.append(date_from)
        param_idx += 1
    if date_to:
        sql += f" AND review_date <= ${param_idx}"
        params.append(date_to)
        param_idx += 1
    if status:
        sql += f" AND status = ${param_idx}"
        params.append(status)
        param_idx += 1

    sql += " ORDER BY review_date DESC LIMIT 10"

    rows = await pool.fetch(sql, *params)
    return [dict(row) for row in rows]


async def get_review_detail(review_id: str, pool: asyncpg.Pool) -> dict:
    """
    특정 검토 건의 상세 정보 + 관련 문서 청크를 조회합니다.
    """
    # 메타 정보
    review = await pool.fetchrow(
        "SELECT * FROM review_history WHERE review_id = $1",
        review_id
    )

    # 해당 검토 건의 문서 청크들 (검토의견서 위주)
    docs = await pool.fetch(
        """SELECT doc_type, content, metadata
           FROM legal_documents
           WHERE review_id = $1 AND doc_type = 'review_opinion'
           ORDER BY chunk_index
           LIMIT 5""",
        review_id
    )

    return {
        "review": dict(review) if review else None,
        "opinion_excerpts": [
            {"content": d["content"][:500], "metadata": d["metadata"]}
            for d in docs
        ]
    }
```

#### 2. Responses API 도구 정의 (모든 검색이 커스텀 function)

1. &#x20;**`type`**: 도구의 종류입니다. OpenAI Responses API에서 사용 가능한 도구 타입은 `"function"` (커스텀 함수), `"file_search"` (내장 문서 검색), `"web_search"` (내장 웹 검색), `"code_interpreter"` (내장 코드 실행) 등이 있습니다. 여기서는 pgvector 검색을 직접 구현하므로 `"function"`을 씁니다.
2. **`name`**: 함수의 고유 이름입니다. 모델이 이 도구를 호출하기로 판단하면, 응답의 `function_call.name`에 이 값이 그대로 들어옵니다. 백엔드에서 이 이름을 보고 어떤 함수를 실행할지 라우팅합니다.
3. **`description`**: **모델이 이 도구를 언제 사용할지 판단하는 핵심 근거**입니다. 모델은 사용자의 질문과 이 description을 비교해서 "이 도구를 호출해야 하나?"를 결정합니다. 그래서 단순한 설명이 아니라 **언제 쓰는지**, **뭘 할 수 있는지**를 구체적으로 적는 게 중요합니다.
4. &#x20;**`parameters`**: 이 함수가 받는 인자들의 스키마입니다. JSON Schema 형식을 따릅니다.
5. &#x20;**`type: "object"`**: parameters는 항상 `"object"` 타입입니다. 함수의 인자들을 key-value 형태로 받겠다는 의미입니다.
6. &#x20;**`properties`**: 각 인자의 정의입니다.
7. &#x20;**`required`**: 필수 인자 목록입니다. 여기 포함된 인자는 모델이 반드시 값을 생성해서 넘깁니다. 포함되지 않은 인자는 모델이 필요하다고 판단할 때만 넘기고, 아니면 생략합니다.

```python
tools = [
    # Tool 1: 시맨틱 검색 (pgvector)
    # → 사용자 질문과 유사한 과거 계약서/검토의견서 내용을 검색
    {
        "type": "function",
        "name": "search_similar_legal_docs",
        "description": (
            "과거 법무검토 문서(계약서 원문, 검토의견서)에서 "
            "사용자 질문과 의미적으로 유사한 내용을 검색합니다. "
            "계약 유형으로 필터링할 수 있습니다. "
            "새로운 계약과 비슷한 과거 사례를 찾을 때 사용하세요."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "query": {
                    "type": "string",
                    "description": "검색할 내용 (자연어). 예: 'SaaS 서비스 이용계약 데이터 이관 조항'"
                },
                "contract_type": {
                    "type": "string",
                    "description": "계약 유형 필터 (예: SaaS, NDA, 라이선스, 용역). 없으면 전체 검색."
                },
                "top_k": {
                    "type": "integer",
                    "description": "반환할 최대 결과 수 (기본값 5)"
                }
            },
            "required": ["query"]
        }
    },
    # Tool 2: 구조화된 검토 이력 조회
    # → 날짜, 상태, 상대방 등 메타데이터 기반 필터 검색
    {
        "type": "function",
        "name": "search_review_history",
        "description": (
            "법무검토 이력을 구조화된 조건(계약 유형, 상대방, 기간, 상태)으로 "
            "조회합니다. 특정 조건에 맞는 검토 건 목록이 필요할 때 사용하세요."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "contract_type": {
                    "type": "string",
                    "description": "계약 유형 (예: SaaS, NDA, 라이선스, 용역)"
                },
                "counterparty": {
                    "type": "string",
                    "description": "계약 상대방 회사명 (부분 일치 검색)"
                },
                "date_from": {
                    "type": "string",
                    "description": "검색 시작일 (YYYY-MM-DD)"
                },
                "date_to": {
                    "type": "string",
                    "description": "검색 종료일 (YYYY-MM-DD)"
                },
                "status": {
                    "type": "string",
                    "enum": ["검토완료", "검토중", "반려", "승인"],
                    "description": "검토 상태"
                }
            },
            "required": ["contract_type"]
        }
    },
    # Tool 3: 특정 검토 건 상세 조회
    {
        "type": "function",
        "name": "get_review_detail",
        "description": (
            "특정 법무검토 건의 상세 정보를 조회합니다. "
            "검토 코멘트, 수정 요청사항, 주요 리스크 포인트, "
            "검토의견서 발췌를 반환합니다."
        ),
        "parameters": {
            "type": "object",
            "properties": {
                "review_id": {
                    "type": "string",
                    "description": "법무검토 ID (예: REV-2024-0315)"
                }
            },
            "required": ["review_id"]
        }
    }
]
```

#### 3. 메인 대화 루프 (Function Call 처리)

```python
import json

LEGAL_CHATBOT_INSTRUCTIONS = """
당신은 법무관리 솔루션의 AI 어시스턴트입니다.

## 역할
사용자의 법무 관련 질문에 대해, 과거 법무검토 이력을 기반으로 답변합니다.

## 도구 사용 전략
- 유사 사례를 찾을 때는 search_similar_legal_docs(시맨틱 검색)와
  search_review_history(메타데이터 검색)를 모두 호출하여 교차 검증하세요.
- 특정 건의 세부 내용이 필요하면 get_review_detail을 호출하세요.
- 시맨틱 검색 쿼리는 구체적으로 작성하세요.
  (예: "SaaS 계약" → "SaaS 서비스 이용계약 데이터 이관 손해배상 SLA")

## 응답 원칙
1. 유사 케이스의 핵심 리스크 포인트와 검토 의견 요약을 함께 제시
2. 반복적으로 지적된 조항이 있다면 패턴을 분석
3. 항상 출처(검토 ID, 문서명, 검토자)를 명시
4. 법적 판단은 하지 않으며, 참고 정보임을 명확히 고지
"""


async def handle_chat_message(user_message: str, previous_response_id: str = None):
    """
    사용자 메시지를 받아 Responses API와 대화하고,
    Function Call이 있으면 실행 후 결과를 돌려보내는 루프
    """

    # --- 첫 요청 ---
    request_params = {
        "model": "gpt-4.1",
        "instructions": LEGAL_CHATBOT_INSTRUCTIONS,
        "tools": tools,
        "input": [{"role": "user", "content": user_message}],
        "store": True,
    }

    # 이전 대화가 있으면 컨텍스트 연결
    if previous_response_id:
        request_params["previous_response_id"] = previous_response_id

    response = client.responses.create(**request_params)

    # --- Function Call 처리 루프 ---
    # 모델이 더 이상 function을 호출하지 않을 때까지 반복
    while has_function_calls(response):
        tool_outputs = []

        for item in response.output:
            if item.type != "function_call":
                continue

            func_name = item.name
            func_args = json.loads(item.arguments)

            # 각 함수를 자체 인프라에서 실행
            if func_name == "search_similar_legal_docs":
                result = await semantic_search_legal_docs(
                    query=func_args["query"],
                    contract_type=func_args.get("contract_type"),
                    top_k=func_args.get("top_k", 5),
                    pool=db_pool
                )

            elif func_name == "search_review_history":
                result = await search_review_metadata(
                    contract_type=func_args.get("contract_type"),
                    counterparty=func_args.get("counterparty"),
                    date_from=func_args.get("date_from"),
                    date_to=func_args.get("date_to"),
                    status=func_args.get("status"),
                    pool=db_pool
                )

            elif func_name == "get_review_detail":
                result = await get_review_detail(
                    review_id=func_args["review_id"],
                    pool=db_pool
                )

            else:
                result = {"error": f"Unknown function: {func_name}"}

            tool_outputs.append({
                "type": "function_call_output",
                "call_id": item.call_id,
                "output": json.dumps(result, ensure_ascii=False, default=str)
            })

        # 모든 function 결과를 한 번에 전달
        response = client.responses.create(
            model="gpt-4.1",
            instructions=LEGAL_CHATBOT_INSTRUCTIONS,
            tools=tools,
            previous_response_id=response.id,
            input=tool_outputs,
        )

    # --- 최종 응답 반환 ---
    return {
        "message": response.output_text,
        "response_id": response.id  # 다음 대화에서 previous_response_id로 사용
    }


def has_function_calls(response) -> bool:
    """응답에 function_call이 포함되어 있는지 확인"""
    return any(item.type == "function_call" for item in response.output)
```

#### 4. API 엔드포인트 (FastAPI 예시)

```python
from fastapi import FastAPI, Request
from pydantic import BaseModel

app = FastAPI()

class ChatRequest(BaseModel):
    message: str
    conversation_id: str | None = None  # previous_response_id 매핑

class ChatResponse(BaseModel):
    message: str
    conversation_id: str

@app.post("/api/chat", response_model=ChatResponse)
async def chat_endpoint(req: ChatRequest):
    result = await handle_chat_message(
        user_message=req.message,
        previous_response_id=req.conversation_id
    )
    return ChatResponse(
        message=result["message"],
        conversation_id=result["response_id"]
    )
```

***

### 전체 흐름 (시퀀스)

```
사용자          프론트엔드        백엔드 서버          OpenAI API         pgvector DB
  │                │                │                   │                  │
  │ "SaaS 검토     │                │                   │                  │
  │  이력 있어?"    │                │                   │                  │
  │───────────────>│  POST /chat    │                   │                  │
  │                │───────────────>│                   │                  │
  │                │                │                   │                  │
  │                │                │  responses.create  │                  │
  │                │                │  (질문 + tools)    │                  │
  │                │                │──────────────────>│                  │
  │                │                │                   │                  │
  │                │                │                   │ 모델 판단:        │
  │                │                │                   │ "2개 함수를       │
  │                │                │                   │  호출해야겠다"     │
  │                │                │                   │                  │
  │                │                │  function_call x2  │                  │
  │                │                │  ① search_similar  │                  │
  │                │                │  ② search_history  │                  │
  │                │                │<──────────────────│                  │
  │                │                │                   │                  │
  │                │                │  ① 임베딩 생성     │                  │
  │                │                │──────────────────>│                  │
  │                │                │<──────────────────│                  │
  │                │                │                   │                  │
  │                │                │  ① 벡터 유사도 검색  │                  │
  │                │                │  ② 메타데이터 조회   │                  │
  │                │                │─────────────────────────────────────>│
  │                │                │<─────────────────────────────────────│
  │                │                │                   │                  │
  │                │                │  function_call     │                  │
  │                │                │  _output x2       │                  │
  │                │                │  + previous_id     │                  │
  │                │                │──────────────────>│                  │
  │                │                │                   │                  │
  │                │                │                   │ 종합 답변 생성     │
  │                │                │  최종 응답          │                  │
  │                │                │<──────────────────│                  │
  │                │                │                   │                  │
  │                │  ChatResponse   │                   │                  │
  │  답변 표시      │<───────────────│                   │                  │
  │<───────────────│                │                   │                  │
```

***

### OpenAI file\_search vs 자체 pgvector 비교

| 항목               | OpenAI file\_search         | 자체 pgvector                   |
| ---------------- | --------------------------- | ----------------------------- |
| 문서 저장 위치         | OpenAI 서버                   | 자체 DB 서버                      |
| 데이터 주권           | OpenAI에 원문 전송               | **원문이 외부로 나가지 않음**            |
| 임베딩 생성           | OpenAI 자동 처리                | 직접 Embedding API 호출           |
| 검색 커스터마이징        | 제한적 (max\_results, 필터)      | **자유로움** (하이브리드 검색, 가중치 조절 등) |
| 청킹 전략            | OpenAI 기본값                  | **직접 제어** (조항 단위, 섹션 단위 등)    |
| 비용               | 저장 $0.10/GB/일 + $2.50/1K호출  | DB 인프라 비용만                    |
| 구현 복잡도           | 낮음 (도구 설정만)                 | 높음 (임베딩 파이프라인 직접 구축)          |
| 검색 품질 튜닝         | 불가                          | **가능** (리랭킹, 하이브리드, 가중치)      |
| Responses API 연동 | tools에 type: "file\_search" | tools에 type: "function"       |

**법무 도메인에서 pgvector가 유리한 이유**:

1. 민감한 법무 문서가 외부 서버에 저장되지 않음
2. 계약서 조항 단위로 청킹하는 등 도메인 특화 전략 가능
3. 기존 법무관리 DB와 JOIN으로 풍부한 컨텍스트 제공 가능
4. 풀텍스트 검색 + 벡터 검색 하이브리드 구성 가능

***

### 검색 품질 향상 팁

#### 하이브리드 검색 (벡터 + 키워드)

법률 용어는 시맨틱 검색만으로는 부족할 수 있습니다. pgvector의 벡터 검색과 PostgreSQL의 풀텍스트 검색을 결합하면 정확도가 올라갑니다.

```sql
-- 벡터 유사도 + 키워드 매칭을 결합한 하이브리드 검색
WITH vector_results AS (
    SELECT
        ld.id,
        ld.review_id,
        ld.content,
        1 - (ld.embedding <=> $1::vector) AS vector_score
    FROM legal_documents ld
    JOIN review_history rh ON ld.review_id = rh.review_id
    WHERE rh.contract_type = $2
    ORDER BY ld.embedding <=> $1::vector
    LIMIT 20
),
keyword_results AS (
    SELECT
        ld.id,
        ts_rank(to_tsvector('simple', ld.content), plainto_tsquery('simple', $3)) AS keyword_score
    FROM legal_documents ld
    WHERE to_tsvector('simple', ld.content) @@ plainto_tsquery('simple', $3)
)
SELECT
    vr.*,
    -- 가중 결합: 벡터 70% + 키워드 30%
    (vr.vector_score * 0.7 + COALESCE(kr.keyword_score, 0) * 0.3) AS combined_score
FROM vector_results vr
LEFT JOIN keyword_results kr ON vr.id = kr.id
ORDER BY combined_score DESC
LIMIT 5;
```

#### 도메인 특화 청킹

```python
def chunk_legal_document(document_text: str) -> list[dict]:
    """
    계약서를 조항 단위로 청킹합니다.
    일반적인 고정 크기 청킹보다 법률 문서에 적합합니다.
    """
    import re

    # 조항 번호 패턴: 제1조, 제2조 ... 또는 1., 2. ...
    article_pattern = r'(제\d+조[의]?\d*\s*[\(（].*?[\)）]|제\d+조|^\d+\.)'
    chunks = re.split(article_pattern, document_text, flags=re.MULTILINE)

    result = []
    current_article = ""

    for i, chunk in enumerate(chunks):
        if re.match(article_pattern, chunk):
            current_article = chunk.strip()
        elif chunk.strip():
            result.append({
                "article_title": current_article,
                "content": f"{current_article}\n{chunk.strip()}",
                "chunk_index": len(result)
            })

    return result
```

***

### 보안 관점 정리

```
┌─────────────────────────────────────────────────────────┐
│               OpenAI로 전송되는 데이터                      │
│                                                         │
│  ✅ 사용자의 자연어 질문                                    │
│  ✅ Function 스키마 정의 (도구 설명)                        │
│  ✅ Function 실행 결과 (content_snippet: 500자 발췌)       │
│  ✅ 시스템 프롬프트                                        │
│                                                         │
│  ❌ 계약서 원문 전체 → 전송되지 않음                         │
│  ❌ 벡터 임베딩 → 전송되지 않음                             │
│  ❌ DB 스키마/접속정보 → 전송되지 않음                       │
└─────────────────────────────────────────────────────────┘
```

function\_call\_output에 실어 보내는 데이터의 양을 제어하면 OpenAI에 전달되는 민감 정보를 최소화할 수 있습니다. 위 코드에서 `content[:500]` 으로 발췌만 보내는 이유가 이것입니다.
