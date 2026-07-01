# 왜 웹 프레임워크(Python 진영)는 비동기에 집착하게 되었을까?

동기와 비동기 코드를 기반으로 살펴보는 Django와 FastAPI의 발전

***

### 기: 모든 것은 "대기 시간"에서 시작된다

웹 서버가 요청 하나를 처리하는 과정을 생각해보자. 사용자가 "내 프로필을 보여줘"라고 요청하면, 서버는 DB에서 사용자 정보를 조회하고, 프로필 이미지를 읽고, 외부 API에서 추가 데이터를 가져온 뒤 응답을 만들어 돌려준다.

여기서 핵심적인 사실이 하나 있다. 이 과정에서 CPU가 실제로 연산하는 시간은 극히 일부라는 것이다. 대부분의 시간은 DB 응답 대기, 네트워크 응답 대기, 파일 읽기 대기 등 **I/O 대기**에 소비된다. 즉 웹 서버의 작업은 본질적으로 **I/O 바운드**다.

그렇다면 이 대기 시간 동안 서버는 무엇을 하고 있을까? 이 질문에 대한 답이 동기와 비동기의 차이이며, 웹 프레임워크 발전의 핵심 동력이다.

***

### 승: 동기 시대 — Django와 WSGI의 전성기

Django는 2005년에 등장했다. 이 시기의 웹 서버는 WSGI(Web Server Gateway Interface) 기반의 **동기 모델**로 동작했다. 요청이 들어오면 워커 하나가 배정되고, 그 워커는 처리가 끝날 때까지 다른 일을 하지 못한다.

python

```python
# Django의 전통적인 동기 뷰
def profile_view(request, user_id):
    user = User.objects.get(id=user_id)           # DB 대기... 워커 블로킹
    friends = Friend.objects.filter(user=user)     # DB 대기... 워커 블로킹
    external = requests.get("https://api.example.com")  # 네트워크 대기... 워커 블로킹
    return render(request, "profile.html", {"user": user, "friends": friends})
```

각 줄에서 I/O 대기가 발생할 때, 해당 워커는 아무것도 하지 못하고 멈춰 있다. 동시에 100명이 접속하면? 워커 100개가 필요하다. 1만 명이면? 워커 1만 개는 현실적으로 불가능하다. 각 워커(프로세스 또는 스레드)는 메모리를 점유하고, 컨텍스트 스위칭 비용도 발생하기 때문이다.

물론 이 모델이 오랫동안 잘 작동했다. 대부분의 웹 서비스는 동시 접속이 그렇게 많지 않았고, 서버를 수평 확장(scale-out)하면 충분히 감당할 수 있었다. 하지만 웹소켓, 실시간 알림, 대규모 API 서비스처럼 **동시 연결 수가 폭증하는 시대**가 오면서 상황이 달라졌다.

#### 파이썬의 GIL, 그리고 멀티스레드

여기서 한 가지 짚고 넘어갈 점이 있다. "파이썬은 GIL(Global Interpreter Lock) 때문에 멀티스레드가 안 되지 않나?"라는 의문이다.

GIL은 멀티스레드를 불가능하게 만드는 것이 아니라, **CPU 바운드 작업의 병렬 실행**을 막는 것이다. 스레드가 I/O 대기 상태에 들어가면 GIL을 해제하므로, 그 사이 다른 스레드가 실행될 수 있다. 웹 서버의 작업은 대부분 I/O 바운드이기 때문에, GIL이 있어도 멀티스레드 WSGI 서버는 효과적으로 동작한다.

그러나 근본적인 한계는 남는다. 워커 수에 비례해서 메모리가 증가하고, 대기 시간 동안 자원이 낭비된다는 사실은 변하지 않는다.

***

### 전: 비동기의 등장 — Uvicorn, FastAPI, 그리고 ASGI

Node.js가 이벤트 루프 기반 비동기 모델로 대규모 동시 연결 처리에 성공하면서, 파이썬 생태계도 변화를 맞이했다. asyncio가 파이썬 3.4(2014년)에 도입되었고, 이를 기반으로 \*\*ASGI(Asynchronous Server Gateway Interface)\*\*라는 새로운 표준이 등장했다.

ASGI 서버인 Uvicorn은 이벤트 루프 위에서 동작한다. 핵심 아이디어는 단순하다. I/O 대기가 발생하면 해당 요청을 잠시 멈추고 다른 요청을 처리한다. 대기가 끝나면 다시 돌아와서 마저 처리한다. 하나의 프로세스, 하나의 스레드로 수천 개의 동시 연결을 감당할 수 있는 이유다.

#### FastAPI — 비동기 우선으로 설계하다

2018년에 등장한 FastAPI는 처음부터 이 비동기 세계를 기본으로 설계되었다.

python

