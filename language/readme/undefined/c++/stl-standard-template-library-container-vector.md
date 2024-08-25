# STL(Standard Template Library) 컨테이너(Container) - Vector

## STL 컨테이너의 목적

* **모든 컨테이너에 적용되는 표준 인터페이스(iterator)**\
  **-> 극단적인 OOP 는 문제가 될 수 있다.**&#x20;
* **std 알고리듬은 많은 컨테이너에서 작동**&#x20;
* **템플릿 프로그래밍 기반**&#x20;
* _**메모리 자동 관리**_\
  _**-> 가장 주요한 특징, 하지만 메모리 파편화는 문제가 될 수 있다.**_&#x20;

## STL 컨테이너 문제점

#### 아래와 같은 STL 의 문제점으로 회사 내부에서 STL 대체품들을 만들기도 한다.&#x20;

\-> EA, Epic Games ..&#x20;

### **문제점1. 모든 컨테이너에 알맞은 표준 인터페이라는 환상..!**

* 모든 컨테이너에 같은 인터페이스가 적용되는 것은 이상하다. \
  \-> 극단적으로 OOP 를 추구한 사례 \
  \-> 모든 컨테이너가 동작하는 방법이 약간은 다른데, 이것을 메서드 명으로 파악하기가 어렵다..

<pre class="language-cpp"><code class="lang-cpp"><strong>// 아래 두개의 컨테이너는 동작하는 방식이 다른데, 메서드 명이 같다 ;; 
</strong><strong>std::vector&#x3C;int> scores;
</strong>scores.push_back(10);

std::list&#x3C;int> ages;
ages.push_back(100);
</code></pre>

### 문제점2. 메모리 단편화

* **빈번한 메모리 재할당은 메모리 단편화를 초래함**&#x20;
* **메모리 단편화는 엄청난 문제가 될 수 있다!**\
  **-> 특히 가상 메모리를 지원하지 않는 플랫폼에서 프로그램을 실행할 때!**
* **메모리 단편화 때문에, 애플리케이션은 뻗어 버릴 수 있다.** \
  **-> 즉, 총 여유 공간은 충분하나 충분히 큰 연속되는 메모리가 없는 경우!**
* 디버깅 및 고치는게 쉽지 않음 ..

## Vector&#x20;

* **어떤 자료형**도 넣을 수 있는 **동적 배열 (자동으로 늘려준다, string 같이)**
* **그 안에 저장된 모든 요소들이 연속적인 메모리 공간에 위치** \
  **-> Vector 는 메모리를 자동적으로 관리해주는 배열로 정리할 수 있다!**
* **배열이기 때문에, **_**인덱스만 알고 있다면 어떤 요소에도 임의로 접근할 수 있다.**_&#x20;

### Vector 만들기

* reserve() 함수 사용 시, 용량이 증가해야 한다면 새로운 저장 공간을 재할당하고 기존 요소들을 모두 새 공간으로 복사한다. \
  \-> 예를 들어서 **사용해야할 공간이 500byte 라면 미리 공간을 할당하면 **_**string 에 비해서 메모리를 재할당하는 횟수가 적어진다.**_\
  **-> 해당 과정을 컴파일러가 최적화해준다.** \
  **(vector 생성 후 바로 500byte 를 할당한다면 처음부터 500byte 를 할당해준다)**

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 12.13.40.png" alt=""><figcaption></figcaption></figure>

* 동일한 크기과 데이터를 같는 vector 를 생성할 수 있다. \
  \-> 복사 생성자와 동작하는 방식이 비슷하다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 12.15.16.png" alt=""><figcaption></figcaption></figure>

### Vector 요소에 접근하기

#### 순차적으로 요소에 접근하게&#x20;

* 반복문을 통해서, 요소에 접근하는 것은 벡터에서만 사용할 수 있다..
  * 맵(map) 에서는 반복문을 사용할 수 없다.&#x20;
  * 때문에, STL 컨테이너를 순회할 때는 반복자(iterator) 를 사용하는 것이 표준방식이다.&#x20;

> #### 반복자(iterator) 와 포인터의 차이점&#x20;
>
> * 포인터는 직접적으로 메모리 주소를 다루고 접근하는 데 사용되는 반면,&#x20;
> * **Iterator는 데이터 구조를 추상화하고 보다 일반적인 방식으로 순회하고 접근하는 데 사용됩니다.** \
>   \-> **포인터보다 Iterator가 더 추상화된 수준의 개념이며, (극단적인 OOP 의 단점도 존재한다)**\
>   \-> 이는 보다 유연하고 안전한 코드 작성을 가능하게 합니다.

<pre class="language-cpp"><code class="lang-cpp">// 지정된 n 의 요소를 참조로 반환한다. 
// operator[](size_t n);
scores[i] = 3;
std::cout &#x3C;&#x3C; name[i] &#x3C;&#x3C; " ";
std::cout &#x3C;&#x3C; myCats[i].GetScore() &#x3C;&#x3C; " ";

// ==========================================
<strong>#include &#x3C;iostream>
</strong>#include &#x3C;vector>

