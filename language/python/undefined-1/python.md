# Python 개발자를 위한 멀티스레드와 동시성 완전 가이드

### 들어가며

파이썬은 자바와 동시성 모델이 근본적으로 다릅니다. **GIL(Global Interpreter Lock)**&#xC774;라는 고유한 제약이 존재하고, 이를 우회하기 위해 `multiprocessing`과 `asyncio`라는 독자적인 경로를 발전시켜 왔습니다. 이 문서는 OS 수준의 기초부터, 파이썬의 실전 동시성 패턴, 그리고 미래의 방향까지를 정리합니다.

***

## Part 1. 기반 지식

### 1장. OS가 프로그램을 실행하는 방법

> 이 장은 언어에 관계없이 알아야 하는 운영체제 수준의 개념입니다.

#### 1.1 멀티태스킹과 멀티프로세싱

**멀티태스킹:** CPU 코어가 1개뿐이라도, 운영체제는 여러 프로그램을 빠르게 번갈아 실행(시분할)해서 동시에 돌아가는 것처럼 보이게 합니다. 어떤 프로그램에 얼마만큼의 시간을 줄지는 **운영체제의 스케줄러**가 결정합니다.

**멀티프로세싱:** CPU 코어가 여러 개라면 **물리적으로** 동시에 여러 작업을 처리할 수 있습니다.

<table><thead><tr><th width="106.953125">구분</th><th>멀티태스킹</th><th>멀티프로세싱</th></tr></thead><tbody><tr><td>관점</td><td>소프트웨어 (OS 스케줄러)</td><td>하드웨어 (CPU 코어)</td></tr><tr><td>원리</td><td>시분할로 동시 실행처럼 보이게 함</td><td>실제로 동시에 실행</td></tr></tbody></table>

#### 1.2 프로세스와 스레드

**프로세스**는 실행 중인 프로그램의 인스턴스입니다.

* 각 프로세스는 **독립적인 메모리 공간**을 가집니다 (코드, 데이터, 힙).
* 하나의 프로세스가 죽어도 다른 프로세스에 영향을 주지 않습니다.

**스레드**는 프로세스 내에서 코드를 실행하는 흐름 단위입니다.

* 같은 프로세스의 스레드끼리는 **메모리를 공유**합니다 (힙, 데이터 영역).
* 각 스레드는 **자신만의 스택**을 가집니다.
* 프로세스보다 생성/관리 비용이 가볍습니다.

#### 1.3 컨텍스트 스위칭 — 깊이 이해하기

스레드를 번갈아 실행할 때마다, 현재 스레드의 상태를 저장하고, 다음 스레드의 상태를 복원해야 합니다. 이 비용을 **컨텍스트 스위칭 비용**이라고 합니다.&#x20;

"안 할수록 좋다"는 단순한 결론보다, **어떤 일이 일어나는지** 이해하는 것이 중요합니다.

**핵심 용어 정리**

<table><thead><tr><th width="215.26953125">용어</th><th>설명</th></tr></thead><tbody><tr><td><strong>가상 주소 공간</strong><br><strong>(Virtual Address Space)</strong> </td><td>각 프로세스가 "나만 메모리 전체를 쓰고 있다"고 착각하게 만드는 구조. <br>(프로세스 A, B가 같은 주소를 써도 실제 RAM에서는 다른 위치를 가리킴)</td></tr><tr><td><strong>페이지 테이블</strong><br><strong>(Page Table)</strong></td><td>가상 주소 → 물리 주소 변환표. 프로세스마다 자신만의 것을 가짐</td></tr><tr><td><strong>MMU</strong><br><strong>(Memory Management Unit)</strong></td><td>CPU 안의 하드웨어 장치. 가상 주소를 물리 주소로 실제 변환하는 역할</td></tr><tr><td><strong>TLB</strong><br><strong>(Translation Lookaside Buffer)</strong></td><td>MMU 안의 캐시. 최근 변환 결과를 저장해두어 빠르게 재사용</td></tr><tr><td><strong>PCB</strong><br><strong>(Process Control Block)</strong> </td><td>프로세스의 모든 상태 정보를 저장하는 자료구조<br>(레지스터, 프로그램 카운터, 페이지 테이블 포인터 등)</td></tr><tr><td><strong>TCB</strong><br><strong>(Thread Control Block)</strong> </td><td>스레드의 상태 정보를 저장하는 자료구조. PCB보다 훨씬 가벼움 <br>(레지스터, 스택 포인터 등 스레드 고유 정보만)</td></tr></tbody></table>

```
CPU가 가상 주소 0x1000에 접근하려 할 때: 
  → MMU 사용 

1단계: TLB 확인 (매우 빠름, 약 1 클럭)
  → 캐싱되어 있으면 바로 물리 주소 반환 (TLB Hit)
  → 없으면 2단계로

2단계: 페이지 테이블 조회 (느림, 메인 메모리 접근 필요)
  → 물리 주소를 찾아서 반환하고, 결과를 TLB에 저장
```

**프로세스 컨텍스트 스위칭 — 무겁다**

프로세스 A에서 B로 전환될 때:

1. 프로세스 A의 모든 상태를 PCB에 저장
2. 프로세스 B의 상태를 PCB에서 불러와 CPU 레지스터에 로드
3. **MMU의 페이지 테이블 포인터가 B의 것으로 변경**
4. _**TLB 캐시 전부 무효화 (TLB Flush)** — 핵심 오버헤드!_

```
프로세스 A 실행 중:
  TLB: [0x1000→0x5A00, 0x2000→0x7B00, ...] ← 캐시가 잘 쌓여 있음 (빠름)

프로세스 B로 전환:
  TLB: [비어 있음] ← 전부 무효화됨 (TLB Flush)
  → 모든 주소 변환을 느린 메인 메모리의 페이지 테이블에서 다시 해야 함
```

