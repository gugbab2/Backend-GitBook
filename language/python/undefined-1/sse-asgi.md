# SSE 스트리밍은 왜 ASGI를 요구하는가 — 동기/비동기·블로킹·코루틴·이벤트 루프의 실체

> LLM 채팅(SSE 스트리밍)을 Django 서버 두 대(게이트웨이 서버 → AI 서버)로 릴레이하는 실무 구조를 파고들며 정리한 학습 노트. 모든 실측은 Django 5.2 + uvicorn 환경에서 직접 수행한 결과다.

***

## 0. 문제 설정

```
브라우저 ──SSE──▶ 게이트웨이 서버(Django) ──SSE──▶ AI 서버(Django) ──▶ LLM API
            (인증·권한·필터링 담당 중계자)      (LLM 호출·SSE 생산자)
```

* LLM 응답은 수십 초\~수 분짜리 **long-lived 연결**이고, 토큰이 생기는 대로 브라우저에 찍혀야 한다(타이핑 효과).
* 두 서버 모두 **메시지 전송 API에 한해 ASGI**로 서빙한다. 게이트웨이는 본체(uWSGI)와 별도로 uvicorn 컨테이너를 하나 더 띄워 nginx가 스트리밍 경로만 그쪽으로 라우팅하고, AI 서버는 전체가 gunicorn + UvicornWorker다.
* 이 노트는 "왜 그래야 하는가"를 밑바닥(코루틴·이벤트 루프·블로킹)까지 파고든 기록이다.

***

## 1. Django 스트리밍의 2×2 매트릭스 (실측)

같은 generator(1초에 1청크 × 3개)를 4가지 조합으로 서빙하고 **바이트 도착 시각**을 raw socket으로 실측했다.

| # | 조합                     | chunk-0    | chunk-1    | chunk-2    | 판정    |
| - | ---------------------- | ---------- | ---------- | ---------- | ----- |
| 1 | WSGI + sync generator  | **t=1.0s** | **t=2.0s** | **t=3.0s** | ✅ 라이브 |
| 2 | WSGI + async generator | t=3.0s     | t=3.0s     | t=3.0s     | ❌ 한방에 |
| 3 | ASGI + sync generator  | t=3.0s     | t=3.0s     | t=3.0s     | ❌ 한방에 |
| 4 | ASGI + async generator | t=1.0s     | t=2.0s     | t=3.0s     | ✅ 라이브 |

**규칙: 서버가 응답을 반복하는 문법과 iterator 종류가 일치하면 라이브, 어긋나면 전량 버퍼링.**

* WSGI 서버의 전송부는 `for chunk in response: write(chunk)` — sync generator를 직접 반복 가능(조합 1 라이브).
* ASGI 서버는 `async for chunk in response: await send(chunk)` — async iterator를 직접 반복 가능(조합 4 라이브).
* 어긋난 조합은 Django의 fallback이 개입한다 (`django/http/response.py`):

```python
# ASGI + sync generator → __aiter__ fallback
for part in await sync_to_async(list)(self.streaming_content):  # ★ list() = 전량 소비
    yield part

# WSGI + async generator → __iter__ fallback
return map(self.make_bytes, iter(async_to_sync(to_list)(self._iterator)))  # 역시 전량 소비
```

`list(gen)`은 generator가 **소진될 때까지** 돌리므로, LLM이 2분 걸리면 클라이언트는 2분간 0바이트를 받다가 전체를 한 번에 받는다. **에러는 어디에도 없다** — 경고 로그 한 줄뿐. 이 계열 문제는 예외 모니터링에 절대 잡히지 않고, UX만 조용히 죽는다.

### 어긋난 조합을 살리는 수동 브릿지 (어댑터 패턴)

sync generator(예: `requests`의 `iter_lines()` 릴레이)를 ASGI에서 라이브로 만들려면, Django fallback의 `list`(전부) 대신 `next`(한 조각)를 스레드로 위임하는 어댑터를 직접 짠다:

