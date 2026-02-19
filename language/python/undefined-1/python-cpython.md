# Python 개발자를 위한 CPython 완전 가이드

### 들어가며

자바 개발자 출신이라면 JVM(Java Virtual Machine)의 동작 원리를 어느 정도 이해하고 있을 것입니다. 소스 코드가 바이트코드로 컴파일되고, JVM이 이를 실행하며, 가비지 컬렉터가 메모리를 관리한다는 흐름은 익숙합니다.

파이썬도 놀랍도록 비슷한 구조를 가지고 있습니다. 하지만 세부 동작은 크게 다릅니다. 파이썬이 "인터프리터 언어" 라고 불리지만, 실제로는 **컴파일 단계가 존재**합니다. 메모리 관리 방식도 JVM의 GC 중심 모델과 달리 **참조 카운팅 + GC 하이브리드** 구조입니다.

이 문서는 "파이썬 코드를 실행하면 내부에서 무슨 일이 일어나는가?"라는 질문에 대해, JVM과의 비교를 곁들이며 기승전결 구조로 답합니다.

***

## Part 1. CPython의 정체

### 1장. 파이썬은 인터프리터 언어인가?

#### 1.1 "파이썬"은 언어 명세일 뿐이다

많은 사람이 "파이썬"이라고 하면 하나의 프로그램을 떠올리지만, 정확히는 **파이썬은 언어 명세(specification)**&#xC774;고, 이를 구현한 **인터프리터가 여러 개** 존재합니다.

| 구현체             | 작성 언어            | 특징                                |
| --------------- | ---------------- | --------------------------------- |
| **CPython**     | C                | 공식 레퍼런스 구현. 우리가 `python`이라고 부르는 것 |
| **PyPy**        | Python (RPython) | JIT 컴파일러 탑재. CPython보다 빠른 경우가 많음  |
| **Jython**      | Java             | JVM 위에서 실행. 자바 라이브러리 직접 사용 가능     |
| **IronPython**  | C#               | .NET CLR 위에서 실행                   |
| **MicroPython** | C                | 마이크로컨트롤러용 경량 구현                   |

이 문서에서 다루는 것은 **CPython**입니다. `pip install`로 패키지를 설치하고, `python` 명령어로 코드를 실행할 때 사용하는 바로 그것입니다.

> **자바와의 비교:** 자바에서도 "자바"는 언어 명세이고, Oracle JDK, OpenJDK, GraalVM, Amazon Corretto 등 여러 구현체가 존재합니다. CPython은 자바 세계의 Oracle JDK/OpenJDK에 해당합니다.

#### 1.2 CPython의 실행 흐름 — 전체 그림

파이썬 코드가 실행되는 과정을 한눈에 보겠습니다.

```
[소스 코드]  →  [컴파일러]  →  [바이트코드]  →  [PVM (인터프리터)]  →  [실행 결과]
 hello.py       CPython        .pyc 파일        Python Virtual       화면 출력
                 내부                            Machine
```

**자바의 실행 흐름과 비교:**

```
자바:   .java  →  javac (컴파일러)  →  .class (바이트코드)  →  JVM  →  실행
파이썬: .py    →  CPython 컴파일러  →  .pyc (바이트코드)    →  PVM  →  실행
```

놀랍도록 유사합니다. 핵심적인 차이점은:

|          | 자바                    | 파이썬 (CPython)               |
| -------- | --------------------- | --------------------------- |
| 컴파일 시점   | 명시적 (`javac` 별도 실행)   | 암묵적 (실행 시 자동으로 컴파일)         |
| 바이트코드 파일 | `.class`              | `.pyc` (`__pycache__` 디렉토리) |
| 실행 엔진    | JVM (JIT 컴파일러 포함)     | PVM (기본은 인터프리터만)            |
| 타입 검사    | 컴파일 타임에 정적 타입 검사      | 런타임에 동적 타입 검사               |
| 최적화 수준   | JIT로 네이티브 코드 수준까지 최적화 | 기본적으로 바이트코드 인터프리팅           |

**"인터프리터 언어"라는 오해:**&#x20;

파이썬도 컴파일을 합니다. 단지 그 과정이 개발자에게 투명하게 숨겨져 있고, 결과물인 바이트코드를 네이티브 코드가 아닌 가상 머신(PVM)이 해석한다는 것이 다릅니다. 정확히 말하면 파이썬은 **"바이트코드로 컴파일된 후 인터프리팅되는 언어"(자바와 동일)**&#xC785;니다.

#### 1.3 CPython은 왜 C로 작성되었는가?

CPython 인터프리터 자체가 C 언어로 작성되어 있다는 사실은 여러 가지 중요한 의미를 가집니다.

**C로 작성된 이유:**

* **성능:** C는 하드웨어에 가까운 저수준 언어로, 인터프리터 자체의 실행 속도가 빠릅니다.
* **OS 접근:** 시스템 콜, 메모리 관리, 스레드 생성 등 OS 기능에 직접 접근할 수 있습니다.
* **C 확장:** NumPy, pandas 같은 고성능 라이브러리가 C/C++로 작성된 확장 모듈을 포함할 수 있습니다.
* **이식성:** C 컴파일러는 거의 모든 플랫폼에 존재하므로, 파이썬도 거의 모든 곳에서 실행됩니다.