```python
# FastAPI의 비동기 핸들러 — 이벤트 루프에서 직접 실행
@app.get("/profile/{user_id}")
async def profile(user_id: int):
    user = await async_db.get_user(user_id)         # await → 대기 중 다른 요청 처리
    friends = await async_db.get_friends(user_id)    # await → 대기 중 다른 요청 처리
    external = await httpx.get("https://api.example.com")  # await → 대기 중 다른 요청 처리
    return {"user": user, "friends": friends}
```

`await` 키워드가 등장할 때마다 이벤트 루프에 제어권을 넘기므로, 대기 시간 동안 다른 요청이 처리된다. 워커 하나로 수천 개의 요청을 동시에 다룰 수 있다.

그러면 FastAPI의 고민은 무엇이었을까? \*\*"동기 생태계를 어떻게 품을 것인가"\*\*였다. 파이썬의 수많은 라이브러리(requests, psycopg2 등)는 동기 기반이고, 개발자 대다수가 동기 패턴에 익숙하다. FastAPI는 이 문제를 우아하게 해결했다.

python

```python
# 동기 함수로 작성해도 동작한다
@app.get("/profile/{user_id}")
def profile(user_id: int):
    user = db.get_user(user_id)
    return {"user": user}
```

`async def`가 아닌 일반 `def`로 작성하면, FastAPI가 내부적으로 이를 스레드풀에 위임(`run_in_executor`)하여 이벤트 루프가 블로킹되지 않도록 한다. 개발자는 동기 코드를 그대로 쓰면서도 비동기 서버의 이점을 어느 정도 누릴 수 있다.

다만 이 경우 해당 요청은 스레드풀을 거치게 되므로, 모든 핸들러를 동기로 작성하면 결국 WSGI와 큰 차이가 없어진다. FastAPI의 성능을 제대로 발휘하려면 `async def` + 비동기 라이브러리(`httpx`, `asyncpg` 등)를 사용하는 것이 핵심이다.

#### Django — 거대한 동기 코드베이스의 점진적 전환

Django의 상황은 훨씬 복잡했다. 15년 넘게 쌓인 동기 코드베이스, 방대한 서드파티 생태계, 수백만 개의 프로덕션 서비스가 Django 위에서 돌아가고 있었다. 한 번에 비동기로 전환하는 것은 불가능했다.

Django는 단계적 접근을 택했다.

**Django 3.0 (2019)** — ASGI 지원 시작. Uvicorn 위에서 Django를 돌릴 수 있게 되었지만, 뷰는 여전히 동기뿐이었다.

**Django 3.1 (2020)** — `async def` 뷰 도입. 비동기 뷰를 작성할 수 있게 되었지만, 핵심인 ORM은 여전히 동기였다.

python

```python
# Django 3.1+ 비동기 뷰... 하지만 ORM은 동기
async def my_view(request):
    # User.objects.all()은 동기이므로 직접 호출 불가
    users = await sync_to_async(list)(User.objects.all())
    return JsonResponse({"users": users})
```

ORM을 비동기로 쓰려면 `sync_to_async`로 감싸야 했다. 내부적으로는 스레드풀 위임이므로 진정한 비동기는 아니었다.

**Django 4.1 (2022)** — 비동기 ORM 인터페이스 도입. ORM 메서드에 비동기 버전이 추가되기 시작했다.

python

```python
# Django 4.1+ 비동기 ORM
async def my_view(request):
    users = [user async for user in User.objects.all()]
    return JsonResponse({"users": [u.name for u in users]})
```

**Django 4.2 \~ 5.x** — 비동기 ORM 지원 범위 지속 확대. 미들웨어, 시그널 등도 순차적으로 비동기 대응을 넓혀가고 있다.

Django의 이 여정은 "비동기가 좋다는 건 알지만, 기존 생태계를 깨뜨리지 않으면서 전환해야 한다"는 현실적 제약 속에서의 선택이었다.

***

### 결: 두 프레임워크가 향하는 같은 곳

FastAPI는 비동기에서 출발해 동기를 품었고, Django는 동기에서 출발해 비동기를 향해 나아가고 있다. 방향은 반대지만, 두 프레임워크가 도달하려는 곳은 같다. \*\*"동기와 비동기가 자연스럽게 공존하는 환경"\*\*이다.

이 흐름의 근본적인 이유는 결국 하나로 귀결된다. 웹 서버의 작업은 대부분 I/O 바운드이고, I/O 대기 시간을 낭비하지 않는 것이 곧 성능이다. 비동기 모델은 이 대기 시간을 가장 효율적으로 활용하는 방법이며, 그래서 파이썬 웹 생태계 전체가 비동기를 향해 움직이고 있다.

주니어 개발자로서 이 흐름을 이해하고 있다면, 단순히 "FastAPI가 빠르다"는 표면적 사실을 넘어 **왜 빠른지, 어떤 조건에서 빠른지, Django는 왜 그 길을 따라가고 있는지**를 설명할 수 있게 된다. 그리고 그 이해가 프레임워크 선택, 아키텍처 설계, 성능 최적화 모든 장면에서 더 나은 판단을 가능하게 해줄 것이다.