**파이썬에서의 의미:** Gunicorn 같은 WSGI 서버가 워커를 프로세스 단위로 띄우는 구조에서는, 워커 간 전환 시 이 비용이 발생합니다. **CPU 바운드 작업의 경우 워커 수를 CPU 코어 수에 맞추는 이유 중 하나가 이 오버헤드를 최소화하기 위함입니다.**

**스레드 컨텍스트 스위칭 — 가볍다**

같은 프로세스 내의 스레드 A에서 B로 전환될 때:

1. 스레드 A의 일부 상태(레지스터, 스택 포인터)만 TCB에 저장
2. 스레드 B의 상태를 TCB에서 불러와 로드
3. **페이지 테이블 포인터 변경 없음** — 같은 주소 공간 공유
4. **TLB 캐시 그대로 유지!**

```
스레드 A 실행 중:
  TLB: [0x1000→0x5A00, 0x2000→0x7B00, ...] ← 캐시가 잘 쌓여 있음

스레드 B로 전환:
  TLB: [0x1000→0x5A00, 0x2000→0x7B00, ...] ← 그대로 유지!
  → 성능 저하 거의 없음
```

**파이썬에서의 의미:** CPython의 스레드는 OS 네이티브 스레드이므로, 스레드 간 전환 시 이 가벼운 컨텍스트 스위칭이 일어납니다. 다만 파이썬에는 GIL이 있어서, 스레드 전환 시 **GIL 획득/해제라는 추가 비용**이 존재합니다.

#### 1.4 CPU 바운드 vs I/O 바운드

<table><thead><tr><th width="127.47265625">구분</th><th>CPU 바운드</th><th>I/O 바운드</th></tr></thead><tbody><tr><td>특징</td><td>계산, 알고리즘 등 CPU를 100% 사용</td><td>네트워크, DB, 파일 등 대기 시간이 긴 작업</td></tr><tr><td>스레드 전략</td><td>CPU 코어 수에 맞춤</td><td>대기 시간 동안 다른 작업 가능, 더 많은 스레드 가능</td></tr><tr><td><strong>파이썬 전략</strong></td><td><code>multiprocessing</code> 사용</td><td><code>threading</code> 또는 <code>asyncio</code> 사용</td></tr></tbody></table>

**웹 서버 실무 예시:** 사용자 요청 하나를 처리할 때 CPU는 1%만 쓰고, 99%는 DB 응답을 기다린다면 — 이것은 전형적인 I/O 바운드 작업입니다. 코어가 4개라도 스레드를 4개로 제한하면 안 됩니다.

***

### 2장. 파이썬 스레드의 정체 — GIL과 OS 스레드

> 파이썬에서 스레드를 쓸 때 반드시 알아야 할 핵심 차이점을 다룹니다.

#### 2.1 파이썬 스레드는 어떻게 만들어지는가?

자바가 JNI를 통해 OS 기능을 호출하듯, CPython도 C로 작성된 내부 코드를 통해 OS의 스레드 생성 기능을 호출합니다.

```python
import threading

t = threading.Thread(target=my_function)
t.start()
```

내부적으로 일어나는 일:

1. `t.start()`가 호출되면 CPython 내부의 C 코드가 실행
2. 이 C 코드가 OS에 맞는 시스템 콜을 실행 (Linux: `pthread_create()`, Windows: `CreateThread()`)
3. OS가 실제 네이티브 스레드를 생성하고, CPython은 이 네이티브 스레드와 Python `Thread` 객체를 연결

<table><thead><tr><th width="156.8359375"></th><th>Java</th><th>Python (CPython)</th></tr></thead><tbody><tr><td>스레드 생성 코드</td><td><code>thread.start()</code></td><td><code>thread.start()</code></td></tr><tr><td>내부 호출</td><td><code>start0()</code> (native method, JNI)</td><td>C 함수 (CPython 내부)</td></tr><tr><td>OS 호출</td><td><code>pthread_create()</code> 등</td><td><code>pthread_create()</code> 등</td></tr><tr><td>결과</td><td>Java Thread ↔ OS Thread 매핑</td><td>Python Thread ↔ OS Thread 매핑</td></tr></tbody></table>

구조적으로 매우 유사합니다. 차이점은 자바는 JNI라는 명시적 인터페이스를 사용하는 반면, CPython은 인터프리터 자체가 C로 작성되어 별도 인터페이스 없이 직접 OS 함수를 호출한다는 점입니다.

#### 2.2 레벨별 스레드 이해

스레드는 동작하는 레벨에 따라 세 가지로 구분됩니다.

**하드웨어 스레드:** 코어의 연산 속도가 메모리보다 빠르기 때문에, 메모리 대기 시간 동안 다른 스레드의 작업을 수행할 수 있도록 한 것입니다. **인텔의 하이퍼스레딩**이 대표적입니다. 싱글 코어에 하드웨어 스레드가 2개라면, OS는 이를 듀얼 코어로 인식합니다.

**커널 레벨 스레드 (네이티브 스레드):** OS 커널이 직접 생성하고 관리하는 스레드입니다. 컨텍스트 스위칭에 커널이 개입하므로 비용이 발생합니다.

**유저 레벨 스레드:** 프로그래밍 언어 레벨에서 추상화한 스레드입니다. `threading.Thread`가 여기에 해당합니다.

```
[유저 레벨]     Python Thread 객체, asyncio 코루틴
                        ↓ (매핑)
[커널 레벨]     OS 네이티브 스레드 (pthread 등)
                        ↓ (스케줄링)
[하드웨어]      CPU 코어의 하드웨어 스레드
```

#### 2.3 매핑 모델: 1:1, M:1, M:N

유저 레벨 스레드를 커널 레벨 스레드와 어떻게 연결하느냐에 따라 세 가지 모델로 나뉩니다.

**1:1 모델 — CPython `threading`의 현재 방식**

유저 레벨 스레드 하나가 커널 레벨 스레드 하나와 직접 연결됩니다.

```
Python Thread 1  ↔  OS Thread 1  →  코어 1
Python Thread 2  ↔  OS Thread 2  →  코어 2
Python Thread 3  ↔  OS Thread 3  →  코어 3
```

