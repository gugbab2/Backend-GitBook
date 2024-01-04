# 출력(Output)

## 1. 입출력을 다루기 전에 스트림과 버퍼에 대해서 생각해보자

#### 스트림(stream)

* c++ 프로그램은 파일이나, 콘솔에서 직접 입출력을 다루지 않고, 스트림(stream) 이라는 흐름을 통해서 다루게 된다.&#x20;
* 스트림은 실제 입력이나 출력이 표현된 추상화된 흐름을 의미한다.&#x20;
* 즉 스트림은 운영체제에 의해 생성되는 가상의 연결고리를 의미하고, 중간 매개자 역할을 한다.

<figure><img src="../../.gitbook/assets/image (3) (1) (1) (1).png" alt="" width="290"><figcaption></figcaption></figure>

#### 버퍼(buffer)

* 스트림은 내부에 버퍼(buffer) 라는 임시 메모리 공간을 가지고 있다.&#x20;
* 버퍼를 사용하면 다음과 같은 장점이 있다.&#x20;
  * 문자를 하나씩 전달하는 것이 아닌, 묶어서 한번에 전달하기에, 전송 시간이 적게 걸린다는 장점이 있다. \
    (전달에는 자원이 소모된다는 것을 잊지말자)
  * ~~사용자가 문자를 잘못 입력했을 경우 수정할 수 있다.~~
* 하지만 즉각적인 반응이 요구되는 시스템에서 버퍼를 사용하는 것은 좋지 않을 수 있다.\
  (상황에 맞게 버퍼 사용 여부를 판단해야 한다)

## 2. 출력의 기본

#### 모든 언어의 시작은 "Hello, World" 출력에서 시작된다.

* 다음과 같은 형식으로 출력하는 것을 볼 수 있다. &#x20;

```cpp
std::cout << "Hello" << ", World" << std::endl;
```

* cout 은 출력하는 것 같고, endl 은 한줄을 마무리할 때 \n 의 의미를 지니는 것 같다.

#### endl 과 \n 의 차이는 무엇인가?

* endl 은 flush 를 하고, \n 은 flush 를 하지 않는다. \
  (flush : 버퍼에 남아있는 데이터를 다 내보낸다)
* 기본적으로 stream 은 입력을 받은 것을 바로 출력하는 것이 아니라, buffer 에 담아둔다.&#x20;
* flush 를 하지 않을시 개발자가 의도하지 않은대로 출력될 수 있다.&#x20;

#### 그럼 std:: 는 무엇일까?&#x20;

* std:: 는 c++ 에서는 namespace 라고 부르며 java 에서 package 와 같다.\
  \-> 클래스명, 메서드명, 변수명 등 겹치는 것을 방지해준다.
* 다음 예를 살펴보자\
  \-> 같은 메서드명을 사용하더라도 컴파일 에러가 발생하지 않는다.&#x20;

<pre class="language-cpp"><code class="lang-cpp">// C
// Hello1.h
void sayHello();

// Hello2.h
void sayHello();

#include "Hello1.h"
#include "Hello2.h"

sayHello(); // 컴파일 오류(실행 이전에 발생하는 오류 : 가장 좋은 오류이다!)

// -------------------------
// C++

<strong>namespace hello{
</strong>    void sayHello();
}

namespace hi{
    void sayHello();
}

#include "hello.h"
#include "hi.h"

hello::sayHello();
hi::sayHello();
</code></pre>

#### using 은 무엇일까?

* 실제 코드 작성시 namespace:: 를 사용하지 않도록 하는 방법으로 반복되는 코드 사용을 줄여준다.

```cpp
using namespace std;

cout << "Hello" << ", World" << endl;
```

#### #pragma once 는 무엇일까?

* 해당 파일을 중복되지 않고 한번만 사용되도록 하는 문법으로 c++ 에서는 주로 헤더파일에서 사용한다.&#x20;

```cpp
#pragma once

namespace hello
{
	void SayHelloExample();
}
```

#### c++ 에서는 출력에 옵션을 지정해주는 조정자(Manipulator) 가 있다.&#x20;

* 다음 예제를 통해 조정자를 살펴보자

```cpp
number = 123;

cout << showpos  << number;    // +123
cout << noshowpos << number;    // 123

cout << dec << number;    // 123(10진수)
cout << hex << number;    // 7b(16진수)
cout << oct << number;    // 173(8진수)

cout << uppercase << hex << number;    // 7B
cout << uppercase << hex << number;    // 7b

cout << showpos << uppercase << hex << number;    // 0x7B
cout << noshowpos << uppercase << hex << number;    // 7b

number = -123
cout << setw(6) << left << number;        // |-123   |
cout << setw(6) << internal << number;    // |-   123|
cout << setw(6) << right << number;       // |   -123|

decimal1 = 100.0;
decimal2 = 100.12;
cout << noshowpoint << decimal1 << " " << decimal2 << decimal    // 100 100.12
cout << showpoint << decimal1 << " " << decimal2 << decimal      // 100.000 100.120

number = 123.456789;
cout << fixed << number;         // 123.456789
cout << scientific << number;    // 1.234568E+02

bReady = true;
cout << boolalpha << bReady;        // true
cout << noboolalpha << bReady;      // 1

// #include <iomanip> 를 상단에 정의해주어야 한다!
number = 123;
cout << setw(5) << number;                    // |  123|
cout << setfill('*') << setw(5) << number;    // **123

number = 123.456789
cout << setprecision(7) << number;            // 123.4567
```