**실무적 의미:**

```python
# 이 파이썬 코드를 실행하면...
result = sum(range(1000000))

# 내부적으로는 C로 작성된 sum() 함수와 range() 객체가 실행됨
# → 파이썬 코드보다 훨씬 빠름
```

파이썬의 내장 함수(`sum`, `len`, `sorted` 등)와 표준 라이브러리 상당 부분은 C로 구현되어 있습니다.&#x20;

"파이썬은 느리다"고 말할 때, 이것은 **순수 파이썬 코드**의 실행 속도를 말하는 것이지, C로 작성된 내장 함수를 활용하는 코드까지 느린 것은 아닙니다.

> **자바와의 비교:** JVM도 대부분 C/C++로 작성되어 있습니다. `java.lang.String`의 내부 구현이 C++인 것처럼, 파이썬의 `str` 타입도 내부적으로는 C 구조체입니다.

***

## Part 2. 바이트코드와 가상 머신

### 2장. 소스 코드에서 실행까지

#### 2.1 컴파일 파이프라인 — 4단계

파이썬 소스 코드가 바이트코드로 변환되는 과정은 4단계로 이루어집니다.

```
소스 코드 (.py)
    ↓ [1단계: 토크나이징 (Tokenizing)]
토큰 스트림
    ↓ [2단계: 파싱 (Parsing)]
AST (Abstract Syntax Tree, 추상 구문 트리)
    ↓ [3단계: 컴파일 (Compiling)]
바이트코드 (Code Object)
    ↓ [4단계: 실행 (Execution)]
PVM이 바이트코드를 한 명령씩 해석/실행
```

각 단계를 실제 코드로 확인해봅시다.

**1단계: 토크나이징 — 소스 코드를 토큰으로 분해**

```python
import tokenize
import io

code = "x = 10 + 20"
tokens = tokenize.generate_tokens(io.StringIO(code).readline)

for tok in tokens:
    print(tok)

# TokenInfo(type=1 (NAME),     string='x',  ...)
# TokenInfo(type=54 (OP),      string='=',  ...)
# TokenInfo(type=2 (NUMBER),   string='10', ...)
# TokenInfo(type=54 (OP),      string='+',  ...)
# TokenInfo(type=2 (NUMBER),   string='20', ...)
```

소스 코드 문자열을 의미 있는 최소 단위(토큰)로 분해합니다. 자바의 `javac`도 동일한 과정을 거칩니다.

**2단계: 파싱 — 토큰을 AST로 변환**

```python
import ast

code = "x = 10 + 20"
tree = ast.parse(code)
print(ast.dump(tree, indent=2))

# Module(
#   body=[
#     Assign(
#       targets=[Name(id='x', ctx=Store())],
#       value=BinOp(
#         left=Constant(value=10),
#         op=Add(),
#         right=Constant(value=20)
#       )
#     )
#   ]
# )
```

토큰 스트림을 **추상 구문 트리(AST)**&#xB85C; 변환합니다. AST는 코드의 구조적 의미를 트리 형태로 표현한 것입니다. \
위 결과를 보면 "x에 10 + 20의 결과를 대입한다"는 구조가 명확하게 드러납니다.

> **실무 활용:** `ast` 모듈은 코드 분석 도구, 린터(linter), 코드 변환 도구 등에서 활용됩니다. \
> 예를 들어, `flake8` 같은 도구가 내부적으로 AST를 분석합니다.

**3단계: 컴파일 — AST를 바이트코드로 변환**

```python
code = "x = 10 + 20"
code_obj = compile(code, "<string>", "exec")

print(type(code_obj))       # <class 'code'>
print(code_obj.co_consts)   # (30, None)  ← 10+20이 컴파일 타임에 30으로 최적화됨!
print(code_obj.co_names)    # ('x',)
```

AST가 바이트코드(Code Object)로 변환됩니다. 여기서 주목할 점은 `10 + 20`이 컴파일 시점에 **상수 폴딩(Constant Folding)** 되어 `30`으로 최적화된다는 것입니다. "파이썬은 최적화를 안 한다"는 것은 사실이 아닙니다.

**4단계: 실행 — PVM이 바이트코드를 해석**

이 단계는 2.3절에서 자세히 다룹니다.

#### 2.2 바이트코드 확인하기 — `dis` 모듈

`dis` 모듈을 사용하면 파이썬 바이트코드를 직접 확인할 수 있습니다.

```python
import dis

def add(a, b):
    return a + b

dis.dis(add)
```

출력:

```
  2           0 LOAD_FAST                0 (a)
              2 LOAD_FAST                1 (b)
              4 BINARY_ADD
              6 RETURN_VALUE
```

각 열의 의미:

| 열       | 의미            | 예시                          |
| ------- | ------------- | --------------------------- |
| 첫 번째 숫자 | 소스 코드 줄 번호    | `2` (2번째 줄)                 |
| 두 번째 숫자 | 바이트코드 오프셋     | `0`, `2`, `4`, `6`          |
| 명령어     | 연산 코드(opcode) | `LOAD_FAST`, `BINARY_ADD` 등 |
| 인자      | 명령어의 인자       | `0 (a)`, `1 (b)`            |