* **장점:** OS가 각 스레드를 서로 다른 코어에 할당 가능. I/O 바운드에서 병렬성 확보에 유리.
* **단점:** 스레드 생성 = OS 스레드 생성이므로 비용이 비쌈. OS가 관리할 수 있는 수에 한계가 있음.
* **CPython 고유의 제약:** GIL 때문에 CPU 바운드에서는 진짜 병렬 실행이 안 됩니다.

> **자바와의 핵심 차이:** 자바의 1:1 모델에서는 스레드 4개가 코어 4개에서 진짜 동시에 실행됩니다. CPython에서는 GIL 때문에 Python 바이트코드를 실행하는 스레드는 항상 1개뿐입니다.

**M:1 모델**

여러 유저 레벨 스레드가 하나의 커널 레벨 스레드에 연결됩니다. CPython은 이 모델을 사용하지 않지만, GIL 때문에 CPU 바운드 작업에서는 **사실상 M:1처럼 동작**한다는 점이 흥미롭습니다.

```
실제 구조 (1:1):
스레드 A ──→ 커널 스레드 A ──→ 코어 1 (GIL 보유, 실행 중)
스레드 B ──→ 커널 스레드 B ──→ 코어 2 (GIL 대기, 놀고 있음)
스레드 C ──→ 커널 스레드 C ──→ 코어 3 (GIL 대기, 놀고 있음)

결과적으로:
스레드 A ──┐
스레드 B ──┼──→ 실행되는 건 1개뿐 ← M:1과 동일한 효과
스레드 C ──┘
```

**M:N 모델 — asyncio의 동작 방식**

M개의 경량 태스크(코루틴)를 N개의 커널 레벨 스레드에 할당합니다. 보통 M >> N입니다.

```
코루틴 1  ─┐
코루틴 2  ─┤
코루틴 3  ─┼→  이벤트 루프 (OS Thread 1)  →  코어 1
코루틴 4  ─┤
코루틴 5  ─┘
```

* **장점:** 태스크 생성/전환이 매우 빠르고 가벼움. 수만\~수십만 개의 동시 태스크 가능.
* **단점:** 스케줄링 로직을 런타임(이벤트 루프)이 직접 관리해야 하므로 구현이 복잡.

#### 2.4 GIL — 파이썬의 독특한 제약

GIL은 **한 번에 하나의 스레드만 파이썬 바이트코드를 실행할 수 있도록** 제한하는 락입니다.

```
자바: 멀티코어 → 스레드가 진짜 병렬로 실행됨
파이썬(CPython): 멀티코어 → 스레드가 있어도 한 번에 하나만 파이썬 코드 실행
```

**GIL이 존재하는 이유:** CPython의 메모리 관리(참조 카운팅)가 스레드 안전하지 않기 때문입니다.

**GIL의 영향:**

* **CPU 바운드:** 스레드를 여러 개 만들어도 병렬 실행이 안 됩니다. 오히려 GIL 경합과 컨텍스트 스위칭 비용만 늘어납니다.
* **I/O 바운드:** I/O 대기 중에는 GIL이 해제되므로, 다른 스레드가 실행될 수 있습니다. **효과적입니다.**

#### 2.5 파이썬에서의 선택지 요약

| 상황                 | 추천 방법                    | 이유                         |
| ------------------ | ------------------------ | -------------------------- |
| I/O 바운드 (네트워크, DB) | `threading` 또는 `asyncio` | I/O 대기 중 GIL 해제됨           |
| CPU 바운드 (계산, 알고리즘) | `multiprocessing`        | 별도 프로세스로 GIL 우회            |
| 대량 동시 I/O (수천\~수만) | `asyncio`                | M:N 모델로 경량 태스크             |
| 고수준 작업 관리          | `concurrent.futures`     | ThreadPool/ProcessPool 추상화 |

#### 2.6 스레드 생성 — `threading` 모듈

**방법 1: 함수를 직접 전달 (가장 일반적 — 자바의 Runnable 방식과 유사)**

```python
import threading
import time

def worker(name):
    print(f"{threading.current_thread().name}: 작업 시작 ({name})")
    time.sleep(2)
    print(f"{threading.current_thread().name}: 작업 완료 ({name})")

t1 = threading.Thread(target=worker, args=("task-1",), name="worker-1")
t2 = threading.Thread(target=worker, args=("task-2",), name="worker-2")

print(f"{threading.current_thread().name}: main 시작")
t1.start()
t2.start()

t1.join()  # t1 종료 대기
t2.join()  # t2 종료 대기
print(f"{threading.current_thread().name}: main 종료")
```

**방법 2: Thread 클래스 상속 (자바의 Thread 상속과 유사)**

```python
import threading
import time

class MyThread(threading.Thread):
    def __init__(self, name, seconds):
        super().__init__(name=name)
        self.seconds = seconds

    def run(self):
        print(f"{self.name}: 작업 시작")
        time.sleep(self.seconds)
        print(f"{self.name}: 작업 완료")

t = MyThread(name="worker-1", seconds=2)
t.start()
t.join()
```

자바에서 `Runnable` 구현이 권장되었던 것처럼, 파이썬에서도 **함수를 전달하는 방식(방법 1)**&#xC774; 더 파이썬답고 일반적입니다.

#### 2.7 데몬 스레드

자바와 동일한 개념입니다. 모든 사용자(비데몬) 스레드가 종료되면 데몬 스레드도 자동 종료됩니다.

```python
import threading
import time

def background_task():
    while True:
        print("백그라운드 작업 중...")
        time.sleep(1)

t = threading.Thread(target=background_task, daemon=True)
t.start()

time.sleep(3)
print("메인 스레드 종료 → 데몬 스레드도 자동 종료")
```

#### 2.8 스레드 기본 정보와 생명 주기

```python
import threading

t = threading.Thread(target=lambda: None, name="my-thread")

print(f"이름: {t.name}")               # my-thread
print(f"데몬 여부: {t.daemon}")         # False
print(f"생존 여부: {t.is_alive()}")     # False (start() 전)
print(f"식별자: {t.ident}")             # None (start() 후 할당)
```

