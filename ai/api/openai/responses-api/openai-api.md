# OpenAI API 진화의 전체 흐름

### 1세대: Completions API (2020\~)

OpenAI가 처음 공개한 API입니다. 구조가 매우 단순했습니다.

* **입력**: 자유 형식의 텍스트 문자열 하나 (`prompt`)
* **출력**: 모델이 이어서 생성한 텍스트
* "대화"라는 개념 자체가 없고, 단순히 텍스트를 넣으면 텍스트가 나오는 구조였습니다.

```python
response = client.completions.create(
    model="gpt-3.5-turbo-instruct",
    prompt="Write a tagline for an ice cream shop."
)
```

2023년 7월에 마지막 업데이트를 받았고, 현재는 레거시로 분류 [Openai](https://developers.openai.com/api/docs/guides/completions/)됩니다. 대화형 인터페이스가 필요하면 개발자가 직접 프롬프트 안에 `User:`, `Assistant:` 같은 구분을 텍스트로 넣어야 했기 때문에 불편했습니다.

***

### 2세대: Chat Completions API (2023. 3\~)

GPT-3.5-turbo, GPT-4 출시와 함께 등장한 API로, **대화 구조를 네이티브로 지원**한 것이 핵심입니다.

* **입력**: `messages` 배열 (role: system/user/assistant)
* **출력**: assistant의 다음 메시지
* system 메시지로 모델의 행동 방식을 지정할 수 있게 되었습니다

```python
response = client.chat.completions.create(
    model="gpt-4o",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ]
)
```

**시간이 지나면서 추가된 주요 기능들:**

* **Function Calling** (2023.6) → 이후 **Tools**로 발전: 모델이 외부 함수를 호출할 수 있게 됨
* **JSON Mode / Structured Outputs**: 모델 출력을 JSON 스키마로 제약
* **Vision**: 이미지 입력 지원 (GPT-4V)
* **Audio**: 음성 입출력 (GPT-4o-audio)
* **Streaming**: 토큰 단위 실시간 스트리밍

**핵심 한계**: API 자체는 **stateless**입니다. 매 요청마다 전체 대화 히스토리를 개발자가 직접 관리하고 다시 보내야 했습니다. 도구 호출 시에도 "모델이 도구를 호출 → 개발자가 결과를 받아서 다시 API에 전달 → 모델이 최종 응답"이라는 루프를 개발자가 직접 구현해야 했습니다.

이 API는 업계 표준이 되어서, 다른 LLM 제공자들(Anthropic, Google 등)도 유사한 인터페이스를 채택 [Medium](https://medium.com/@praveenkumarsingh/openais-api-evolution-responses-api-vs-chat-completions-api-0463d73ce631)했고, OpenAI도 무기한 지원을 약속했습니다.

***

### 2.5세대: Assistants API (2023. 11\~ → 2026. 8 지원 종료 예정)

Chat Completions의 한계를 보완하기 위해 만들어진 **상위 레벨 API**입니다. "AI 비서를 쉽게 만들자"라는 목표로, 개발자가 직접 관리하던 것들을 OpenAI 서버에서 처리해 주는 구조였습니다.

**핵심 개념들:**

* **Assistant**: 모델 + 지시사항 + 도구를 묶은 설정 객체
* **Thread**: 서버 측에서 관리되는 대화 세션 (개발자가 히스토리를 직접 관리할 필요 없음)
* **Run**: Thread에서 Assistant를 실행하는 단위
* **Built-in Tools**: Code Interpreter, File Search(RAG) 등 서버 측 도구 내장

**왜 문제가 있었나:**

* 매 메시지마다 전체 스레드 내용(문서 포함)을 재처리하면서 토큰 비용이 급격히 올라가는 문제 [Eesel AI](https://www.eesel.ai/blog/openai-assistants-api)가 있었습니다.
* 초기에는 실시간 스트리밍을 지원하지 않아 폴링 방식으로 응답을 확인해야 했습니다.
* 끝까지 베타 상태를 벗어나지 못했습니다. [Syntackle](https://syntackle.com/blog/openai-assistants-to-responses-api/)
* 2025년 8월에 공식적으로 deprecated되었고, 2026년 8월 26일에 완전히 종료될 예정 [Openai](https://developers.openai.com/api/docs/deprecations/)입니다.

***

### 3세대: Responses API (2025. 3\~)

Chat Completions와 Assistants API의 교훈을 합쳐 만든 **차세대 통합 API**입니다.

#### Chat Completions와 비교한 핵심 차이

**① 에이전틱 루프가 내장됨**

모델이 웹 검색, 파일 검색, 코드 인터프리터, 컴퓨터 사용, MCP 서버 등 여러 도구를 하나의 API 요청 안에서 자동으로 호출하고 처리 [OpenAI](https://platform.openai.com/docs/guides/migrate-to-responses)할 수 있습니다. Chat Completions에서는 개발자가 이 루프를 직접 짜야 했습니다.

**② Stateful 옵션**

`previous_response_id`를 전달하면 서버에서 대화 상태를 관리합니다. 매번 전체 히스토리를 보낼 필요가 없어집니다. 또한 Conversations API와 연동하여 Assistants의 Thread 같은 영속적 대화도 가능합니다.

**③ Reasoning 모델과의 통합**

OpenAI는 reasoning 모델의 사고 과정(chain-of-thought)을 공개하지 않는데, Responses API의 stateful 구조를 통해 서버 측에서 이 reasoning trace를 대화 컨텍스트에 자동으로 포함 [Sean Goedecke](https://www.seangoedecke.com/responses-api/)시킬 수 있습니다. Chat Completions에서는 이것이 불가능해서 reasoning 모델의 성능이 제한되었습니다.

**④ 성능·비용 이점**

내부 테스트에서 캐시 활용률이 40\~80% 개선되어 비용이 낮아지고, SWE-bench에서 같은 프롬프트로 3% 성능 향상 [OpenAI](https://platform.openai.com/docs/guides/migrate-to-responses)이 있었다고 합니다.

**⑤ 유연한 입력**

문자열 하나를 바로 넘길 수도 있고, messages 배열을 넘길 수도 있으며, `instructions`로 시스템 레벨 지시를 별도 분리할 수도 있습니다.

### Responses API의 한계점

**① 컨텍스트 윈도우 제약은 여전함**

`previous_response_id`는 개발자가 히스토리를 직접 관리하는 번거로움을 덜어줄 뿐, 내부적으로는 이전 대화 전체를 모델에 넣는 것과 동일합니다. 대화가 길어지면 모델의 컨텍스트 윈도우(GPT-4o: 128K, GPT-5.2: 400K)를 초과하여 `context_length_exceeded` 에러가 발생합니다. 이전 input 토큰도 전부 과금되므로, 대화가 길어질수록 비용이 선형으로 증가합니다. OpenAI는 이를 해결하기 위해 Compaction API(`/responses/compact`)를 제공하지만, 이는 손실 압축(loss-aware compression)이므로 오래된 대화의 세부 내용이 유실될 수 있습니다.

**② 커스텀 Function Call은 자동 실행되지 않음**

"에이전틱 루프 내장"은 내장 도구(`web_search`, `file_search`, `code_interpreter`)에만 해당됩니다. 개발자가 정의한 커스텀 function은 모델이 직접 실행할 수 없으므로, 모델이 function\_call을 반환하면 개발자가 실행하고 결과를 돌려주는 핑퐁 루프를 여전히 직접 구현해야 합니다. 자체 DB를 사용하는 대부분의 실무 시나리오에서는 이 루프가 필수입니다.

**③ Vendor Lock-in 심화**

`previous_response_id`, Compaction, 암호화된 reasoning trace 등 Responses API 고유 기능에 의존할수록 다른 LLM 제공자(Anthropic, Google 등)로의 전환이 어려워집니다. Chat Completions는 업계 표준이 되어 대부분의 제공자가 호환 인터페이스를 지원했지만, Responses API의 stateful 구조와 내장 도구 체계는 OpenAI 고유의 설계입니다.

**④ Stateful 구조의 디버깅 어려움**

`previous_response_id`로 서버에 상태를 위임하면, 현재 모델에 어떤 컨텍스트가 들어가 있는지를 개발자가 정확히 파악하기 어렵습니다. 특히 Compaction이 적용된 후에는 압축된 내용이 암호화되어 불투명하므로, 예상치 못한 응답이 나왔을 때 원인을 추적하기 까다롭습니다.

**⑤ 데이터 보안 고려사항**

`store: true`(기본값)로 사용하면 대화 데이터가 OpenAI 서버에 저장됩니다. 법무, 의료 등 민감 데이터를 다루는 경우 `store: false` 설정이나 Zero Data Retention(ZDR) 정책을 적용해야 하는데, 이 경우 `previous_response_id` 기반 상태 관리를 사용할 수 없어 Responses API의 핵심 편의성이 제한됩니다.

#### 최근 추가된 기능들 (2025 후반\~2026)

Skills 지원, Hosted Shell 도구, WebSocket 모드, 서버 측 compaction, phase 라벨링(commentary/final\_answer 구분) [OpenAI](https://platform.openai.com/docs/changelog) 등이 계속 추가되고 있습니다.

***

### 전체 흐름 요약

| 시기      | API              | 핵심 특징                       | 상태             |
| ------- | ---------------- | --------------------------- | -------------- |
| 2020    | Completions      | 텍스트 in → 텍스트 out            | 레거시            |
| 2023.3  | Chat Completions | 대화 구조, function calling     | 무기한 지원 (업계 표준) |
| 2023.11 | Assistants       | 서버 측 상태 관리, 내장 도구           | 2026.8 종료 예정   |
| 2025.3  | **Responses**    | 에이전틱 루프, stateful, 내장 도구 통합 | **현재 권장**      |

