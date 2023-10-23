# 출력(Output)

#### 모든 언어의 시작은 "Hello, World" 출력에서 시작된다.

* 다음과 같은 형식으로 출력하는 것을 볼 수 있다. &#x20;

```cpp
std::cout << "Hello" << ", World" << std::endl;
```

* cout 은 출력하는 것 같고, endl 은 한줄을 마무리할 때 \n 의 의미를 지니는 것 같다.

#### 그럼 std: 는 무엇일까?&#x20;

* std: 는 c++ 에서는 namespace 라고 부르며 java 에서 package 와 같다.\
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