자바와 달리, 파이썬에는 **스레드 우선순위 설정이나 스레드 그룹 개념이 없습니다.** 스케줄링은 전적으로 OS에 위임합니다. 생명 주기도 자바만큼 세분화되지 않습니다.

```
자바:   NEW → RUNNABLE → (BLOCKED / WAITING / TIMED_WAITING) → TERMINATED
파이썬: 생성 → start() → 실행 중 → (대기) → 종료
```

`t.is_alive()`로 스레드 실행 여부만 확인할 수 있습니다.

#### 2.9 join() — 스레드 종료 대기

자바와 동일한 개념입니다.

```python
import threading
import time

results = {}

def sum_range(name, start, end):
    time.sleep(1)
    results[name] = sum(range(start, end + 1))

t1 = threading.Thread(target=sum_range, args=("task1", 1, 50))
t2 = threading.Thread(target=sum_range, args=("task2", 51, 100))

t1.start()
t2.start()

# join() 없이 바로 결과를 읽으면 빈 딕셔너리일 수 있음!
t1.join()
t2.join()
# join(timeout=5)  # 최대 5초만 대기 가능

total = results["task1"] + results["task2"]
print(f"합계: {total}")  # 5050
```

#### 2.10 인터럽트 — 파이썬의 스레드 중단

자바에서는 `thread.interrupt()`로 스레드에 인터럽트 신호를 보낼 수 있었습니다.&#x20;

**파이썬에는 스레드 인터럽트 메커니즘이 없습니다.** 대신 `threading.Event`를 사용합니다.

```python
import threading
import time

def worker(stop_event: threading.Event):
    while not stop_event.is_set():
        print(f"[{threading.current_thread().name}] 작업 중...")
        # time.sleep() 대신 Event.wait() 사용 — 즉시 응답 가능!
        stop_event.wait(timeout=1.0)

    print(f"[{threading.current_thread().name}] 자원 정리")
    print(f"[{threading.current_thread().name}] 종료")

stop = threading.Event()
t = threading.Thread(target=worker, args=(stop,), name="work")
t.start()

time.sleep(3)
print("[main] 작업 중단 지시")
stop.set()  # 즉시 반응 — 자바의 interrupt()처럼 대기 중인 스레드를 깨움
t.join()
```

**핵심:** `time.sleep()` 대신 `stop_event.wait(timeout=1.0)`을 사용하면, `set()`이 호출되는 즉시 대기에서 빠져나옵니다. 자바에서 `interrupt()`가 `sleep()` 중인 스레드를 깨우는 것과 유사한 효과입니다.

#### 2.11 메모리 가시성 — 파이썬에서는?

자바에서 `volatile` 키워드가 필요했던 이유는 각 CPU 코어의 캐시 메모리 때문이었습니다. **파이썬에서는 GIL 덕분에 메모리 가시성 문제가 사실상 발생하지 않습니다.** GIL이 해제/획득될 때 메모리 동기화가 이루어지기 때문입니다.

단, 이것은 CPython 구현의 특성이지 언어 명세가 아니므로, **공유 자원에 대한 동기화(Lock)는 여전히 필수**입니다.

***

## Part 2. 동시성 문제와 동기화

### 3장. 공유 자원을 안전하게 다루기

> 멀티스레드의 핵심 문제인 레이스 컨디션과 이를 해결하는 동기화 기법을 다룹니다.

#### 3.1 공유 자원과 레이스 컨디션

자바의 출금 예제와 동일한 문제가 파이썬에서도 발생합니다.

```python
import threading
import time

class BankAccount:
    def __init__(self, balance):
        self.balance = balance

    def withdraw(self, amount):
        # 검증 단계
        if self.balance < amount:
            print(f"[{threading.current_thread().name}] 검증 실패: "
                  f"잔액 {self.balance} < 출금액 {amount}")
            return False

        # 출금 단계 — 검증과 출금 사이에 다른 스레드가 끼어들 수 있음!
        time.sleep(0.001)  # DB 통신 등의 지연
        self.balance -= amount
        print(f"[{threading.current_thread().name}] 출금 성공: "
              f"{amount}원, 잔액: {self.balance}")
        return True

account = BankAccount(1000)

t1 = threading.Thread(target=account.withdraw, args=(800,), name="t1")
t2 = threading.Thread(target=account.withdraw, args=(800,), name="t2")

t1.start()
t2.start()
t1.join()
t2.join()

print(f"최종 잔액: {account.balance}")
# 기대: 200원 (한 번만 성공해야 함)
# 실제: -600원이 될 수 있음! (둘 다 검증 통과)
```

**문제의 본질:** 검증 단계와 출금 단계 사이에 다른 스레드가 끼어들어 `balance` 값을 바꿀 수 있습니다. 이런 코드 영역을 **임계 영역(Critical Section)**&#xC774;라고 합니다. 여러 스레드가 동시에 접근해서는 안 되는 공유 자원을 수정하는 부분입니다.

