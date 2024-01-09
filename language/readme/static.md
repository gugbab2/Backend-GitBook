# static 키워드

#### static

* 범위(scope)의 제한을 받는 전역 변수
* 어떤 범위?
  * 파일 속
  * 네임스페이스 속
  * 클래스 속
  * 함수 속

#### extern 키워드

* 다른 파일의 전역변수에 접근을 가능하게 해준다.
* 아래 C 언어 예제를 살펴보자
  * 선행처리기를 통해서 ExternTest.h 파일을 가져온다.
  * 컴파일러에 의해 ExternTest.c / Main.c 파일 각각 obj 파일을 만들게 된다.\
    \-> 원래라면, 각각 obj 파일을 가지고 있기 때문에, 서로의 변수의 주소값은 알 수가 없다.\
    \-> 이때, globalValue 의 주소는 빈 값으로 가지고 있는다.
  * 링커에 의해서 obj 파일을 합치고 실행파일을 만들 때 해당 빈 주소값을 extern 키워드가 선언된 주소로 채워준다.

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

#### C++ 함수 속 정적 변수

* 함수 안에 정적 변수를 선언 할 경우, 해당 정적 변수는 함수 안에 범위를 제한하게 된다.\
  \-> 메모리는 반납되지 않고 계속 사용된다.\
  \-> Java 에는 함수 속 정적 변수가 없다. (오직 정적 멤버 변수만 있다)

<figure><img src="../../.gitbook/assets/image (70).png" alt=""><figcaption></figcaption></figure>

#### 정적 멤버 변수

* 개체가 만들어지기 전 메모리가 할당되는 변수로, 보통의 C++ 에서는 .h 파일에서 선언을 하고 .cpp 파일에서 초기화를 한다.
* _**클래스당 하나의 COPY 만 존재**_\
  \-> 때문에, 개체의 메모리 레이아웃이 아닌, 클래스 메모리 레이아웃에 포함되어 있다.
* exe 파일 안에 메모리가 잡혀있다.\
  \-> 개체마다 생겨나는 메모리가 아닌 클래스 하나에 있는 메모리이기 때문에, 한번 메모리를 할당하면 이후에 할당할 일이 없다.\
  \-> ONLY ONE!

<figure><img src="../../.gitbook/assets/image (71).png" alt=""><figcaption></figcaption></figure>

#### 정적 멤버 변수 BEST PRACTICE

* 함수 안에서 정적 변수를 사용하지 말아라! (코드의 가독성을 하락시킨다..!)
  * 클래스 안에 정적 멤버 변수 형태로 사용해라
* 전역변수 대신 정적 멤버 변수를 사용해라!
  * scope 를 제한하기 위해서!

#### 정적 멤버 함수

* 논리적인 범위(scope) 에 제한 된 전역 함수
* 해당 클래스의 정적 멤버에만 접근 가능\
  \-> 멤버 변수는 개체가 생성되기 이전에 메모리를 할당 받기 때문에, 정적 멤버 함수에 멤버 변수가 사용된다는 것은 논리가 맞지 않는다.
* 개체가 없이도 정적 함수를 호출할 수 있음\
  `Math::Square(10);`
* 다음 예제는 컴파일 에러가 난다!

<figure><img src="../../.gitbook/assets/image (74).png" alt=""><figcaption></figcaption></figure>