**이것이 PVM이 실제로 실행하는 명령어**입니다.

**바이트코드 실행 흐름 — 스택 기반 가상 머신**

PVM은 **스택 기반 가상 머신**입니다. 모든 연산이 스택에 값을 넣고(`push`) 빼는(`pop`) 방식으로 이루어집니다.

```
add(3, 5) 호출 시:

LOAD_FAST 0 (a)     → 스택: [3]
LOAD_FAST 1 (b)     → 스택: [3, 5]
BINARY_ADD           → 3과 5를 꺼내서 더하고, 결과를 넣음 → 스택: [8]
RETURN_VALUE         → 스택 최상단 값(8)을 반환
```

> **자바와의 비교:** JVM도 스택 기반 가상 머신입니다. 자바의 `iadd` 명령어가 파이썬의 `BINARY_ADD`에 해당합니다. 이 부분은 구조적으로 거의 동일합니다.

**좀 더 복잡한 예제**

```python
import dis

def greet(name):
    message = f"Hello, {name}!"
    print(message)
    return message

dis.dis(greet)
```

```
  2           0 LOAD_CONST               1 ('Hello, ')
              2 LOAD_FAST                0 (name)
              4 FORMAT_VALUE             0
              6 LOAD_CONST               2 ('!')
              8 BUILD_STRING             3
             10 STORE_FAST               1 (message)

  3          12 LOAD_GLOBAL              0 (print)
             14 LOAD_FAST                1 (message)
             16 CALL_FUNCTION            1
             18 POP_TOP

  4          20 LOAD_FAST                1 (message)
             22 RETURN_VALUE
```

f-string도 결국 `FORMAT_VALUE`와 `BUILD_STRING`이라는 바이트코드 명령어로 변환되는 것을 볼 수 있습니다.

#### 2.3 PVM(Python Virtual Machine) — 실행 엔진

PVM은 CPython 내부에서 바이트코드를 읽고 실행하는 핵심 엔진입니다. C 함수`_PyEval_EvalFrameDefault()` 가 그 실체입니다.

**PVM의 동작 방식:**

```
while True:
    opcode = 다음_바이트코드_명령어_읽기()

    if opcode == LOAD_FAST:
        # 지역 변수를 스택에 push
    elif opcode == BINARY_ADD:
        # 스택에서 두 값을 pop, 더하고, 결과를 push
    elif opcode == RETURN_VALUE:
        # 스택 최상단 값을 반환하고 루프 종료
    # ... 수백 개의 opcode 처리
```

이것이 바이트코드 **디스패치 루프(dispatch loop)**&#xC785;니다. \
실제 CPython 소스에서는 성능을 위해 `switch-case` 또는 **computed goto** 방식을 사용합니다.

**자바 JVM과의 핵심 차이 — JIT 컴파일러:**

|       | JVM                 | CPython PVM   |
| ----- | ------------------- | ------------- |
| 실행 방식 | 인터프리팅 + **JIT 컴파일** | 인터프리팅만 (기본)   |
| 최적화   | 핫스팟을 네이티브 코드로 컴파일   | 바이트코드 수준 최적화만 |
| 성능    | 반복 실행 시 네이티브에 근접    | 항상 바이트코드 해석   |

JVM은 자주 실행되는 코드(핫스팟)를 JIT(Just-In-Time) 컴파일러로 네이티브 머신 코드로 변환합니다. 덕분에 반복 실행 시 성능이 크게 향상됩니다.

**CPython은 기본적으로 이 기능이 없어서, 매번 바이트코드를 해석합니다. 이것이 파이썬이 자바보다 느린 주된 이유 중 하나입니다.**

> **변화의 조짐:**&#x20;
>
> * Python 3.11부터 **특화 적응형 인터프리터(Specializing Adaptive Interpreter, PEP 659)**&#xAC00; 도입되어, 자주 실행되는 바이트코드를 타입에 특화된 버전으로 자동 교체합니다.&#x20;
> * Python 3.13에서는 실험적 **JIT 컴파일러(PEP 744)**&#xAC00; 추가되었습니다.&#x20;
> * 파이썬도 JVM의 방향으로 진화하고 있습니다.

#### 2.4 .pyc 파일과 `__pycache__`

```
my_project/
├── main.py
├── utils.py
└── __pycache__/
    └── utils.cpython-312.pyc    ← utils.py의 컴파일된 바이트코드
```

파이썬은 모듈을 `import`할 때 컴파일된 바이트코드를 `.pyc` 파일로 캐싱합니다. 다음에 같은 모듈을 `import`하면 소스 코드 수정 여부를 확인하고, 변경이 없으면 `.pyc`를 바로 사용합니다.

```python
# .pyc 파일 확인
import py_compile

py_compile.compile('utils.py')  # utils.py → __pycache__/utils.cpython-312.pyc
```

**자바 `.class` 파일과의 비교:**

