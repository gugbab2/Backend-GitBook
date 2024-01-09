# 인터럽트에서 DirectX 까지

## 인터럽트

#### Interupt

* 하드웨어의 동작을 방해하는, 신호
* 컴퓨터는 기본적으로 주변기기와 함께 붙어서 작동하게 되어있다.\
  \-> 이 상호작용 관계에서 I/O 가 일어나는데, 이때 인터럽트가 발생한다.

#### 외부 인터럽트

<figure><img src="../../.gitbook/assets/image (76).png" alt=""><figcaption></figcaption></figure>

#### 내부 인터럽트

<figure><img src="../../.gitbook/assets/image (77).png" alt=""><figcaption></figcaption></figure>

## 인터럽트 동작 순서

<figure><img src="../../.gitbook/assets/image (78).png" alt=""><figcaption></figcaption></figure>

## 인터럽스 우선순위

<figure><img src="../../.gitbook/assets/image (79).png" alt=""><figcaption></figcaption></figure>

## 알아야 할 좋은 내용!

#### 컴퓨터 조립을 생각해보자

* CPU, RAM 을 사서 \_**메모리 매니저**\_가 포함된 Main Board 를 사서 그 위에 조립한다.\
  \-> 저가형 Main Board 를 구매할 시 메모리 매니저의 성능이 떨어진다면 CPU, RAM 의 성능이 떨어진다.
* 메모리 매니저를 브리지 칩셋(노스브리지, 사우스브리지) 라 부른다.
  * 노스브리지 : 빠른 하드웨어 관리
  * 사우스브리지 : 느린 하드웨어(주변기기) 관리

#### 인텔의 혁명

* 인텔에서 메인보드의 종속적인 상황이 마음에 들지 않아 노스브리지 기능을 일부 담당하는 CPU 를 만들기 시작했다.
* 현재 인텔 CPU 는 일부 노스브리지의 기능을가지고 있다고 생각하면 된다.

#### I/O 성능

* Hello, World 를 모니터에 호출할 때, 순서를 간단하게 생각해보자.\
  \-> 모든 I/O 가 일어날 때 인터럽트가 일어난다.
  * API 호출을 통해 구성요소에 Hello, World 전달 (System Call)
  * 구성요소가 API 요청값을 확인 ( Write)
  * 구성요소가 디바이스 드라이버에 값을 전달
  * 디바이스 드라이버가 비디오기기(그래픽카드) 에 요청 전달.
  * 비디오기기가 모니터에 요청 전달.
  * 역순으로 응답 전달
* 고성능 주변기기를 사용하게 되면 비동기로 I/O 가 이루어져 성능이 좋다.
  * CPU 혼자 일을 처리하지 않는다!
* 일반적으로 Hello, World 를 실행하게 되면, API 가 구성요소를 호출하게 되고, 이에 따라 수많은 과정을 거치게 되며 성능이 떨어지게 된다..\
  **-> I/O 가 수없이 일어나는 게임에서 이점을 해결하기 위한 API 가 있는데, 바로 DirectX!**\
  **-> 구성요소를 거치지 않고 Dvice Driver 를 직접 호출하는 API가 DirectX 이다!**

<figure><img src="../../.gitbook/assets/image (80).png" alt=""><figcaption></figcaption></figure>