> #### GIL이 보장하는 것과 안 하는 것
>
> GIL은 **바이트코드 명령어 1개**의 실행이 원자적임을 보장합니다. 하지만 파이썬 코드 한 줄이 반드시 바이트코드 1개인 건 아닙니다.
>
> ````python
> count = 0
>
> def increment():
>     global count
>     count += 1   # 이게 한 줄이지만...
> ```
>
> `count += 1`을 바이트코드로 보면:
> ```
> LOAD_GLOBAL   count   # 1. count 값을 읽어옴
> LOAD_CONST    1       # 2. 상수 1을 로드
> BINARY_ADD            # 3. 더함
> STORE_GLOBAL  count   # 4. 결과를 count에 저장
> ```
>
> **4개의 바이트코드**로 이루어져 있습니다. GIL은 각 바이트코드 사이에서 스레드 전환이 일어날 수 있습니다.
>
> ## 문제가 발생하는 시나리오
> ```
> count = 0 인 상태에서 두 스레드가 count += 1 실행
>
> 스레드 A: LOAD_GLOBAL count  → 0을 읽음
>           ──── 여기서 GIL 전환! ────
> 스레드 B: LOAD_GLOBAL count  → 0을 읽음 (아직 0)
>           LOAD_CONST 1
>           BINARY_ADD         → 1
>           STORE_GLOBAL count → count = 1
>           ──── GIL 전환! ────
> 스레드 A: LOAD_CONST 1
>           BINARY_ADD         → 1 (0 + 1, 아까 읽은 값 기준)
>           STORE_GLOBAL count → count = 1
>
> 결과: 2가 되어야 하는데 1이 됨!
> ````
>
> #### 그래서 락이 필요
>
> ```python
> import threading
>
> count = 0
> lock = threading.Lock()
>
> def increment():
>     global count
>     with lock:         # 임계 영역 시작
>         count += 1     # 이 안에서는 다른 스레드가 끼어들 수 없음
>                        # 임계 영역 끝
> ```
>
> `lock`을 잡으면 다른 스레드가 같은 `lock`을 잡으려 할 때 대기하게 되니, 바이트코드 중간에 끼어드는 문제를 방지할 수 있습니다.

#### 3.2 Lock — 파이썬의 synchronized

자바의 `synchronized`에 대응하는 것이 `threading.Lock`입니다.

```python
import threading
import time

class BankAccount:
    def __init__(self, balance):
        self.balance = balance
        self._lock = threading.Lock()  # 자바의 모니터 락에 해당

    def withdraw(self, amount):
        with self._lock:  # 자바의 synchronized 블록에 해당
            # 임계 영역 시작
            if self.balance < amount:
                print(f"[{threading.current_thread().name}] 검증 실패")
                return False

            time.sleep(0.001)
            self.balance -= amount
            print(f"[{threading.current_thread().name}] 출금 성공: "
                  f"{amount}원, 잔액: {self.balance}")
            return True
            # with 블록 벗어나면 자동으로 lock 해제

account = BankAccount(1000)

t1 = threading.Thread(target=account.withdraw, args=(800,), name="t1")
t2 = threading.Thread(target=account.withdraw, args=(800,), name="t2")

t1.start()
t2.start()
t1.join()
t2.join()

print(f"최종 잔액: {account.balance}")  # 항상 200원 (정상)
```

**`with` 문과 Lock:** `with self._lock:`은 자바의 `synchronized` 블록과 같은 역할입니다. `with` 블록을 벗어나면 자동으로 락이 해제되므로, 예외가 발생해도 안전합니다.

```python
# with 문 없이 수동으로 사용하는 경우 (권장하지 않음)
self._lock.acquire()
try:
    # 임계 영역
    pass
finally:
    self._lock.release()  # 반드시 해제해야 함
```

#### 3.3 RLock — 재진입 가능 락

자바의 `ReentrantLock`에 대응합니다. 같은 스레드가 이미 획득한 락을 다시 획득할 수 있습니다.

* 데드락 상황에서 사용한다.&#x20;

```python
import threading

lock = threading.RLock()  # 재진입 가능

def outer():
    with lock:
        print("outer 진입")
        inner()  # 같은 스레드가 다시 lock 획득 시도

def inner():
    with lock:  # RLock이므로 같은 스레드는 다시 획득 가능
        print("inner 진입")

outer()  # 정상 동작 (Lock이었다면 데드락 발생)
```

#### 3.4 Lock의 한계 — synchronized와 같은 문제

자바의 `synchronized`가 가진 단점은 파이썬의 `Lock`에도 동일하게 적용됩니다.

**자바 synchronized의 단점 (복습):**

* 무한 대기: `BLOCKED` 상태의 스레드는 락이 풀릴 때까지 무한 대기
* 타임아웃 불가, 인터럽트 불가
* 공정성 보장 안 됨

**파이썬 Lock의 상황:** `Lock.acquire(timeout=5)`처럼 타임아웃을 지정할 수 있다는 점에서 자바의 기본 `synchronized`보다는 유연합니다. 하지만 `Condition`을 활용한 세밀한 제어가 필요한 상황은 동일하게 발생합니다.

```python
import threading

lock = threading.Lock()

# 타임아웃 지정 가능 — 자바 synchronized에서는 불가능했던 기능
acquired = lock.acquire(timeout=5)  # 최대 5초만 대기
if acquired:
    try:
        # 임계 영역
        pass
    finally:
        lock.release()
else:
    print("락 획득 실패 — 시스템이 바쁩니다")
```

***

## Part 3. 생산자/소비자 패턴

### 4장. 스레드 간 협력 — 생산자/소비자 문제

> 동시성의 고전 문제인 생산자/소비자 문제를 단계적으로 해결합니다.

#### 4.1 문제 정의

생산자는 데이터를 만들고, 소비자는 데이터를 소비합니다. 둘 사이에 **버퍼(큐)**&#xAC00; 있습니다.

* 버퍼가 가득 차면 → 생산자는 기다려야 함
* 버퍼가 비어 있으면 → 소비자는 기다려야 함

자바에서 이 문제를 해결하기 위해 `synchronized` → `wait()/notify()` → `ReentrantLock + Condition` → `BlockingQueue`로 발전시켰습니다. 파이썬에서도 동일한 단계를 밟아봅니다.

#### 4.2 1단계: Lock만 사용 — 데이터를 버리는 구현

```python
import threading
from collections import deque

class NaiveBoundedQueue:
    """자바 BoundedQueueV1에 해당"""
    def __init__(self, max_size):
        self.queue = deque()
        self.max_size = max_size
        self.lock = threading.Lock()

    def put(self, data):
        with self.lock:
            if len(self.queue) >= self.max_size:
                print(f"[put] 큐 가득 참, 버림: {data}")
                return
            self.queue.append(data)
            print(f"[put] 저장: {data}")

    def take(self):
        with self.lock:
            if not self.queue:
                print(f"[take] 큐 비어 있음, None 반환")
                return None
            data = self.queue.popleft()
            print(f"[take] 획득: {data}")
            return data
```