```python
_STREAM_END = object()  # StopIteration은 async 경계에서 변질되므로(PEP 479) 종료를 '값'으로 표현

def wrap_sync_stream_for_asgi(sync_generator):
    async def stream():
        try:
            while True:
                part = await sync_to_async(next, thread_sensitive=True)(sync_generator, _STREAM_END)
                if part is _STREAM_END:
                    break
                yield part          # 꺼낸 즉시 전송 → 라이브 복원
        finally:
            await sync_to_async(sync_generator.close, thread_sensitive=True)()  # 클라이언트 단절 시 업스트림 정리
    return stream()
```

Django fallback과의 차이는 단 하나 — **스레드에 시키는 일의 단위**(`list` 전부 vs `next` 한 조각)다.

***

## 2. 판단의 두 축 — 기능(라이브)과 자원(동시성)은 별개다

조합 1(WSGI+sync)이 라이브였다는 사실이 중요하다. **"sync 서빙 = 버퍼링"이 아니다.** 그런데도 WSGI가 배제되는 이유는 자원 모델이다:

```
기능 축 (라이브냐):     매칭만 맞으면 됨 → WSGI+sync ✅ / ASGI+async ✅
자원 축 (몇 개 버티냐): WSGI = 스트림 1건이 워커 프로세스 1개를 몇 분간 독점 ❌
                       (uWSGI harakiri=120 같은 타임아웃이 긴 스트림을 강제 절단하기도)
──────────────────────────────────────────────
두 축을 동시에 만족하는 칸 = ASGI + async iterator, 단 하나
```

* 체인의 동시 처리 한도는 **min(각 홉의 한도)** — 한쪽만 ASGI면 병목이 이동할 뿐 천장은 그대로다. 그래서 게이트웨이·AI 서버가 **각자 독립적으로** 같은 결론(ASGI)에 수렴한다.
* 반면 라이브 여부는 **AND(소스부터 모든 홉)** — 각 홉이 자기 프로세스 안에서 매칭만 맞추면 되므로, 서버끼리 서빙 방식이 섞여도 기능은 성립한다.
* "ASGI로 간다"는 결정 하나에 매칭 규칙이 "iterator도 async로"를 자동으로 붙여준다. 둘은 두 번의 선택이 아니라 한 세트다.

***

## 3. generator와 코루틴은 같은 엔진이다

이벤트 루프 없이 코루틴을 **손으로** 굴려보면 증명된다:

```python
class Ticket:
    def __await__(self):
        wake_value = yield "교환권"   # ← await의 바닥에는 지금도 yield가 있다
        return wake_value

async def coroutine_worker():
    received = await Ticket()
    return f"작업 완료 (받은 것: {received})"

coro = coroutine_worker()
handed_out = coro.send(None)      # generator와 똑같이 send로 굴러간다 → "교환권"이 밖으로
coro.send("chunk-0 도착")          # 이벤트 루프가 '깨울 때' 하는 일이 정확히 이것
# → StopIteration.value == "작업 완료 (받은 것: chunk-0 도착)"  ← 완료 신호도 generator와 동일
```

역사도 그렇다: PEP 255(generator) → PEP 342(send) → PEP 380(yield from) → 초기 asyncio(@coroutine + yield from) → PEP 492(async/await 전용 문법). **코루틴 = 일시정지 가능한 함수 프레임**이라는 엔진은 동일하고, 실체는 "지역변수 + 재개 위치"를 담아 힙에 보존된 몇 KB짜리 객체다. 스레드(MB 스택 + OS 스케줄링)보다 자릿수로 싸다.

### 그런데 왜 문법을 갈랐나 — 멈춤의 방향이 반대라서

|           | `yield`                                       | `await`                                 |
| --------- | --------------------------------------------- | --------------------------------------- |
| 멈추는 이유    | **생산** — "내가 만든 제품을 가져가라"                     | **수급** — "내가 쓸 재료가 올 때까지 기다린다"          |
| 밖으로 나가는 것 | 소비자가 원하는 **목적물**(청크)                          | 스케줄러용 **사무 서류**(Future = "완료되면 깨워줘" 티켓) |
| 깨우는 주체    | **소비자** — `next()`/`send()`/`anext()`를 부르는 코드 | **이벤트 루프** — 예약된 Future가 완료됐을 때         |
| 사는 것      | 응답성·증분 전달 (기능 축)                              | **동시성** (자원 축)                          |

