# 예외(Exception)

#### C++ 의 예외는 어디에서 올까?

* 언어 자체에서 오는 예외는 없다.\
  \-> c++ 의 예외는 프로그래머가 만든다.&#x20;
* C++ 표준 라이브러리가 많은 예외를 던지기도 한다.
* 하지만, Java 나 C# 에서 있는 당연한 예외가 C++ 에는 없다.

## 예외 발생 사례

* **대부분의 경우에는 상당히 불필요하다.**&#x20;
* **생성자 부분에서는 논리적으로 상당히 필요하다!**\
  **-> 예외에 대한 상황을 코드로 컨트롤 할 수 없다..** \
  _**(생성자에서 동적할당을 할 때, 메모리가 부족해 NULL 을 받는다면? -> 예외처리 밖에 방법이 없다)**_
* 하지만, c++ 은 성능을 중요시하는 업계! 예외처리는 기본적으로 느리기 때문에, 예외처리 기능을 끈다.. \
  \-> 그 외에도 기본적으로 생성자에서 메모리 부족으로 발생하는 문제를 예외처리 한다고 해서 해결할 수가 없다..&#x20;

### 범위(range) 이탈

* 다음과 같이 try catch 를 사용해 해당 예외에 대한 처리를 적절하게 해줄 수 있다.&#x20;
* **하지만 예외처리한 코드에 대한 어셈블리어를 보게 되면, 그렇지 않은 어셈블리어에 비해서 코드량이 많다.** \
  **-> 예외처리는 공짜가 아니다.. 비용이 든다.**&#x20;

<figure><img src="../../.gitbook/assets/image (75).png" alt=""><figcaption><p>범위 이탈</p></figcaption></figure>

* 또한 다음의 경우에서 볼 수 있듯이, 예측이 가능한 부분이라면, 예외처리를 하지 않고 분기문으로 처리가 가능하다. \
  \-> 엄격하게 말할 때, **예외는 프로그래머가 예측할 수 없어야 한다.** \
  **(해당 사례는 충분히 예측이 가능하기에 예외라고 부르기에 애매하다;;)**

<figure><img src="../../.gitbook/assets/image (81).png" alt=""><figcaption></figcaption></figure>

### 0으로 나누기&#x20;

* 정수를 0으로 나누었을 때 예외가 발생하는데, 이 예외는 c++ 의 예외가 아닌 **OS 단의 예외**이다. \
  **-> OS 단의 예외는 어플리케이션 단에서 발생하는 것이 아닌, OS 단에서 발생하는 것이기 때문에, 속도가 느릴 수 밖에 없다.**&#x20;

<figure><img src="../../.gitbook/assets/image (82).png" alt=""><figcaption></figcaption></figure>

* 다음과 같이 예측이 가능한 부분이라면, 예외처리를 하지 않고 분기문으로 처리가 가능하다.&#x20;

<figure><img src="../../.gitbook/assets/image (83).png" alt=""><figcaption></figcaption></figure>

### NULL 개체 사용

* null 개체를 사용할 때 발생하는 오류도 c++ 의 오류가 아닌, OS 단의 예외이다. \
  \-> 없는 메모리를 참조한다고 하니 OS 에서 예외를 발생시킨다.&#x20;

<figure><img src="../../.gitbook/assets/image (84).png" alt=""><figcaption></figcaption></figure>

* 다음과 같이 예측이 가능한 부분이라면, 예외처리를 하지 않고 분기문으로 처리가 가능하다.&#x20;

<figure><img src="../../.gitbook/assets/image (85).png" alt=""><figcaption></figcaption></figure>

### 생성자

<figure><img src="../../.gitbook/assets/image (87).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (88).png" alt=""><figcaption></figcaption></figure>

## OS 예외 vs C++ 예외

<figure><img src="../../.gitbook/assets/image (86).png" alt=""><figcaption></figcaption></figure>