**문제:** 큐가 가득 차면 데이터를 버리고, 비어 있으면 None을 반환합니다. 조금만 기다리면 되는데 기다리지 못하는 것이 아쉽습니다.

#### 4.3 2단계: Lock + sleep — 무한 대기 문제

자바의 `BoundedQueueV2`에서 발생한 것과 **동일한 문제**가 파이썬에서도 발생합니다.

```python
import threading
import time
from collections import deque

class SleepBoundedQueue:
    """자바 BoundedQueueV2에 해당 — 무한 대기 문제 발생!"""
    def __init__(self, max_size):
        self.queue = deque()
        self.max_size = max_size
        self.lock = threading.Lock()

    def put(self, data):
        with self.lock:
            while len(self.queue) >= self.max_size:
                print(f"[put] 큐 가득 참, 대기 중...")
                time.sleep(1)  # 락을 가진 채로 sleep!
            self.queue.append(data)

    def take(self):
        with self.lock:
            while not self.queue:
                print(f"[take] 큐 비어 있음, 대기 중...")
                time.sleep(1)  # 락을 가진 채로 sleep!
            return self.queue.popleft()
```

**치명적 문제:** 자바에서 겪은 것과 동일합니다. 생산자가 **락을 가진 채로** sleep하면, 소비자가 락을 얻을 수 없어서 큐에서 데이터를 꺼갈 수 없습니다. 소비자가 꺼내가야 빈 공간이 생기는데, 락을 얻지 못해 꺼갈 수 없으니 **무한 대기**에 빠집니다.

#### 4.4 3단계: Condition — wait/notify 패턴

자바의 `Object.wait()/notify()`에 대응하는 것이 파이썬의 `threading.Condition`입니다. **핵심은 대기할 때 락을 반납한다는 것**입니다.

```python
import threading

class ConditionBoundedQueue:
    """자바 BoundedQueueV3에 해당 — Condition 1개 사용"""
    def __init__(self, max_size):
        self.queue = []
        self.max_size = max_size
        self.condition = threading.Condition()  # 내부에 Lock 포함

    def put(self, data):
        with self.condition:  # lock 획득
            while len(self.queue) >= self.max_size:
                print(f"[put] 큐 가득 참, 대기")
                self.condition.wait()  # 핵심: lock 반납 + 대기
                # 자바의 Object.wait()와 동일
            self.queue.append(data)
            print(f"[put] 생산: {data}")
            self.condition.notify()  # 대기 중인 스레드 하나 깨움

    def take(self):
        with self.condition:  # lock 획득
            while not self.queue:
                print(f"[take] 큐 비어 있음, 대기")
                self.condition.wait()  # lock 반납 + 대기
            data = self.queue.pop(0)
            print(f"[take] 소비: {data}")
            self.condition.notify()  # 대기 중인 스레드 하나 깨움
            return data
```

**자바와의 대응 관계:**

| 자바                               | 파이썬                      |
| -------------------------------- | ------------------------ |
| `synchronized` + `Object.wait()` | `Condition.wait()`       |
| `Object.notify()`                | `Condition.notify()`     |
| `Object.notifyAll()`             | `Condition.notify_all()` |

**같은 한계:** Condition이 하나이므로 생산자가 생산자를 깨우거나, 소비자가 소비자를 깨우는 비효율이 발생할 수 있습니다. 자바 `BoundedQueueV3`에서 겪은 것과 동일한 문제입니다.

#### 4.5 4단계: Condition 분리 — 생산자/소비자 대기 공간 분리

자바의 `BoundedQueueV5` (ReentrantLock + 2개의 Condition)에 해당합니다.

```python
import threading

class EfficientBoundedQueue:
    """자바 BoundedQueueV5에 해당 — Condition 2개로 분리"""
    def __init__(self, max_size):
        self.queue = []
        self.max_size = max_size
        self.lock = threading.Lock()
        self.not_full = threading.Condition(self.lock)   # 생산자 대기 공간
        self.not_empty = threading.Condition(self.lock)   # 소비자 대기 공간

    def put(self, data):
        with self.not_full:
            while len(self.queue) >= self.max_size:
                self.not_full.wait()  # 생산자 전용 대기
            self.queue.append(data)
            self.not_empty.notify()   # 소비자를 깨움

    def take(self):
        with self.not_empty:
            while not self.queue:
                self.not_empty.wait()  # 소비자 전용 대기
            data = self.queue.pop(0)
            self.not_full.notify()     # 생산자를 깨움
            return data
```

**핵심:** 생산자는 소비자를 깨우고, 소비자는 생산자를 깨웁니다. 불필요한 깨우기가 사라집니다.

#### 4.6 5단계: queue.Queue — 파이썬의 BlockingQueue

자바의 `java.util.concurrent.BlockingQueue`에 대응하는 것이 파이썬의 `queue.Queue`입니다.&#x20;

**실무에서는 항상 이것을 사용하세요.** 위의 구현들은 원리를 이해하기 위한 것입니다.

```python
import queue
import threading
import time

q = queue.Queue(maxsize=2)  # 자바의 ArrayBlockingQueue(2)

def producer(name):
    for i in range(3):
        data = f"data-{i}"
        q.put(data)  # 큐가 가득 차면 자동 대기
        print(f"[{name}] 생산: {data}")

def consumer(name):
    for _ in range(3):
        data = q.get()  # 큐가 비면 자동 대기
        print(f"[{name}] 소비: {data}")
        q.task_done()

t1 = threading.Thread(target=producer, args=("P1",))
t2 = threading.Thread(target=consumer, args=("C1",))

t1.start()
t2.start()
t1.join()
t2.join()
```

**`queue.Queue`의 다양한 메서드 — 자바 BlockingQueue와의 대응:**