* yield에는 스케줄러가 없다. 멈춘 generator는 누군가 `next()`를 부르기 전까지 **영원히 방치**된다. 이 pull 구조가 자연스러운 backpressure가 된다(소비자가 느리면 생산도 자동으로 느려짐).
* **yield는 응답성을 사고 await는 동시성을 산다.** 조합 1이 그 반례 실험 — yield만 있는 스트리밍은 라이브였지만 서버는 1명 전용이었다.
* `async def` + `yield` + `await` = **async generator** — "받아서(await), 넘긴다(yield)"의 교대라서 스트리밍 릴레이의 자연스러운 형태다.

***

## 4. 이벤트 루프 = 요청서 보관함 + 조건 감시 + send(), 세 부품이 전부다

40줄로 직접 만들어서 확인했다:

```python
import heapq, time

def mini_sleep(seconds):
    yield ("wake_me_at", time.monotonic() + seconds)   # '요청서'를 밖으로 내밀고 정지

def chat_stream(name):
    for chunk_index in range(3):
        yield from mini_sleep(1)                        # await asyncio.sleep(1)에 해당
        print(f"[{name}] chunk-{chunk_index} 전송")

def mini_event_loop(coroutines):
    schedule = []                                       # (깨울 시각, 일련번호, 코루틴)
    for serial, coroutine in enumerate(coroutines):
        heapq.heappush(schedule, (time.monotonic(), serial, coroutine))
    while schedule:
        wake_at, serial, coroutine = heapq.heappop(schedule)
        idle = wake_at - time.monotonic()
        if idle > 0:
            time.sleep(idle)                            # 실제 asyncio는 epoll.wait(timeout)
        try:
            kind, when = coroutine.send(None)           # 다음 정지점까지 실행 = '깨움'
        except StopIteration:
            continue
        heapq.heappush(schedule, (when, serial, coroutine))

mini_event_loop([chat_stream("채팅A"), chat_stream("채팅B"), chat_stream("채팅C")])
```

실행 결과: **채팅 3개가 스레드 1개로 t=1.0/2.0/3.0s에 나란히 진행, 전체 3초 종료(직렬이면 9초).**

* CPU가 실제 일한 시간은 밀리초 단위. 9초→3초의 이득은 전부 \*\*"세 개의 대기를 한 시간축에 포갠 것"\*\*에서 나왔다. **동시성 = 대기의 겹침.** 그래서 async는 I/O 대기가 지배하는 워크로드에서만 빛난다.
* 실제 asyncio와의 대응: 요청서 튜플 → **Future**, `time.sleep(idle)` → **`selector.select(timeout)` = epoll**(타이머+소켓을 한 시스템 콜로 동시 감시), `send(None)` → Task의 한 step.

### 루프는 "따로 스레드"가 아니다 — 코루틴과 같은 스레드다 (실측)

```
[메인] 프로그램 메인 스레드   : ...4832
[루프] 이벤트 루프 스레드     : ...4832   ← 같음
[코루틴A] 본문 실행 스레드    : ...4832   ← 같음
[코루틴B] 본문 실행 스레드    : ...4832   ← 같음
[A/B] sync_to_async 작업 스레드: ...9680   ← 여기만 다름
```

`send()`는 평범한 함수 호출이라 **코루틴 본문은 루프의 콜스택 위에서, 루프와 같은 스레드로** 돈다. 감독(루프 스레드)이 선수들(코루틴 스레드)을 지휘하는 그림이 아니라, **한 무대를 번갈아 쓰는** 그림이다. 여기서 두 규칙이 연역된다:

1. **루프 위 블로킹 금지** — 코루틴이 `time.sleep(30)`하면 `send()`가 30초간 반환을 안 하므로 루프의 while문 자체가 정지 = 그 워커의 모든 스트림 동반 정지. (CPU 무거운 작업도 동일)
2. **협력적 멀티태스킹** — 무대가 하나라서 각자 짧게 쓰고 빨리 내려와(yield/await)야 굴러간다.

***

## 5. 블로킹의 물리학과 sync\_to\_async

### 블로킹 recv는 스레드를 커널에 인질로 맡긴다

