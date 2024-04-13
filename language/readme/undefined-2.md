# 새로운 자료형

## nullptr

* 아래 예제에서 NULL 을 사용하면 가끔 뭔가 굉장히 이상한 일이 벌어진다.&#x20;

```cpp
// Class.h
float GetScore(const char* name);
float GetScore(int id);

// Main.cpp
Class* myClass = new Class("gugbab2");
// ...
int score = myClass->GetScore(NULL);    // 어떤 함수가 호출될까?(컴파일 에러 ;;)
```

* NULL 은 0을 정의한 것으로 NULL 포인터를 의미할 수도 있지만, int 를 의미할 수도 있다.\
  `#define NULL 0`

```cpp
int number = NULL;     // OK
int* ptr = NULL;       // OK
```

* nullptr 은 포인터 상수만을 지칭하는 키워드로 위의 문제를 해결할 수 있다. \
  `typedef decltype(nullptr) nullptr_t;`

```cpp
int anotherNumber = nullptr;     // ERROR
int* anotherPtr = nullptr;       // OK
```

#### nullptr 사용 예시

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-13 18.55.29.png" alt=""><figcaption></figcaption></figure>

## enum class

### C-스타일 Enum

* 기존 C-스타일 Enum 은 하나의 독립된 타입은 아니고, 내부적으로 컴파일러가 정수형으로 변환해주었다.&#x20;
* **때문에, 기존의 enum 은 int 형과 동일하게 사용할 수 있었다.** \
  **-> 타입 체킹을 전혀 안한다..**

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-13 19.08.03.png" alt=""><figcaption></figcaption></figure>

### enum class

* **기존과 다르게, 타입 체킹을 지원한다!**

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-13 19.11.50.png" alt=""><figcaption></figcaption></figure>

* enum class 에서 사용하는 정수형을 명시할 수 있다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-13 19.14.12.png" alt=""><figcaption></figcaption></figure>
