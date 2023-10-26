# 초기 C++ 새로운 기능(bool 타입, Reference)

## 1. C 에서 없던 bool 데이터 형이 생겨났다.&#x20;

```cpp
if(IsStudent() == false)
{

}

if(IsStudent() == false)
{

}
```

## 2. 참조(Reference) 는 매우 중요한 개념이다!

#### 참조&#x20;

* 포인터를 사용하는 좀 더 안전한 방법
* 하지만 Java 처럼 제한적이지는 않음
*   아래 내용을 먼저 살펴보자

    * 값에 의한 호출(call by value)

    <figure><img src="../../.gitbook/assets/스크린샷 2023-10-24 22.13.01.png" alt="" width="375"><figcaption></figcaption></figure>

    *   참조에 의한 호출(call by reference)

        * c / c++\
          \-> c / c++ 는 포인터를 제공하지 않으면 call by value 처럼 동작하게 된다.&#x20;

        <figure><img src="../../.gitbook/assets/스크린샷 2023-10-24 22.16.23.png" alt="" width="375"><figcaption></figcaption></figure>

        <figure><img src="../../.gitbook/assets/스크린샷 2023-10-24 22.19.46.png" alt="" width="375"><figcaption></figcaption></figure>

        *   Java\
            \-> Java 는 primitive 타입이 아닌 것(reference 타입)에 대해서는 값의 주소를 제공하기 때문에, c / c++ 과 헷갈리는 부분이다.  \
            \-> **해당 주소는 포인터 연산자가 아니다! 왜냐면 포인터 연산자처럼 주소를 바꿀수가 없다..**\
            \-> 자바는 성능보다는 안전함을 위한 언어이기 때문에, 포인터를 의도적으로 사용하지 않는다.

            <figure><img src="../../.gitbook/assets/스크린샷 2023-10-24 22.30.59.png" alt="" width="375"><figcaption></figcaption></figure>
    * 포인터(pointer)
      *   c / c++ 에서 포인터 1을 더하는 연산을 할 때, 값이 추가되는 것이 아닌 해당 포인터를 dereferencing 했을 때 size 만큼 추가하게 된다.&#x20;

          <figure><img src="../../.gitbook/assets/스크린샷 2023-10-24 22.34.33.png" alt="" width="375"><figcaption></figcaption></figure>

## 3. C++ 의 참조(reference)는 위와 같이 복잡한 방법에 대한 해결책이다. &#x20;

#### 참조는 별칭이다!&#x20;

\-> `int& reference = number`\
`int* pointer = &number` 는 완전 다른 의미이기 때문에, 주의하자!

<pre class="language-cpp"><code class="lang-cpp">int number = 100;
int&#x26; reference = number;

// NULL 이 될 수 없다!
int&#x26; reference = NULL;    // error

// 초기화 중 반드시 선언되어야 한다. 
int&#x26; reference;    // error

<strong>//---------------------------
</strong><strong>// 참조하는 대상을 바꿀 수 없다. 
</strong><strong>int number1 = 100;
</strong><strong>int number2 = 200;
</strong><strong>
</strong><strong>int&#x26; reference = number1;
</strong><strong>reference = number2;         // 참조 대상 변경이 아닌 대입 : 세 변수 모두 200이 됨
</strong></code></pre>

#### 함수 매개변수로서의 참조

* **number1, number2 변수는 NULL 이 될 수 없다. (안전한 코드!)**
* **우리가 소유하지 않은 메모리 장소를 가리킬 수 없다! (포인터 연산이 불가하다)**

```cpp
// 포인터
void swap(int* number1, int* number2)
{
    int temp = *number1;
    *number1 = *number2;
    *number2 = temp;
}

// 참조
void swap(int& number1, int& number2)
{
    int temp = number1;
    number1 = number2;
    number2 = temp;
}
```

## 4. 컴퓨터는 참조가 뭔지 알까?

* 모른다..
* 포인터와 참조는 같은 어셈블리 명령어를 생성한다.
* 참조는 오직 인간을 위한 것임
* 컴파일러는 참조를 포인터로 바꾸어 준다. 기계가 이해할 수 있도록 \
  **-> 컴퓨터는 참조는 모르지만 포인터는 안다!**
