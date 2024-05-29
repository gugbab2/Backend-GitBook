# Lamda Expression

## 람다식 등장 배경

* 람다는 이름 없는 함수라고 정의할 수 있다.&#x20;
* 람다 이전에 std::sort 함수는 다음과 같이 함수를 정의해야만 했다. \
  \-> **다시 사용하지 않더라도 아래와 같은 형태로 사용하기 위해서는 함수를 정의해야만 했다.**\
  **(유지보수 비용이 증가한다)**

<figure><img src="../../../.gitbook/assets/image (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

* 위와 같은 문제를 해결하기 위해서 이름 없는 함수를 만들어 낼 수 있는데, 이를 lamda expression 라 부른다.&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (2).png" alt=""><figcaption></figcaption></figure>

## 람다식이란?

* 이름 없는 함수 개체&#x20;
* 내포(nested) 되는 함수

<figure><img src="../../../.gitbook/assets/image (4) (2).png" alt=""><figcaption></figcaption></figure>

### 캡처 블록&#x20;

* 람다식을 품는 범위(scope) 안에 있는 변수를 람다식에 넘겨줄 때 사용

#### 캡처의 종류&#x20;

* \[]
  * 비어 있음. 캡처하지 않음
* \=
  * 값(shallow copy)에 의한 캡처, 모든 외부 변수를 캡처함&#x20;
  * 람다식 안에서 수정할 수 없음&#x20;
* &
  * 참조(reference) 에 의한 캡처, 모든 외부 변수를 캡처함
* <변수 이름>
  * 특정 변수를 값으로 캡처
  * 람다식 안에서 수정할 수 없음&#x20;
* &<변수 이름>
  * 특정 변수를 참조로 캡처&#x20;

#### 캡처 예시&#x20;

* 예시1 : 캡처하지 않는 경우

```cpp
int main() {
    // 방법1
    // 변수로 받을 수 있다. 
    auto noCapture = []() {std::cout << "No capture example 1" << std::endl;};
    noCapture();
    
    // 방법2. 바로 실행 
    [] {std::cout << "No capture example 2" << std::endl;}();
}    
```

* 예시2 : (에러!) 외부 변수 사용하기 \
  \-> 캡처를 사용하지 않아서 외부 변수에 대한 정보를 알 수 없다.&#x20;

<pre class="language-cpp"><code class="lang-cpp"><strong>int main() {
</strong>    float score1 = 80.0f;
    float score2 = 20.0f;
    
    auto max = []() {return score1 > score2 ? score1 : score2;};    // Complie error
    
    std::cout &#x3C;&#x3C; "Max value is " &#x3C;&#x3C; max() &#x3C;&#x3C; std::endl;
    ...
}
</code></pre>

* 예시3 : 값에 의한 캡처 \
  \-> 값에 의한 캡처를 사용하므로 외부 모든 변수의 값을 사용할 수 있게 되었다. \
  \-> 값을 바꾸는 것은 불가하다.&#x20;

<pre class="language-cpp"><code class="lang-cpp"><strong>int main() {
</strong>    float score1 = 80.0f;
    float score2 = 20.0f;
    
    auto max = [=]() {return score1 > score2 ? score1 : score2;};    // success
    
    std::cout &#x3C;&#x3C; "Max value is " &#x3C;&#x3C; max() &#x3C;&#x3C; std::endl;
    ...
}
</code></pre>

* 예시4 : (에러!) 값에 의한 캡처&#x20;

```cpp
int main() {
    float score1 = 80.0f;
    float score2 = 20.0f;

    auto changeValue = [=]()
    {
        score1 = 100.0f;        // Compile error, score1 을 수정할 수 없음
    };
    
    changeValue();
    ...
}
```

* &#x20;예시5 : 참조에 의한 캡처&#x20;

```cpp
int main() {
    float score1 = 80.0f;
    float score2 = 20.0f;

    auto changeValue = [&]()
    {
        score1 = 100.0f;       
        score2 = 100.0f;  
    };
    
    changeValue();
    ...
}
```

* 예시6 : (에러) 값에 의한 특정 변수 캡처&#x20;

<pre class="language-cpp"><code class="lang-cpp"><strong>int main() {
</strong>    
    float score1 = 30.0f;
    float score2 = 20.0f;
    float score3 = 100.0f;
    
    auto printValue = [score1]()
    {
        // Compile error, score2 는 캡처하지 않았기에 사용할 수 없다. 
        std::cout &#x3C;&#x3C; score1 &#x3C;&#x3C; "/" &#x3C;&#x3C; score2 &#x3C;&#x3C; std::endl;
    };
    
    printValue();
    ...
}
</code></pre>