```python
data = socket.recv(4096)   # 커널: "데이터 올 때까지 이 스레드를 재운다"
```

블로킹 소켓 read는 OS 수준에서 **잠들어 기다려주는 스레드 1개를 반드시 요구**한다. `requests`·동기 LLM SDK를 쓰는 한 이 물리학은 못 피한다.

`sync_to_async`는 블로킹을 없애는 게 아니라 **"누구의 스레드가 잠들 것인가"를 바꾸는 장치**다 — 이벤트 루프 스레드(전원 마비)에서 일회용 워커 스레드(그 요청만 대기)로 격리 이주.

### thread\_sensitive=True와 요청당 전용 스레드 (실측)

Django의 ASGI 핸들러는 요청마다 `ThreadSensitiveContext`를 열어, 그 요청의 `thread_sensitive=True` 호출을 **전용 스레드 1개에 고정**한다(DB 커넥션이 스레드-로컬이라 흩어지면 깨짐). 동시 스트림 3개로 실측하면:

```
시작: 활성 스레드 1 (루프뿐)
스트림 3개 진행 중: 활성 스레드 4 (루프 1 + 요청별 전용 recv 스레드 3)
요청A recv 스레드: 64288, 64288, 64288   ← 매 청크 같은 스레드
요청B recv 스레드: 55200, 55200, 55200
요청C recv 스레드: 46112, 46112, 46112
```

스트림의 삶은 99%가 "다음 청크 대기" = recv 안이므로, 아무 순간의 스냅샷에서나 **스트림 수 ≈ 잠든 워커 스레드 수**다. 이것이 sync 브릿지 구조의 동시성 천장(= 스레드 수)의 실체.

### epoll — 순수 async가 스레드를 안 쓰는 이유

```
블로킹 방식:   소켓 25개 = 잠든 스레드 25개 (각자 자기 소켓 앞에서 대기)
epoll 방식:    소켓 25개 = 시스템 콜 1개 (루프가 일괄 감시, 준비된 코루틴만 깨움)
              → 소켓당 잠드는 스레드 0개
```

httpx AsyncClient·async LLM SDK가 이 방식이다. "기다림에 스레드가 드느냐"가 블로킹/넌블로킹의 최종 구분이고, epoll은 N개의 기다림을 시스템 콜 1개로 합쳐 그 비용을 0으로 만든다.

***

## 6. 자원 지도 — 워커 4개에 100개 요청이 오면

"워커"는 계층마다 다른 것을 가리킨다. **프로세스 > 스레드 > 코루틴** 3층으로 고정하고 셈하면:

```
                100개의 스트리밍 요청
                        │ ① 소켓 수준에서 프로세스로 분배
    ┌──────────┬──────────┬──────────┐
프로세스1     프로세스2    프로세스3    프로세스4      ← "워커 4개" = 프로세스 4개
(~25 요청)
    ├─ 이벤트 루프 스레드 1개 ── 코루틴 25개가 교대 사용
    └─ sync 워커 스레드 ~25개 ── 각 요청의 블로킹 recv가 하나씩 점유
```

| 모델                   | 동시 스트림 100개의 비용        |
| -------------------- | ---------------------- |
| WSGI (uWSGI)         | 프로세스 100개 필요 — 사실상 불가능 |
| ASGI + sync 브릿지 (현재) | 프로세스 4 + 스레드 \~104     |
| 순수 async (httpx 등)   | 프로세스 4 + 스레드 4         |

* 프로세스를 2\~4개 두는 이유는 동시성이 아니라 **CPU 코어**(GIL 때문에 코어 활용은 프로세스 분리로). I/O 동시성은 코루틴이, CPU 병렬성은 프로세스가 담당하는 분업.
* "async 코드인데 WSGI로 서빙"은 최악의 칸이다: 코루틴은 임시 루프에서 **돌아는 가지만**(에러 없음), 공유 루프가 없어 동시성 0 + async 계약이 없어 라이브 0. **"돌아간다"와 "값어치를 낸다"는 별개의 검증 항목이다.**

***

## 7. 전체 파이프라인 — 토큰 하나의 여정

