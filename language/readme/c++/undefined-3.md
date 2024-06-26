# 이동생성자 및 이동대입연산자

### 값의 분류&#x20;

#### lvalue

* 임시적인 지속되는 것인 아닌, 정해진 시간 동안 지속되는 값
* **일반적으로 사용되는 변수가 lvalue 이다.**&#x20;
* ex) \
  이름이 있는 변수, \
  const 변수, \
  배열 변수, \
  클래스 멤버 등..&#x20;

#### rvalue

* 임시적으로 지속되는 값
* 정의 하기가 상당히 모호하다 ..
* ex) \
  리터럴(문자열 리터럴 제외), \
  함수의 리턴 값을 변수에 저장할 때, \
  값을 넣기 전 임시적으로 저장되는 값&#x20;

#### rvalue 참조 연산자(&&)

* c++11 이후 새로 나온 연산자&#x20;
* 기존의 & 연산자는 lvalue 참조에 사용&#x20;
* && 연산자는 rvalue 참조에 사용

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-24 20.34.58.png" alt=""><figcaption></figcaption></figure>

#### std:move()

* rvalue 의 참조를 반환&#x20;
* **lvalue 를 rvalue 로 반환**\
  **-> lvalue 가 아닌, rvalue 를 사용한다는 것은 메모리 복사가 필요 없다는 것을 뜻한다.**&#x20;

## C++ 11 이전의 문제&#x20;

#### 함수 리턴 시 메모리 복사의 문제가 발생한다!

* **메모리 사용 순서**\
  1\. scores 에서 메모리 할당\
  2\. ConvertToPercentage 함수 내 percentages 에서 메모리 할당 **-> 1차 복사**\
  3\. ConvertToPercentage 함수 리턴 시 임시 메모리 할당 **-> 2차 복사**
* **함수의 리턴 값을 변수에 저장할 때(임시값, rvalue), 두 번의 복사가 일어난다!**\
  \-> 값의 크기가 크면 클 수록 문제는 크다.. \
  \-> **최근에는 컴파일러가 이러한 문제가 일어나지 않도록 동작한다.**&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-24 20.16.38.png" alt=""><figcaption></figcaption></figure>

## 이동생성자

### 복사 생성자&#x20;

* 복사 생성자를 사용했을 때는 내부적으로 깊은 복사가 일어나 원본과 복사본 모두 메모리를 가지고 있는 것을 볼 수 있다. \
  \-> 말 그대로 메모리가 복사된다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-24 20.39.33.png" alt=""><figcaption></figcaption></figure>

### 이동 생성자

* **이동 생성자를 사용했을 때는 다른 개체의 맴버 소유권을 가져오게 된다.**&#x20;
* 복사 생성자와 달리 메모리 재할당을 하지 않는다.&#x20;
* 메모리 재할당을 하지 않기 때문에, 복사 생성자보다 약간 빠를 수 있다.
* **약간 얕은 복사와 비슷하다.** \
  **-> 원본의 값을 유지하지 않는다는 점에서 얕은 복사와 다르다..** \
  **(원본의 값을 유지하지 않기 때문에, 생성자의 매개변수가 const 가 아니다!)**

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-24 20.46.24.png" alt=""><figcaption></figcaption></figure>

## 이동 대입 연산자

* **이동 생성자와 정말 같은 개념이다.**&#x20;
  * 메모리 복사를 하지 않는다.&#x20;
  * 얕은 복사와 비슷하게 동작한다.&#x20;
* 다른점은 다음과 같다.&#x20;
  * **생성자는 새로 만들려는 개체의 멤버변수 값을 초기화 하는 과정이라면, 대입 연산자는 이미 개체의 멤버변수 값이 있고 그 값을 초기화하는 과정이다.**&#x20;
  * **때문에, 자신의 멤버변수 중 동적할당이 된 멤버변수가 있다면, 메모리를 수거 한 이후에 초기화해주어야 한다.**&#x20;
  * **자기 자신에 이동 대입연산자를 사용하는 경우 아무런 연산도 필요 없다..**

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-24 21.04.33.png" alt=""><figcaption></figcaption></figure>

## STL 컨테이너용 이동 문법&#x20;

* **C++11 이후로 STL 컨테이너에 이동생성자와 이동 대입 연산자가 생김**&#x20;
* 그래서, 그것들을 위해서 따로 구현할 필요가 없다.&#x20;

## ravalue 최적화에 대한 평가&#x20;

* **이동생성자, 이동 대입 연산자에 한정하여 성능이 좋다. (멤버변수 중 동적할당이 이루어지는 경우에 한정)**\
  \-> 힙 메모리에서 값 복사가 이루어지지 않는 것은 성능이 좋다.&#x20;
* **하지만, 포인터 대신 개체 자체를 반환하는 함수에서 rvalue 를 반환하는 것은 실제로 매우 느리다.** \
  **-> 반환 값 최적화(return value optimization) 최적화를 깨뜨린다.**&#x20;
* **BEST PRACTICE**
  * **기본적으로 개체를 반환하자!**
  * **더 빨라진다고 입증된 경우에만 rvalue 를 반환하도록 바꾸자!**