|        | 자바 `.class`          | 파이썬 `.pyc`         |
| ------ | -------------------- | ------------------ |
| 생성 시점  | `javac`로 명시적 컴파일     | `import` 시 자동 생성   |
| 포맷 안정성 | 버전 간 호환성 높음          | 버전 간 호환 **보장 안 됨** |
| 배포     | `.class`나 `.jar`로 배포 | `.py` 소스 코드로 배포    |

파이썬의 `.pyc` 포맷은 CPython 구현의 세부사항이며, 버전 간 호환성이 보장되지 않습니다. 그래서 파이썬 프로젝트는 `.pyc`가 아닌 `.py` 소스 코드를 배포합니다.

#### 2.5 Code Object — 바이트코드의 컨테이너

함수가 컴파일되면 **Code Object**가 생성됩니다. 이 객체에는 바이트코드뿐 아니라 함수 실행에 필요한 모든 메타데이터가 담겨 있습니다.

```python
def example(x, y):
    z = x + y
    return z

code = example.__code__

print(code.co_name)        # 'example'
print(code.co_varnames)    # ('x', 'y', 'z')
print(code.co_consts)      # (None,)
print(code.co_argcount)    # 2
print(code.co_stacksize)   # 2 (스택의 최대 깊이)
print(code.co_bytecode)    # 바이트 문자열 (바이트코드 원본)
```

| 속성             | 설명                |
| -------------- | ----------------- |
| `co_name`      | 함수 이름             |
| `co_varnames`  | 지역 변수 이름 튜플       |
| `co_consts`    | 사용되는 상수 값 튜플      |
| `co_argcount`  | 인자 개수             |
| `co_stacksize` | 실행 시 필요한 스택 최대 깊이 |
| `co_code`      | 실제 바이트코드 바이트열     |

> **자바와의 비교:** 자바의 `.class` 파일에 담긴 메서드 정보(Constant Pool, Local Variable Table 등)와 개념적으로 동일합니다.

#### 2.6 Frame Object — 실행 컨텍스트

Code Object가 "어떤 코드를 실행할지"를 정의한다면, **Frame Object**는 "지금 실행 중인 상태"를 나타냅니다. 함수가 호출될 때마다 Frame이 생성됩니다.

```python
import sys

def inner():
    frame = sys._getframe()  # 현재 실행 중인 프레임 가져오기
    print(f"현재 함수: {frame.f_code.co_name}")
    print(f"호출한 함수: {frame.f_back.f_code.co_name}")
    print(f"지역 변수: {frame.f_locals}")

def outer():
    x = 42
    inner()

outer()
# 현재 함수: inner
# 호출한 함수: outer
# 지역 변수: {}  (inner의 지역 변수)
```

Frame에는 다음이 담겨 있습니다.

| 속성          | 설명                      |
| ----------- | ----------------------- |
| `f_code`    | 실행 중인 Code Object       |
| `f_locals`  | 지역 변수 딕셔너리              |
| `f_globals` | 전역 변수 딕셔너리              |
| `f_back`    | 호출자의 Frame (콜스택 역추적 가능) |
| `f_lasti`   | 마지막으로 실행한 바이트코드 오프셋     |

함수 호출이 중첩되면 Frame이 스택처럼 쌓입니다. 이것이 **콜 스택(Call Stack)**&#xC785;니다.

```
outer() 호출 → Frame(outer) 생성
  inner() 호출 → Frame(inner) 생성
    실행 중...
  inner() 반환 → Frame(inner) 소멸
outer() 반환 → Frame(outer) 소멸
```

> **자바와의 비교:** JVM의 스택 프레임과 동일한 개념입니다. 자바에서 메서드를 호출할 때마다 스택 프레임이 쌓이고, 반환하면 제거되는 것과 같습니다. 자바의 `this` 참조가 스택 프레임에 저장되듯, 파이썬의 `self`도 지역 변수로 Frame에 저장됩니다.

***

## Part 3. 메모리 관리

### 3장. 객체는 어떻게 생성되고 사라지는가

#### 3.1 모든 것은 객체다 — PyObject

파이썬에서 **모든 것은 객체**입니다. _정수, 문자열, 함수, 클래스, 모듈_ 전부 객체입니다.

```python
print(type(42))          # <class 'int'>
print(type("hello"))     # <class 'str'>
print(type(print))       # <class 'builtin_function_or_method'>
print(type(int))         # <class 'type'>
```

CPython 내부에서 모든 파이썬 객체는 C 구조체 `PyObject`로 표현됩니다.

```c
// CPython 내부의 모든 객체의 기본 구조 (단순화)
typedef struct _object {
    Py_ssize_t ob_refcnt;    // 참조 카운트
    PyTypeObject *ob_type;    // 타입 정보 포인터
} PyObject;
```

모든 파이썬 객체는 최소한 이 두 필드를 가집니다.&#x20;

* **참조 카운트(`ob_refcnt`)**
* **타입 포인터(`ob_type`)**

이것이 파이썬 메모리 관리의 출발점입니다.

