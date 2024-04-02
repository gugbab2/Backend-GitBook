# static 키워드

#### static 키워드

* 범위(scope)의 제한을 받는 전역 변수\
  \-> **전역변수 :  .exe 파일 내 하나만 존재하는 변수**
* 어떤 범위?
  * **파일 속**
  * **네임스페이스 속**
  * **클래스 속**
  * **함수 속**

#### 전역변수(extern 키워드)

* extern 키워드는다른 파일의 전역변수에 접근을 가능하게 해준다.
* 아래 예제를 살펴보자
  * 선행처리기에 의해 # include ExternTest.h 가 선언 된 파일에 ExternTest.h 의 내용을 그대로 복붙한다.&#x20;
  * 컴파일러에 의해 ExternTest.c / Main.c 파일 각각 obj 파일을 만들게 된다.\
    \-> 각각 obj 파일을 가지고 있기 때문에, 서로의 주소값을 모른다.   \
    \-> **때문에, globalValue 의 주소는 빈 값으로 가지고 있는다.**
  * **링커에 의해 obj 파일을 합치고 실행 파일을 만들 때 해당 빈 주소값을 extern 키워드가 선언된 주소로 채워준다.**

```cpp
// ExternTest.h
extern int globalValue;

void IncreaseValue();

// ExternTest.c
#include "ExternTest.h"

int globalValue = 2;
void IncreaseValue()
{
    globalValue++;
}

// Main.c
#include "ExternTest.h"    // 해당 파일을 COPY

int main()
{
    printf("%d", globalValue);
}
```

## C 스타일 static

### 파일 속 정적 변수

* 이전과 달리 StaticTest.cpp 파일 내 `static int globalValue = 2;`  변수에 static 키워드가 붙은 것을 알 수 있는데, 이는 변수의 범위를 파일 내로 한정하게 된다.&#x20;
* 때문에, 다른 파일에서 해당 변수를 사용하고자 하면 링커에서 에러를 발생한다.

```c
// StaticTest.h
extern int globalValue;

void IncreaseValue();

// StaticTest.cpp
#include "ExternTest.h"

static int globalValue = 2;
void IncreaseValue()
{
    globalValue++;
}

// Main.c
#include "ExternTest.h"    // 해당 파일을 COPY

int main()
{
    // 링커(linker) 에러
    printf("%d", globalValue);
}
```

### 함수 속 정적 변수

* 함수 안에 정적 변수를 선언 할 경우, 해당 정적 변수는 함수 안에 범위를 제한하게 된다.\
  \-> 메모리는 반납 되지 않고 계속 사용된다.\
  \-> Java 에는 함수 속 정적 변수가 없다. (오직 정적 멤버 변수만 있다)

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

## C++ 스타일 static&#x20;

### 정적 멤버 변수

* 개체가 만들어지기 전 메모리가 할당되는 변수로, 보통의 C++ 에서는 .h 파일에서 선언을 하고 .cpp 파일에서 초기화를 한다.
* _**클래스당 하나의 COPY 만 존재**_\
  \-> 때문에, 개체의 메모리 레이아웃이 아닌, 클래스 메모리 레이아웃에 포함되어 있다.
* exe 파일 안에 메모리가 잡혀있다.\
  \-> 개체마다 생겨나는 메모리가 아닌 클래스 하나에 있는 메모리이기 때문에, 한번 메모리를 할당하면 이후에 할당할 일이 없다.\
  \-> ONLY ONE!

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

#### 정적 멤버 변수 BEST PRACTICE

* 함수 안에서 정적 변수를 사용하지 말아라! \
  \-> 함수 안에 정적 변수나 클래스 안에 정적 멤버 변수나 용도의 큰 차이가 없다..&#x20;
  * 클래스 안에 **정적 멤버 변수 형태로 사용해라!** \
    **(가독성 향상 : 헤더파일만 읽으면 용도 파악이 가능하다!)**
* 전역변수 대신 정적 멤버 변수를 사용해라!
  * scope 를 제한하기 위해서!

### 정적 멤버 함수

* 논리적인 범위(scope) 에 제한 된 전역 함수
* **해당 클래스의 정적 멤버 변수에만 접근 가능**\
  **-> 정적 멤버 함수는 개체가 생성되기 이전에 메모리를 할당 받기 때문에,** \
  **정적 멤버 함수에 개체 생성된 후 메모리를 할당 받는 멤버 변수가 사용된다는 것은 논리가 맞지 않는다.**&#x20;
* **개체 생성과 관련이 없기 때문에, 개체 없이도 정적 함수를 호출할 수 있음**\
  **`Math::Square(10);`**
* 정적 멤버 함수는 헤더  파일에서 static 을 사용하면구현체에서는 사용할 필요가 없다.&#x20;
* 다음 예제는 컴파일 에러가 난다!

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>