| 동작           | 자바 BlockingQueue                            | 파이썬 queue.Queue                           |
| ------------ | ------------------------------------------- | ----------------------------------------- |
| 무한 대기        | `put(e)` / `take()`                         | `put(data)` / `get()`                     |
| 시간 제한 대기     | `offer(e, time, unit)` / `poll(time, unit)` | `put(data, timeout=5)` / `get(timeout=5)` |
| 즉시 반환 (예외)   | `add(e)` → IllegalStateException            | `put_nowait(data)` → queue.Full           |
| 큐 비었을 때 (예외) | `remove()` → NoSuchElementException         | `get_nowait()` → queue.Empty              |

```python
import queue

q = queue.Queue(maxsize=2)

# 즉시 반환 — 큐가 가득 차면 예외
try:
    q.put_nowait("data")
except queue.Full:
    print("큐가 가득 참")

# 시간 제한 대기 — 5초까지만 기다림
try:
    q.put("data", timeout=5)
except queue.Full:
    print("5초 내에 빈 공간이 생기지 않음")

# 큐가 비었을 때 즉시 반환
try:
    data = q.get_nowait()
except queue.Empty:
    print("큐가 비어 있음")
```

**다른 큐 종류:**

```python
import queue

queue.Queue(maxsize=10)          # FIFO (일반적인 큐)
queue.LifoQueue(maxsize=10)      # LIFO (스택)
queue.PriorityQueue(maxsize=10)  # 우선순위 큐
```

***

## Part 4. 실무 도구와 파이썬의 미래

### 5장. 실무에서 쓰는 고수준 동시성 도구

> 실무에서는 저수준 Lock/Condition보다 고수준 API를 사용합니다.

#### 5.1 concurrent.futures — 스레드풀과 프로세스풀

스레드를 직접 생성/관리하는 대신, **풀(Pool)**&#xC744; 사용하는 것이 실무의 표준입니다.

```python
from concurrent.futures import ThreadPoolExecutor, as_completed
import time

def fetch_url(url):
    """I/O 바운드 작업 시뮬레이션"""
    time.sleep(1)
    return f"{url} 완료"

urls = ["https://a.com", "https://b.com", "https://c.com"]

# ThreadPoolExecutor — I/O 바운드 작업에 적합
with ThreadPoolExecutor(max_workers=3) as executor:
    futures = {executor.submit(fetch_url, url): url for url in urls}

    for future in as_completed(futures):
        url = futures[future]
        result = future.result()
        print(f"{url} → {result}")
```

```python
from concurrent.futures import ProcessPoolExecutor

def heavy_computation(n):
    """CPU 바운드 작업"""
    return sum(i * i for i in range(n))

# ProcessPoolExecutor — CPU 바운드 작업에 적합 (GIL 우회)
with ProcessPoolExecutor(max_workers=4) as executor:
    results = executor.map(heavy_computation, [10**6, 10**6, 10**6, 10**6])
    for r in results:
        print(f"결과: {r}")
```

**자바와의 대응:**

| 자바                                | 파이썬                                 |
| --------------------------------- | ----------------------------------- |
| `ExecutorService`                 | `concurrent.futures.Executor`       |
| `Executors.newFixedThreadPool(n)` | `ThreadPoolExecutor(max_workers=n)` |
| `Future<T>`                       | `concurrent.futures.Future`         |
| `future.get()`                    | `future.result()`                   |

#### 5.2 multiprocessing — 프로세스로 GIL 우회

CPU 바운드 작업의 병렬 처리를 위해 프로세스를 사용합니다.&#x20;

프로세스마다 독립된 GIL을 가지므로 진짜 병렬 실행이 가능합니다.

```python
from multiprocessing import Pool

def heavy_task(n):
    return sum(i * i for i in range(n))

# 프로세스 4개로 병렬 처리
with Pool(4) as pool:
    results = pool.map(heavy_task, [10_000_000] * 4)
```

다만 1장에서 다룬 것처럼, 프로세스 간에는 **TLB 무효화가 발생하는 무거운 컨텍스트 스위칭** 비용이 있고, 데이터를 주고받으려면 **IPC 오버헤드**가 발생하며, **메모리 사용량도 증가**합니다.

#### 5.3 asyncio — 코루틴으로 대량 동시 처리

파이썬에는 멀티스레드 외에 `asyncio`라는 강력한 선택지가 있습니다. 2장에서 다룬 M:N 모델로, 단일 스레드에서 수만 개의 동시 I/O 작업을 처리할 수 있습니다.

```python
import asyncio

async def fetch_data(name, seconds):
    print(f"[{name}] 요청 시작")
    await asyncio.sleep(seconds)  # I/O 대기 시뮬레이션
    print(f"[{name}] 요청 완료")
    return f"{name} 결과"

async def main():
    # 3개의 작업을 동시에 실행 (단일 스레드!)
    results = await asyncio.gather(
        fetch_data("A", 2),
        fetch_data("B", 1),
        fetch_data("C", 3),
    )
    print(f"모든 결과: {results}")

asyncio.run(main())
# 총 소요 시간: ~3초 (순차 실행이면 6초)
```

**코루틴의 동작 원리:**

1. 코루틴 A가 이벤트 루프 위에서 실행되다가 DB 조회(`await`)를 만남
2. 이벤트 루프가 코루틴 A의 상태를 메모리에 저장하고 일시 중단
3. 이벤트 루프가 즉시 대기 중이던 코루틴 B를 가져와 실행
4. DB 조회가 완료되면 코루틴 A가 다시 실행 가능 상태가 되어 재개

> **주의:** 코루틴 안에서 동기 블로킹 호출(예: `requests.get()`, `time.sleep()`)을 하면 이벤트 루프 전체가 멈춥니다. 반드시 비동기 대응 라이브러리를 사용하거나, `asyncio.to_thread()`로 동기 코드를 별도 스레드에서 실행해야 합니다.

**Java 가상 스레드와 asyncio 비교**

Java 21의 가상 스레드와 Python asyncio는 같은 문제(대량 I/O 동시 처리)를 풀지만, 접근이 다릅니다.

```python
# Python — async/await이 명시적으로 필요
async def get_user(user_id):
    user = await db.fetch_one("SELECT * FROM users WHERE id = $1", user_id)
    return user
```