int main()
{
    std::vector&#x3C;int> scores;
    scores.reserve(2); 
    
    scores.push_back(30);
    scores.push_back(50);

    // 배열 같이 사용할 수 있다는 것은 장점이다. 
    for(size_t i=0; i&#x3C;scores.size(0); ++i)
    {
        std::cout &#x3C;&#x3C; scores[i] &#x3C;&#x3C; " ";
    }    
    ...
}

// ==========================================
#include &#x3C;iostream>
#include &#x3C;vector>

int main()
{
    std::vector&#x3C;int> scores;
    scores.reserve(2); 
    
    scores.push_back(30);
    scores.push_back(50);

    // STL 컨테이너에서 사용 가능한 iterator 를 사용하는 방법 
    for(std::vector&#x3C;int>::iterator iter=scores.begin(); i&#x3C;scores.end(); ++iter)
    {
        // 반복자는 사실상 포인터이다. 
        std::cout &#x3C;&#x3C; *iter &#x3C;&#x3C; " ";
    }    
    ...
}
</code></pre>

#### 뒤에서부터 Vector 요소에 접근하기

* begin(), end() 가 아닌 rbegin(), rend() 를 사용하면 된다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 13.08.18.png" alt="" width="563"><figcaption></figcaption></figure>

```cpp
#include <iostream>
#include <vector>

int main()
{
    std::vector<int> scores;
    scores.reserve(2); 
    
    scores.push_back(30);
    scores.push_back(50);

    // STL 컨테이너에서 사용 가능한 reverse_iterator 를 사용하는 방법 
    for(std::vector<int>::reverse_iterator iter=scores.rbegin(); i<scores.rend(); ++iter)
    {
        // 반복자는 사실상 포인터이다. 
        std::cout << *iter << " ";
    }    
    ...
}
```

### Vector 요소의 삽입과 삭제

#### 특정 위치에 요소 삽입하기&#x20;

```cpp
#include <vector>

int main()
{
    std::vector<int> scores;
    scores.reserve(2); 
    
    scores.push_back(30);
    scores.push_back(50);
    
    std::vector<int>::iterator begin = scores.begin();
    begin = scores.insert(begin, 100);
    ...
}
```

#### 특정 위치에 요소 삭제하기

```cpp
#include <vector>

int main()
{
    std::vector<int> scores;
    scores.reserve(10); 
    
    scores.push_back(10);
    scores.push_back(20);
    scores.push_back(30);
    scores.push_back(40);
    scores.push_back(50);
    
    std::vector<int>::iterator begin = scores.begin();
    begin = scores.erase(it);
    ...
}
```

### 하지만 Vector 는 메모리 복사 && 재할당 문제가 발생한다.(배열의 고질적인 문제)

* **메모리 복사 : 삽입되는 인덱스 뒤에 데이터는 모두 복사 되어야 한다.**&#x20;
*   **메모리 재할당 : 할당된 메모리보다 데이터를 더 집어넣을 때 메모리를 재할당 받고 기존 데이터를 옮긴다.**&#x20;

    <figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 13.17.26.png" alt=""><figcaption></figcaption></figure>

### Vector 요소 교환하기

* **swap 함수는 각각의 vector 의 주소만 변경하면 되기 때문에, 무거운 연산은 아니다!**

```cpp
#include <vector>

int main()
{
    std::vector<int> scores;
    scores.reserve(2); 
    
    scores.push_back(30);
    scores.push_back(50);
    
    std::vector<int> anotherScores;
    anotherScores.assign(7,100);
    
    scores.swap(anotherScores);
}
```

### 개체(object) 백터&#x20;

* **이전에 c++ 은 개체 배열 생성 시 각 요소 마다 주소값이 아닌 개체 멤버변수를 저장한다고 했다.**&#x20;
* **백터 또한 내부적으로는 배열로 구현되어 있기 때문에, 요소마다 주소값이 아닌 멤버변수를 저장한다.**&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 13.37.17.png" alt=""><figcaption></figcaption></figure>



### 개체(object) 포인터 백터&#x20;

* 백터에 개체를 직접 저장 시 다음과 같이 2가지 문제가 발생한다. \
  \-> 개체 배열은 값을 직접 저장한다는 특정 때문이다.&#x20;
  * 사이즈 보다 개체를 많이 저장해야 할 시 기존 데이터를 모두 복사해야 한다. \
    (개체의 일반적으로 사이즈는 크다..)
  * 백터의 사본을 만들 때 기존에 데이터를 모두 복사해야 한다. \
    (개체의 일반적으로 사이즈는 크다..)

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 13.48.37.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-04-11 13.52.03.png" alt=""><figcaption></figcaption></figure>

#### 때문에, 개체의 크기가 크다면 포인터를 저장해야 한다. (Java 의 방식!)

* **개체 포인터 백터를 모두 사용했다면, delete 를 호출해서 메모리를 반납해주어야 한다!**

```cpp
int main 
{
    vector<Scores*> scores;
    
    scores.push_back(new Score(30, "c++"));
    scores.push_back(new Score(80, "Java"));
    scores.push_back(new Score(70, "c#"));
    
    for(vector::<Scores*>::interator iter = scores.begin(); iter != scores.end(); ++iter)
    {
        delete *iter;
    }
    scores.clear
    
    return 0; 
}
```