> **자바와의 비교:** 자바에서도 모든 객체는 Object 헤더를 가지며, 여기에 클래스 정보 포인터와 해시코드, 락 정보 등이 담깁니다. 하지만 자바에는 **참조 카운트가 없습니다**. 이것이 두 언어의 메모리 관리 방식을 근본적으로 다르게 만듭니다.

#### 3.2 참조 카운팅 — 주 메모리 관리 메커니즘

CPython의 **주(primary)** 메모리 관리 방식은 **참조 카운팅(Reference Counting)**&#xC785;니다.

**원리:** 모든 객체는 자신을 참조하는 변수의 수를 추적합니다. 이 참조 카운트가 0이 되면 즉시 메모리에서 해제됩니다.

```python
import sys

a = [1, 2, 3]          # 리스트 객체 생성, 참조 카운트 = 1
print(sys.getrefcount(a))  # 2 (a + getrefcount의 임시 참조)

b = a                   # 같은 객체를 b도 참조, 참조 카운트 = 2
print(sys.getrefcount(a))  # 3

c = [a, a]              # 리스트 안에서도 참조, 참조 카운트 = 4
print(sys.getrefcount(a))  # 5

del b                   # 참조 제거, 참조 카운트 감소
del c                   # 리스트 삭제 → 내부 참조도 감소
del a                   # 마지막 참조 제거 → 참조 카운트 = 0 → 즉시 해제!
```

**참조 카운트가 증가하는 경우:**

* 변수에 할당: `a = obj`
* 컨테이너에 추가: `my_list.append(obj)`
* 함수 인자로 전달: `func(obj)`

**참조 카운트가 감소하는 경우:**

* `del` 문: `del a`
* 변수가 다른 객체를 참조: `a = other_obj`
* 변수가 스코프를 벗어남 (함수 종료 등)
* 컨테이너에서 제거: `my_list.remove(obj)`

**참조 카운팅 vs JVM GC — 근본적인 차이**

|          | CPython (참조 카운팅)      | JVM (Tracing GC)              |
| -------- | --------------------- | ----------------------------- |
| 해제 시점    | **참조 카운트가 0이 되는 즉시**  | GC가 실행될 때 (비결정적)              |
| 예측 가능성   | 높음 (언제 해제되는지 예측 가능)   | 낮음 (GC 타이밍을 모름)               |
| 오버헤드     | 참조할 때마다 카운트 조정        | GC 실행 시 일시 정지(Stop-the-World) |
| 순환 참조 처리 | **처리 불가** (별도 GC 필요)  | 자동 처리                         |
| 메모리 사용   | 객체마다 카운트 필드 (8바이트) 추가 | 객체 헤더에 GC용 마크 비트              |

**실무적 의미:**

```python
# 파이썬: 파일이 즉시 닫힘 (참조 카운팅 덕분)
def read_file():
    f = open("data.txt")
    data = f.read()
    return data
# 함수 반환 시 f의 참조 카운트가 0 → 파일 핸들 즉시 해제

# 자바: 파일이 언제 닫힐지 모름 (GC 의존)
// 그래서 자바에서는 try-with-resources가 필수
```

하지만 CPython에서도 이 동작에 의존하면 안 됩니다.&#x20;

* 다른 파이썬 구현체(PyPy 등)는 참조 카운팅을 사용하지 않고,
* 순환 참조가 일어날 경우, 참조 카운트가 줄어들지 않는다.&#x20;
* **`with` 문을 사용하는 것이 올바른 방법**입니다.\
  (`with` 문은 블록이 끝나면 무조건 정리한다)

```python
# 올바른 방법 — 어떤 파이썬 구현체에서든 안전
with open("data.txt") as f:
    data = f.read()
# with 블록을 벗어나면 f.close()가 보장됨
```

#### 3.3 순환 참조 문제 — 참조 카운팅의 한계

참조 카운팅이 처리하지 못하는 치명적인 상황이 있습니다. **순환 참조(Circular Reference)**&#xC785;니다.

```python
class Node:
    def __init__(self, name):
        self.name = name
        self.ref = None

a = Node("A")
b = Node("B")

# 서로를 참조하는 순환 구조 생성
a.ref = b    # A → B
b.ref = a    # B → A

del a        # a 변수 삭제 → 하지만 B가 여전히 A를 참조 → 카운트 = 1
del b        # b 변수 삭제 → 하지만 A가 여전히 B를 참조 → 카운트 = 1
# 두 객체 모두 참조 카운트가 1 → 해제되지 않음!
# 하지만 프로그램에서는 이 객체들에 접근할 방법이 없음 → 메모리 누수!
```

```
삭제 전:                    삭제 후:
a → [Node A] ←──┐         [Node A] ←──┐
         │      │              │      │
         ↓      │              ↓      │
b → [Node B] ───┘         [Node B] ───┘
                           (접근 불가하지만 참조 카운트 ≠ 0)
```

이 문제를 해결하기 위해 CPython은 **세대별 가비지 컬렉터(Generational GC)**&#xB97C; 추가로 사용합니다.

#### 3.4 세대별 가비지 컬렉터 — 순환 참조 해결

CPython의 GC는 순환 참조를 탐지하고 해제하기 위한 **보조** 메커니즘입니다. 주 메커니즘은 여전히 참조 카운팅입니다.

