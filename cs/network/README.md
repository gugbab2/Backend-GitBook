# NETWORK

## 네트워크와 네트워킹 그리고 개념

<figure><img src="../../.gitbook/assets/image (19) (1).png" alt=""><figcaption></figcaption></figure>

#### 네트워크

* 관계!
* 네트워킹(상호작용)이 이루어지기 위해서는 의존적(Layered) 관계가 이루어져야 한다.\
  \-> OSI 7Layer

#### 네트워킹

* 상호작용!

#### OSI 7Layer

* 명백히 개념에 관한 이야기이다..\
  \-> 실제 우리 개발환경에서 사용하는 것은 개념이 아닌, 구현이다!\
  \-> 때문에, 네트워크 공부 시 OSI 7Layer 를 보게된다면,, 너무 어렵게 접근하게 된다..
* 개념과 구현을 구분해야만 한다!\
  \-> 연예인과 아이유는 다는 요소이다.\
  \-> 아이유와 친해지고 싶다면, 연예인을 공부하는 것이 아닌, 아이유 팬클럽에 가입해야 한다.

## User mode 와 Kernel mode

<figure><img src="../../.gitbook/assets/image (23) (1).png" alt=""><figcaption></figcaption></figure>

### H/W

* Interupt
  * CPU 의 행위를 방해하는 요소
  * 컴퓨터는 보통 주변기기와 함께 사용되며, 서로 I/O 를 하며 동작되는데, I/O 할 때마다 Interupt 가 발생한다.\
    \-> I/O 가 많을 수록 CPU 의 성능이 줄어들 수 있다.

### S/W

Kernel mode(신계)

* Driver
  * 하드웨어를 제어하기 위한 소프트웨어\
    \-> 드라이버가 설치되어 있지 않다면, 하드웨어를 제어할 수 없다.
* 구성요소
  * TCP/IP 는 수많은 소프트웨어 구성요소(프로토콜) 중 하나
  * 커널을 구동하기 위한 소프트웨어 프로토콜

#### User mode(인간계)

\-> L5 이상!

* File
  * 커널의 구성요소를 실행하기 위한 인터페이스 형태
* Socket
  * 특별히! TCP/IP 구성요소를 실행시키기 위한 인터페이스 형태
* Process
  * Program 을 실행한 형태
  * 우리가 실행중인 크롬도 프로세스이다.
  * **구성요소(소켓!) 을 여는 주체는 프로세스이다!**
