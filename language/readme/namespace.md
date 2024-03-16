# namespace

## namespace&#x20;

* 말 그대로 이름 공간이다. 함수의 이름이 겹치는 불상사를 막기 위해서, namespace 안에 함수를 정의하고 해당 함수를 호출하기 위해서 namespace 를 통해 접근할 수 있도록 한다.
* 다음 예제를 보자 \
  \-> 연산자 `::` 를 가리켜 '범위지정 연산자 (scope resolution operator)' 라 부른다.

```cpp
#include <iostream>

namespace BestComImpl
{
    void SimpleFunc(void)
    {
        std::cout<<"BestCom 이 정의한 함수"<<std::endl;
    }
}

namespace ProgComImpl
{
    void SimpleFunc(void)
    {
        std::cout<<"ProgCom 이 정의한 함수"<<std::endl;
    }
}

int main(void)
{
    BestComImpl::SimpleFunc();
    ProgComImpl::SimpleFunc();
    return 0;
}
```

## using

* `using` 키워드를 통해서 namespace 를 작성하지 않고 사용할 수 있다.&#x20;
* 다음 예제를 보자

```cpp
#include <iostream>

namespace BestComImpl
{
    void SimpleFunc1(void)
    {
        std::cout<<"BestCom 이 정의한 함수"<<std::endl;
    }
}

namespace ProgComImpl
{
    void SimpleFunc2(void)
    {
        std::cout<<"ProgCom 이 정의한 함수"<<std::endl;
    }
}

using BestComImpl;
using ProgComImpl;
    
int main(void)
{
    SimpleFunc1();
    SimpleFunc2();
    return 0;
}
```