**세대(Generation)의 개념**

객체를 나이에 따라 세대로 분류합니다.&#x20;

"대부분의 객체는 빨리 죽는다"는 **세대 가설(Generational Hypothesis)**&#xC5D0; 기반합니다. (JVM 과 동일)

```
세대 0 (Young)  ← 새로 생성된 모든 객체가 여기에 들어감
    ↓ GC에서 살아남으면 승격
세대 1 (Middle)
    ↓ GC에서 살아남으면 승격
세대 2 (Old)   ← 오래 살아남은 객체
```

* **세대 0:** 가장 자주 GC 실행 (새 객체는 대부분 금방 죽으므로)
* **세대 1:** 덜 자주 실행
* **세대 2:** 가장 드물게 실행 (오래 산 객체는 계속 살 가능성이 높으므로)

```python
import gc

# 현재 GC 임계값 확인
print(gc.get_threshold())  # (700, 10, 10)

# 의미: 세대 0에서 700개의 객체가 할당되면 GC 실행
#       세대 0 GC가 10번 실행되면 세대 1 GC 실행
#       세대 1 GC가 10번 실행되면 세대 2 GC 실행
```

**GC의 순환 참조 탐지 원리 (간략)**

1. 컨테이너 객체들(리스트, 딕셔너리, 클래스 인스턴스 등)의 참조를 추적
2. 각 객체의 참조 카운트에서 내부 참조를 임시로 제거
3. 그래도 외부에서 접근 가능한 객체는 **살아있는 것**으로 판정
4. 외부에서 접근할 수 없는 객체들은 순환 참조 그룹으로 판단하여 **해제**

```python
import gc

class Node:
    def __init__(self, name):
        self.name = name
        self.ref = None
    def __del__(self):
        print(f"Node {self.name} 해제됨")

a = Node("A")
b = Node("B")
a.ref = b
b.ref = a

del a
del b
# 아직 해제되지 않음 (순환 참조)

gc.collect()  # 수동으로 GC 실행
# 출력: Node A 해제됨
# 출력: Node B 해제됨
```

**JVM GC와의 비교**

|                | CPython GC                                                        | JVM GC                       |
| -------------- | ----------------------------------------------------------------- | ---------------------------- |
| 역할             | 순환 참조 해결 (보조)                                                     | **모든** 메모리 해제 (주)            |
| 주 메커니즘         | 참조 카운팅이 주, GC가 보조                                                 | Tracing GC가 주 (Mark-Sweep 등) |
| 세대 수           | 3세대 (0, 1, 2)                                                     | 구현에 따라 다름 (G1GC, ZGC 등)      |
| Stop-the-World | <p>GIL 영향으로 GC 동안 앱 코드 실행이 잠시 중단된다<br>(자바에서 STW 매커니즘과 차이가 있음)</p> | GC 실행 시 발생 (최적화 계속 진행 중)     |
| 순환 참조          | GC가 탐지/해제                                                         | 자동 처리 (Tracing이니까)           |
| 비순환 객체 해제      | 참조 카운팅으로 즉시                                                       | GC 실행 시                      |

#### 3.5 CPython의 메모리 최적화 기법

CPython은 성능을 위해 몇 가지 영리한 최적화를 사용합니다.

**작은 정수 캐싱 (Integer Interning)**

```python
a = 256
b = 256
print(a is b)  # True — 같은 객체!

a = 257
b = 257
print(a is b)  # False — 다른 객체!
```

CPython은 **-5부터 256까지의 정수**를 인터프리터 시작 시 미리 생성하고 캐싱합니다.&#x20;

이 범위의 정수를 사용할 때마다 새 객체를 만들지 않고 기존 객체를 재사용합니다.

**문자열 인터닝 (String Interning)**

```python
a = "hello"
b = "hello"
print(a is b)  # True — 같은 객체! (짧은 문자열은 인터닝됨)

a = "hello world!"
b = "hello world!"
print(a is b)  # False — 다른 객체 (공백, 특수문자 포함 시 인터닝 안 될 수 있음)
```

식별자처럼 생긴 짧은 문자열은 자동으로 인터닝됩니다.

> **자바와의 비교:** 자바도 `-128 ~ 127` 범위의 `Integer`를 캐싱하고, 문자열 리터럴을 String Pool에 인터닝합니다. 같은 최적화 아이디어를 서로 다른 범위로 적용한 것입니다.

**메모리 풀 (pymalloc)**

CPython은 작은 객체(512바이트 이하)에 대해 자체 메모리 할당기(`pymalloc`)를 사용합니다.&#x20;

OS의 `malloc()`을 매번 호출하지 않고, 미리 할당받은 메모리 풀에서 빠르게 할당/해제합니다.

```
OS 메모리 (malloc)
    ↓
pymalloc 아레나 (256KB 단위)
    ↓
풀 (4KB 단위, 같은 크기의 블록들)
    ↓
블록 (8, 16, 24, ... 512 바이트)
```

> **자바와의 비교:** JVM도 힙 메모리를 Young/Old 영역으로 나누어 관리하며, TLAB(Thread Local Allocation Buffer) 등의 최적화를 사용합니다. 개념적으로 유사합니다.