증분성은 소스에서 시작한다(**0번 홉**). LLM 추론은 autoregressive라 토큰을 하나씩 만드는 게 천연 상태고, `streaming=True`는 그 리듬을 노출하는 것이다(오히려 non-streaming이 서버측 버퍼링이라는 인공물).

```
[홉0] LLM 추론 — 토큰 단위 생성 (streaming=True)
[홉1] AI 서버(ASGI): sync LLM generator → sync_to_async(next) 브릿지 → async generator → 즉시 send
[홉2] 게이트웨이(ASGI): requests.iter_lines(워커 스레드 recv) → 어댑터로 async 변환 → 즉시 send
[홉3] nginx: X-Accel-Buffering: no (proxy_buffering 기본값이 버퍼링이므로 응답 헤더로 해제)
[홉4] 브라우저: 토큰 단위 렌더링
```

* **어느 한 홉이라도 버퍼링하면** 증상은 동일하다: 침묵 → 한방 출력, 에러 없음. 이 표가 곧 디버깅 체크리스트다(라우팅 확인 → iterator 매칭 확인 → 소스 streaming 확인 → nginx 헤더 확인).
* 깨움의 사슬은 **프로세스마다 닫혀 있다.** yield를 깨우는 것은 항상 같은 프로세스의 소비자, await를 깨우는 것은 항상 같은 프로세스의 루프. 프로세스 사이에는 바이트만 흐른다 — 한 프로세스의 send가 다른 프로세스의 recv를 풀어줄 뿐.
* 모든 이벤트가 증분일 수는 없다. 답변 chunk는 증분, "완성본에 대한 판단"(메타데이터·추천 등)은 본질적으로 종결형이라 스트림 후반에 온다. 좋은 SSE 프로토콜은 이 둘을 이벤트 타입으로 구분한다.

### sync↔async 경계의 통행 규칙 — "값만 통과"

제어 흐름·암묵 상태는 경계를 못 넘으므로 수동 운반해야 한다:

* `StopIteration` → async 경계에서 RuntimeError로 변질(PEP 479) → 종료를 **센티널 값**(`_STREAM_END = object()`)으로 표현
* 코루틴에서 Django ORM 직접 호출 → `SynchronousOnlyOperation` → `sync_to_async`로 래핑
* contextvars → Task 생성 시점에 복사되므로, 긴 스트림에 컨텍스트(예: 로깅용 테넌트 정보)를 유지하려면 같은 Context 객체를 명시적으로 재사용

***

## 8. 암기 카드

1. 판별 질문은 하나: **"기다리는 동안 이 스레드는 뭘 하나"** — 잡혀 있으면 블로킹.
2. generator와 코루틴은 **같은 엔진** — 다른 건 멈춤의 방향(생산 vs 수급)과 깨우는 자(소비자 vs 루프).
3. **yield는 응답성을 사고, await는 동시성을 산다** — 스트리밍 서버엔 둘 다 필요해서 async generator.
4. `await` = 루프에 양보, `sync_to_async` = 블로킹의 격리 이주, `asyncio.wait(timeout)` = 다중 대기.
5. 이벤트 루프 = **한 스레드를 통째로 차지한 while문** (요청서 보관함 + epoll + send). 코루틴 본문도 그 스레드에서 돈다 → 루프 위 블로킹 금지는 구조의 귀결.
6. 매칭도 깨움도 **프로세스 내부 문제** — 홉 사이엔 바이트만. 라이브는 **AND**(소스부터 전 홉), 용량은 **min()**(최약 홉).
7. 블로킹 recv는 **스레드를 커널에 인질로** 잡는다. sync 브릿지 구조의 천장 = 스레드 수, 순수 async(epoll) = 인질 0.

***

## 연관 노트

* [Python 비동기 프로그래밍 - 왜 필요하고 어떻게 동작하는가](python-1.md)
* [웹 서버는 어떻게 Python 코드를 실행하게 되었는가 — CGI, WSGI, ASGI의 발전과 실전 배포](python-cgi-wsgi-asgi.md)
* [WSGI, ASGI 핸들러 구현체 비교](wsgi-asgi.md)
* [Python은 왜 한 번에 하나만 실행할까? - GIL 이해하기](python-gil.md)
