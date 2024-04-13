# 템플릿(Template) 프로그래밍

## 템플릿이란?

* Java 와 C# 에서의 제네릭(generic) 메서드 / 클래스와 비슷&#x20;
* STL 컨테이너 또한 템플릿&#x20;
* **덕분에 코드를 자료형마다 중복으로 작성하지 않아도 됨**\
  **-> 사용되는 자료형으로 컴파일 시 자동으로 넣어준다!**

### **템플릿은 어떻게 동작할까?**&#x20;

* 템플릿을 인스턴스화할 때마다, 컴파일러가 내부적으로 코드를 생성해준다!

<figure><img src="../../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>

### 이것은 무엇을 의미할까?&#x20;

* **템플릿에 넣는 자료형 가짓수에 비례해서 exe 파일 크기 증가** \
  **-> exe 파일 크기가 증가 함에 따라 컴파일 시간도 증가하는데, 과연 c++ 를 쓰는 업계에서 그래도 될까?**
* **컴파일 에러를 검출할 수 있다.**
* **템플릿 사용을 통해서 다형성을 부여할 수 있다.**&#x20;

## 함수 템플릿&#x20;

* 코드를 자료형마다 중복으로 작성하지 않는 대표적인 예제이다.&#x20;
* 다음의 코드에서 개체+ 연산 시, 클래스에서 + 연산자를 오버로딩하지 않는다면, 컴파일 에러가 발생한다.&#x20;

<figure><img src="../../.gitbook/assets/image (15).png" alt=""><figcaption></figcaption></figure>

* 함수 템플릿을 호출할 때 타입을 생략 가능하다!\
  \-> Math 함수가 그렇게 작동한다! (내부적으로 static 으로 선언되어 있다)

<figure><img src="../../.gitbook/assets/image (16).png" alt=""><figcaption></figcaption></figure>

## 클래스 템플릿&#x20;

> #### 클래스 내부의 배열의 크기를 초기화하는 팁&#x20;
>
> * 다음과 같이 클래스 내부에 크기가 정해져 있는 배열을 선언할 때 상수를 사용하는 방법은 다음과 같다.&#x20;
>   * \#define MAX\_VALUE 3\
>     \-> 전역적으로 사용할 수 있기에 사용하지 않는다.. ~~(이게 단점?)~~
>   * const int MAX\_VALUE = 3;\
>     \-> 해당 클래스를 사용하는 cpp 파일마다 4바이트를 사용해야한다..
>   * static const int MAX\_VALUE = 3; \
>     \-> 불필요한 4바이트를 사용해야 한다. ( const int 보다는 낫다)\
>     _**-> 하지만 요즘에는 컴파일러가 좋아져서 static const variable 들은 거의 무조건 inline 을 하는 편이다.**_&#x20;
>   * **enum { MAX = 3};**\
>     **-> enum 은 그냥 상수의 이름을 붙이는 방법이다!**\
>     **-> 불필요한 4바이트를 사용하지 않아도 된다! (좋은 방법)**
>
> ```cpp
> class MyIntArray 
> {
> public : 
>    bool Add(int data);
>    MyIntArray();
> private : 
>    enum{MAX = 3};
>    int mSize;
>    int mArray[MAX];
> }
> ```

* 클래스 템플릿을 사용해 아래 코드를 컴파일 해보면 다음과 같은 오류를 보게 된다.&#x20;
* 그 이유는 다음과 같다.&#x20;
  * 컴파일러가 "Main.cpp" 을 컴파일 할 때, " MyArray.cpp" 를 못찾는다. \
    \-> "MyArray.h" 를 통해서 오직 MyArray 클래스 선언만 볼 수 있다.&#x20;
  * 따라서, 컴파일러가 MyArray\<int> 를 만들어 줄 수 없다. \
    **-> 인라인 함수에서도 동일한 문제를 발견할 수 있다..** \
    **(구현체를 모두 헤더파일로 옮겼었다)**

<figure><img src="../../.gitbook/assets/image (18).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (20).png" alt=""><figcaption></figcaption></figure>

* **해결책은 구현을 모두 헤더 파일로 옮겨야 한다.**&#x20;
* 하지만 헤더 파일에 코드가 많아지는 것은 다음의 단점이 따라온다.&#x20;
  * **실행 파일의 크기가 커진다..**&#x20;
  * **헤더 파일의 변경이 있을 시 사용하는 모든 파일을 다시 컴파일 해야 하므로, 컴파일 시간이 길어진다.**&#x20;

<figure><img src="../../.gitbook/assets/image (22).png" alt=""><figcaption></figcaption></figure>

* 함수형 템플릿에서는 타입을 생략할 수 있었지만, **클래스 템플릿은 반드시 타입을 넣어주어야 한다!**\
  **-> 함수형 템플릿은 매개변수를 통해서 타입을 추측이 가능하다.**&#x20;

<figure><img src="../../.gitbook/assets/image (23).png" alt=""><figcaption></figcaption></figure>

## 템플릿 매개변수가 2개인 경우

* template 매개변수를 2개를 받을 수 있다!

#### 괜찮은 트릭!

* 템플릿 매개변수를 통해서 외부에서 클래스 내부에 배열의 크기를 받을 수 있다.

<figure><img src="../../.gitbook/assets/image (24).png" alt=""><figcaption></figcaption></figure>

#### template 매개변수를 2개 받는 예시

<figure><img src="../../.gitbook/assets/image (25).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (26).png" alt=""><figcaption></figcaption></figure>
