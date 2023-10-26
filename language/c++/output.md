# 출력(Output)

#### 모든 언어의 시작은 "Hello, World" 출력에서 시작된다.

* 다음과 같은 형식으로 출력하는 것을 볼 수 있다. &#x20;

```cpp
std::cout << "Hello" << ", World" << std::endl;
```

* cout 은 출력하는 것 같고, endl 은 한줄을 마무리할 때 \n 의 의미를 지니는 것 같다.

#### endl 과 \n 의 차이는 무엇인가?

* endl 은 flush 를 하고, \n 은 flush 를 하지 않는다.&#x20;
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