```java
// Java 가상 스레드 — 동기 코드 그대로, JVM이 자동 처리
User getUser(int userId) {
    User user = db.query("SELECT * FROM users WHERE id = ?", userId);
    return user;
}
```

자바 가상 스레드는 기존 동기 코드를 거의 수정하지 않고도 비동기의 효율을 얻을 수 있습니다. 반면 asyncio는 `async/await`을 사용해야 하고, 동기 라이브러리(`requests`, `psycopg2` 등)를 비동기 대응 라이브러리(`aiohttp`, `asyncpg` 등)로 교체해야 합니다. 코드 변경이 크지만, `await` 지점이 명시적이므로 제어 흐름을 파악하기 쉽다는 장점이 있습니다.

**언제 무엇을 쓸까?**

| 상황                 | threading    | asyncio    |
| ------------------ | ------------ | ---------- |
| 기존 동기 라이브러리 사용     | O            | X          |
| 많은 동시 I/O (수천\~수만) | 스레드 과다 생성 문제 | O (가벼움)    |
| CPU 바운드            | X (GIL)      | X (단일 스레드) |

***

### 6장. 파이썬 동시성의 미래 — Free-threading

#### 6.1 Free-threading (PEP 703)

Python 3.13부터 실험적으로 도입된 Free-threading은 **GIL을 완전히 제거**하는 시도입니다.

```
현재 CPython (GIL 있음):
  Thread 1: [실행] → 대기 → [실행] → ...
  Thread 2: 대기 → [실행] → 대기 → ...
  → 동시에 Python 코드를 실행하는 건 항상 1개

Free-threading (GIL 없음):
  Thread 1: [실행] [실행] [실행] → 코어 1
  Thread 2: [실행] [실행] [실행] → 코어 2
  → Java처럼 진짜 병렬 실행 가능
```

이것이 안정화되면:

* Python의 `threading`도 자바처럼 진짜 병렬 실행이 가능
* `multiprocessing`의 IPC 오버헤드와 메모리 중복 문제를 피할 수 있음
* CPU 바운드에서도 멀티스레드로 성능을 낼 수 있음

다만 현재는 실험적 단계이며, 기존 C 확장 라이브러리들의 스레드 안전성 확보가 선행되어야 하므로 점진적으로 진행되고 있습니다.

#### 6.2 실무 의사결정 종합

```
작업의 성격은?
├── I/O 바운드 (네트워크, DB, 파일)
│   ├── 동시 연결 수십 개 이하 → ThreadPoolExecutor
│   ├── 동시 연결 수백~수만 개 → asyncio (ASGI: Uvicorn 등)
│   └── 기존 동기 라이브러리 필수 → ThreadPoolExecutor
│
├── CPU 바운드 (계산, 알고리즘)
│   └── ProcessPoolExecutor 또는 multiprocessing.Pool
│
└── 공유 자원 동기화 필요?
    ├── 단순 임계 영역 → Lock (with 문)
    ├── 생산자/소비자 → queue.Queue
    └── 복잡한 조건 대기 → Condition
```

**웹 서버 실무:**

| 상황                | 도구                                     |
| ----------------- | -------------------------------------- |
| 일반 웹 요청 처리        | Gunicorn (멀티프로세스) 또는 Uvicorn (asyncio) |
| DB/API 호출이 많은 경우  | asyncio 또는 threading                   |
| 이미지 처리, 수학 연산     | multiprocessing                        |
| 대량 동시 연결 (수천\~수만) | asyncio 기반 ASGI 서버 (Uvicorn 등)         |

***

### 자바 → 파이썬 전체 대응표

| 개념         | 자바                       | 파이썬                                       |
| ---------- | ------------------------ | ----------------------------------------- |
| 스레드 생성     | `new Thread(runnable)`   | `threading.Thread(target=func)`           |
| 동기화        | `synchronized`           | `threading.Lock` + `with`                 |
| 재진입 락      | `ReentrantLock`          | `threading.RLock`                         |
| 조건 대기      | `wait()` / `notify()`    | `Condition.wait()` / `Condition.notify()` |
| 조건 분리      | `Lock.newCondition()`    | `threading.Condition(lock)`               |
| 메모리 가시성    | `volatile`               | 불필요 (GIL이 보장)                             |
| 스레드 인터럽트   | `thread.interrupt()`     | `threading.Event`                         |
| 블로킹 큐      | `BlockingQueue`          | `queue.Queue`                             |
| 스레드풀       | `ExecutorService`        | `ThreadPoolExecutor`                      |
| 프로세스 병렬    | 기본 지원 (멀티스레드)            | `multiprocessing` (GIL 우회)                |
| 대량 비동기 I/O | 가상 스레드 (Java 21)         | `asyncio`                                 |
| 데몬 스레드     | `thread.setDaemon(true)` | `Thread(daemon=True)`                     |

***

### 마치며

프로세스와 스레드의 컨텍스트 스위칭은 언어에 관계없이 동일한 OS 수준의 동작입니다. 하지만 그 위에서 각 언어가 스레드를 어떻게 활용하느냐는 크게 다릅니다.

* **자바**는 1:1 모델에서 출발하여 가상 스레드(M:N)로 진화
* **파이썬**은 GIL이라는 고유한 제약 때문에 `multiprocessing`과 `asyncio`라는 우회 경로를 발전시켜 옴

**핵심 원칙:**

1. **I/O 바운드는 `threading` 또는 `asyncio`**, CPU 바운드는 `multiprocessing`
2. **공유 자원에는 반드시 Lock**, 생산자/소비자 패턴에는 `queue.Queue`
3. **실무에서는 저수준 API 대신 `concurrent.futures`와 `queue` 모듈**을 우선 사용

자바에서 배운 동시성의 원리는 파이썬에서도 유효합니다. 다만 도구의 이름과 사용법이 다를 뿐입니다. 그리고 Free-threading이라는 근본적인 해결책이 실험되고 있는 지금, 파이썬의 동시성 모델은 또 한 번의 전환점을 앞두고 있습니다.