***

## Part 4. 객체 모델과 실무 인사이트

### 4장. 이 지식이 실무에서 왜 중요한가

#### 4.1 동적 타입의 비용 이해하기

파이썬이 자바보다 느린 근본적인 이유 중 하나는 **동적 타입 시스템**입니다.

<pre class="language-python"><code class="lang-python"># 파이썬: a + b를 실행할 때마다...
def add(a, b):
    return a + b

<strong># PVM은 매번 이런 과정을 거침:
</strong># 1. a의 타입을 확인 (int? float? str? 사용자 정의 클래스?)
# 2. 해당 타입의 __add__ 메서드를 찾음
# 3. b의 타입과 호환되는지 확인
# 4. 실제 덧셈 수행
</code></pre>

```java
// 자바: 컴파일 타임에 이미 결정됨
int add(int a, int b) {
    return a + b;  // iadd 명령어 하나로 끝
}
```

파이썬은 **매번 실행 시점에 타입을 확인**해야 하므로, 같은 연산이라도 오버헤드가 큽니다.&#x20;

* 이것이 PEP 659(특화 적응형 인터프리터) - v3.11 가 해결하려는 문제입니다.&#x20;

자주 실행되는 코드에서 "이 변수는 항상 int다"라는 패턴을 감지하면, 타입 확인을 생략하는 특화된 바이트코드로 교체합니다.

#### 4.2 `is` vs `==` — 객체 모델의 이해

```python
a = [1, 2, 3]
b = [1, 2, 3]
c = a

print(a == b)   # True  — 값이 같은가?
print(a is b)   # False — 같은 객체인가? (id가 다름)
print(a is c)   # True  — 같은 객체! (id가 같음)
```

`is`는 두 변수가 **같은 PyObject를 가리키는지** (C 포인터 비교), `==`는 `__eq__` 메서드를 호출하여 **값이 같은지** 비교합니다.

> **자바와의 비교:** 자바의 `==`가 파이썬의 `is`에 해당하고, 자바의 `.equals()`가 파이썬의 `==`에 해당합니다.

```python
# None 비교는 항상 is를 사용
if result is None:     # 올바름 (None은 싱글톤)
    pass

if result == None:     # 작동하지만 권장하지 않음
    pass
```

#### 4.3 메모리 프로파일링 — 실무 도구

```python
# 객체별 참조 카운트 확인
import sys

data = {"key": "value"}
print(sys.getrefcount(data))  # 참조 카운트

# 객체 크기 확인
print(sys.getsizeof(data))          # 딕셔너리 객체 자체의 크기
print(sys.getsizeof("hello"))       # 문자열 객체 크기
print(sys.getsizeof(42))            # 정수 객체 크기 (28 바이트!)
```

```python
# GC 상태 확인 및 디버깅
import gc

gc.set_debug(gc.DEBUG_STATS)  # GC 실행 시 통계 출력
gc.collect()                   # 수동 GC 실행

# 현재 추적 중인 객체 수
print(len(gc.get_objects()))
```

```python
# tracemalloc — 메모리 할당 추적
import tracemalloc

tracemalloc.start()

# ... 코드 실행 ...
data = [list(range(1000)) for _ in range(100)]

snapshot = tracemalloc.take_snapshot()
top_stats = snapshot.statistics('lineno')

for stat in top_stats[:5]:
    print(stat)
# 어떤 코드 줄에서 얼마나 메모리를 할당했는지 확인 가능
```

#### 4.4 성능 관련 실무 팁

**1. 내장 함수/C 확장을 활용하라**

<pre class="language-python"><code class="lang-python"><strong># 느림 — 순수 파이썬 루프
</strong>total = 0
for i in range(1000000):
    total += i

# 빠름 — C로 구현된 내장 함수
total = sum(range(1000000))
</code></pre>

**2. 지역 변수가 전역 변수보다 빠르다**

```python
# PVM 관점에서:
# 전역 변수: LOAD_GLOBAL → 딕셔너리 조회 (해시)
# 지역 변수: LOAD_FAST → 배열 인덱스 접근 (O(1))

x = 10  # 전역

def func():
    y = 10  # 지역 — LOAD_FAST로 접근, 더 빠름
```

**3. 불필요한 순환 참조를 피하라**

```python
# 나쁜 예: 순환 참조 생성
class Parent:
    def __init__(self):
        self.child = Child(self)

class Child:
    def __init__(self, parent):
        self.parent = parent  # Parent ↔ Child 순환 참조

# 좋은 예: weakref 사용
import weakref

class Child:
    def __init__(self, parent):
        self.parent = weakref.ref(parent)  # 약한 참조 → 참조 카운트 증가 안 함
```

**4. `__slots__`로 메모리 절약**

파이썬에서 일반 클래스의 인스턴스를 만들면, 각 인스턴스마다 `__dict__` 라는 딕셔너리가 자동으로 생성된다. \
이 `__dict__`는 딕셔너리이기 때문에 해시 테이블 구조를 가지고, 속성을 **동적으로 자유롭게 추가/삭제**할 수 있습니다.