* 예시7 : 참조에 의한 특정 변수 캡처

```cpp
int main() {
    
    float score1 = 30.0f;
    float score2 = 20.0f;
    
    auto changeValue = [&score1]()
    {
        score1 = 100.0f;
    };
    
    changeValue();
    ...
}
```

* 예시8 : 캡처 옵션 섞기

```cpp
int main() {
    
    float score1 = 30.0f;
    float score2 = 20.0f;
    
    auto changeValue = [=, &score1]()
    {
        score1 = 100.0f;                    // 참조에 의한 캡처
        std::cout << score2 << std::endl;    // 값에 의한 캡처
    };
    
    changeValue();
    ...
}
```

### 매개변수 목록&#x20;

* 선택사항이다.&#x20;
* 빈 괄호를 생략할 수 있다. \
  \-> 하지만 가독성 측면에서 명시적으로 생략하지 않는 것이 좋다.&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (2).png" alt=""><figcaption></figcaption></figure>

#### 매개변수 예시&#x20;

* 예시1 : 두 값 더하기

```cpp
int main()
{
    int score1 = 70;
    int score2 = 80;
    
    auto add = [](int a, int b)
    {
        return a + b;
    };
    
    std::cout << add(score1, score2) << std::endl;
    ...
}
```

* 예시2 : 정렬하기&#x20;

```cpp
#include <algorithm>
#include <vector>

int main()
{
    std::vector<float> scores;
    
    scores.push_back(50.f);
    scores.push_back(88.5f);
    scores.push_back(70.f);
    
    std::sort(scores.begin(), scores.end(), [](float a, float b) {return (a>b);});
    std::sort(scores.begin(), scores.end(), [](float a, float b) {return (a<=b);});
    
    return 0;
}
```

### 지정자&#x20;

* 선택사항이다.&#x20;
* mutable
  * 값에 의해 캡처된 개체를 수정할 수 있게 한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (6) (2).png" alt=""><figcaption></figcaption></figure>

#### 지정자 예시

* 예시1 : 값에 의해 캡처된 개체를 수정&#x20;

```cpp
int main()
{
    int value = 100;
    
    auto foo = [value]() mutable
    {
        // mutable 을 통해서 값을 변경 가능하다. 
        std::cout << ++value << std::endl;
    };
    
    foo();
    ... 
}
```

### 반환형&#x20;

* 선택사항이다.&#x20;
* 반환형을 적지 않으면 반환문을 통해서 추론해준다.&#x20;

<figure><img src="../../../.gitbook/assets/image (7) (2).png" alt=""><figcaption></figcaption></figure>

#### 반환형 예시&#x20;

* 예시1 : 반환형 명시&#x20;

```cpp
int main() 
{
    auto add = [](int a, int b) -> int 
    {
        return a + b;
    }
    
    std::cout << "2 + 3 = " << add(2,3) << std::endl;
    
    return 0;
}
```

## 람다식의 장단점

### 장점&#x20;

* 간단한 함수를 빠르게 작성할 수 있다.&#x20;

```cpp
auto max = [](float a, float b){return a > b ? a : b; };
```

### 단점

* 디버깅하기 힘들어진다..\
  \-> 함수의 이름이 정해져 있지 않기 때문에, call stack 에서 바로 확인이 어렵다..

<figure><img src="../../../.gitbook/assets/image (8) (2).png" alt=""><figcaption></figcaption></figure>

* **함수 재사용성이 낮다..** \
  **-> 사람들은 보통 함수를 새로 만들기 전에 클래스에 있는 기존 함수를 찾아본다.** \
  **-> 이 과정에서 람다 함수는 잘 띄지 않아서 못 찾을 가능성이 높다.** \
  **-> 그럼 코드 중복이 발생한다.**&#x20;

## 베스트 프랙티스

* **기본적으로, 이름 있는 함수를 쓰자! (전통적 방식)**
* **자잘한 함수는 람다로 써도 괜찮다. (한 줄짜리 함수)** \
  **-> 하지만 재사용할 것 같다면 이름 있는 함수를 쓰자.**
* _**정렬 함수처럼 STL 컨테이너에 매개변수로 전달할 함수들은 람다를 사용하기 좋은 후보들이다.**_