* 딕셔너리 자체가 해시 테이블, 키 배열, 값 배열 등을 유지해야 하기 때문에 **메모리를 많이 먹습니다.**

`__slots__`를 선언하면 "이 클래스의 인스턴스는 이 속성만 가질 수 있다"고 미리 고정하는 것입니다.&#x20;

* 속성 이름과 위치가 컴파일 시점에 결정되니, 해시 테이블이 필요 없고 가볍다.&#x20;

```python
# 일반 클래스: 인스턴스마다 __dict__ 딕셔너리 생성 (무거움)
class PointNormal:
    def __init__(self, x, y):
        self.x = x
        self.y = y

# __slots__ 사용: __dict__ 대신 고정 크기 배열 사용 (가벼움)
class PointSlots:
    __slots__ = ('x', 'y')
    def __init__(self, x, y):
        self.x = x
        self.y = y

import sys
print(sys.getsizeof(PointNormal(1, 2)))  # ~152 바이트
print(sys.getsizeof(PointSlots(1, 2)))   # ~56 바이트
```

#### 4.5 파이썬 인터프리터의 진화

<table><thead><tr><th width="133">버전</th><th>주요 변화</th></tr></thead><tbody><tr><td><strong>3.11</strong></td><td>특화 적응형 인터프리터 (PEP 659) — 10~60% 성능 향상</td></tr><tr><td><strong>3.12</strong></td><td>immortal objects — 자주 쓰이는 객체의 참조 카운팅 오버헤드 제거</td></tr><tr><td><strong>3.13</strong></td><td>실험적 JIT 컴파일러 (PEP 744) + 실험적 Free-threading (PEP 703)</td></tr></tbody></table>

파이썬은 "느린 언어"라는 꼬리표를 떼기 위해 빠르게 진화하고 있습니다. 특히 JIT 컴파일러의 도입은 JVM이 걸어온 길을 파이썬도 따라가기 시작했다는 신호입니다.

***

## 전체 정리&#x20;

### CPython vs JVM 아키텍처

| 개념          | 자바 (JVM)                   | 파이썬 (CPython)            |
| ----------- | -------------------------- | ------------------------ |
| 소스 → 바이트코드  | `javac` (명시적 컴파일)          | CPython 컴파일러 (암묵적)       |
| 바이트코드 파일    | `.class`                   | `.pyc` (`__pycache__`)   |
| 실행 엔진       | JVM (인터프리터 + JIT)          | PVM (인터프리터)              |
| VM 유형       | 스택 기반                      | 스택 기반                    |
| 타입 검사       | 컴파일 타임 (정적)                | 런타임 (동적)                 |
| 메모리 관리 주 방식 | Tracing GC                 | 참조 카운팅                   |
| 순환 참조 처리    | GC가 자동 처리                  | 세대별 GC (보조)              |
| 정수 캐싱       | `-128 ~ 127`               | `-5 ~ 256`               |
| 문자열 인터닝     | String Pool                | 식별자 형태 문자열               |
| 작성 언어       | C/C++                      | C                        |
| JIT 컴파일     | 성숙 (HotSpot, C1/C2, Graal) | 실험적 (3.13\~, PEP 744)    |
| GIL         | 없음                         | 있음 (Free-threading 실험 중) |

### CPython 실행 흐름 요약

```
.py 소스 코드
    ↓ 토크나이징
토큰 스트림
    ↓ 파싱
AST (추상 구문 트리)
    ↓ 컴파일
Code Object (바이트코드 + 메타데이터)
    ↓ 캐싱 (.pyc)
    ↓ 실행
PVM (디스패치 루프)
    ├── Frame Object 생성 (실행 컨텍스트)
    ├── 스택에서 push/pop으로 연산
    ├── 객체 생성 시 참조 카운트 = 1
    ├── 참조 소멸 시 카운트 감소 → 0이면 즉시 해제
    └── 순환 참조는 세대별 GC가 주기적으로 정리
```

***

### 마치며

"파이썬은 인터프리터 언어이니까 느리다"는 반만 맞는 말입니다. 파이썬도 바이트코드로 컴파일하고, 가상 머신이 실행하며, 메모리를 자동 관리합니다. JVM과 놀라울 정도로 유사한 구조를 가지고 있습니다.

차이가 나는 지점은 명확합니다.

* **JIT 컴파일러의 부재** (3.13부터 실험 도입 중)
* **동적 타입으로 인한 런타임 오버헤드** (특화 인터프리터로 완화 중)
* **GIL로 인한 멀티스레드 제약** (Free-threading으로 해결 시도 중)

하지만 파이썬의 강점은 이 구조적 "단점"을 C 확장, `asyncio`, `multiprocessing` 등의 우회로로 극복해왔다는 점이고, 3.11 이후 인터프리터 자체의 성능이 급격히 개선되고 있다는 점입니다.

CPython 인터프리터의 내부를 이해하면, "왜 이 코드가 느린지", "왜 `with` 문을 써야 하는지", "왜 `is`와 `==`가 다른지"를 깊이 있게 이해할 수 있습니다. 그리고 이 이해가 더 나은 파이썬 코드를 작성하는 토대가 됩니다.
