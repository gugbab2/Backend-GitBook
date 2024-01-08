# 이해하면 인생이 바뀌는 TCP/IP 송, 수신 구조

## TCP/IP 통신의 전체적인 흐름&#x20;

#### 1. 네이버에서 파일을 다운로드 받는 상황을 생각해보자!

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-04 21.27.38.png" alt=""><figcaption></figcaption></figure>

* 먼저, 파일을 주고 받는 클라이언트, 서버는 프로세스(호스트)의 형태이다.
* 두번째, 인터넷 상에서 정보가 유통될 때 Packet 의 형태로 공유하게 된다.\
  \-> 여기서 중요한 점은 파일의 크기가 1.5MB 라고 할 때, MTU 는 1500byte 이기 때문에, 약 1000배 이상의 차이가 난다.&#x20;

#### 2. 파일을 다운로드할 때 서버는 어떻게 송신할까?

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-04 21.44.30.png" alt=""><figcaption></figcaption></figure>

* Socket(본질적으로 File) 에서 입출력이 일어나면, Socket 에 확보된 메모리 공간인 Buffer 가 있기 마련이다.\
  \-> 물론, 서버 프로세스 내에도 Buffer 가 있다. (메모리에 올릴 Buffer 의 크기는 개발자가 설정한다)
* 송신하는 쪽에서는 항상 **Encapsulation** 이 일어난다.&#x20;

1. 파일을 일정한 크기로 나누어 서버 프로세스 내 Buffer 로 Copy 한다.&#x20;
2. 프로세스의 Buffer 에서 Socket 의 Buffer 로 Copy 한다. \
   \-> 단위 : Stream&#x20;
3. Socket 의 Buffer 를 분해해서 L4(TCP) 계층으로 전송한다. \
   \-> 단위 : Segment(TCP 기준, UDP 의 경우는 Datagram 이라고 표현한다)\
   \-> Segment 는 각각의 순서를 가지고 있다. (TCP : 연결지향인 이유)\
   **-> 순차적으로 Segment 를 송신한다.** \
   **-> 몇개의 Segment 를 보낸 뒤 ACK 를 기다리기 위해서 Wait 한다.**
4. L3(IP) 계층에서 Packet 으로 Encapsulation 한다. \
   \-> 단위 : Packet
5. L2(NIC) 계층에서 4 와 마찬가지로 Encapsulation 한다. \
   \-> 단위 : Frame
6. 수 많은 라우터를 거쳐 클라이언트에게 전달된다. \
   \-> 보통 한개의 라우터가 아니다.. 수많은 라우터를 거쳐 전달된다

#### &#x20;3. 파일을 다운로드할 때 클라이언트는 어떻게 수신할까?

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-04 22.03.30.png" alt=""><figcaption></figcaption></figure>

* 수신하는 측에서는 **Decapsulation** 이 일어난다.
* **동시에 Socket 의 Buffer 를 비우고 채우기 때문에, 속도차가 발생하게 된다.**&#x20;

1. L3 계층에서 Frame 안에서 Packet 을 끄집어 내고 Frame 은 사라진다.&#x20;
2. L4 계층에서 Segment 를 끄집어내고 Packet 은 사라진다.&#x20;
3. Segment 들은 순차적으로 Socket 의 Buffer 에 쌓이게 된다.\
   \-> 해당 동작은 운영체제의 TCP 스택에서 담당한다.  \
   **-> 네트워크 단에서는 Socket 의 Buffer 에 **_**채우는**_** 동작이 일어나게 된다.** \
   **-> TCP 는 연결지향으로 Segment 를 잘 전달받았을 때, ACK 를 서버에 전달한다.** \
   **(ACK 를 보낼 때 현재 Socket Buffer 여유공간을 함께 전달한다)**
4. 프로세스는 Socket 의 Buffer 에서 프로세스의 Buffer 로 Copy 한다.  \
   **-> 프로세스 단에서는 Socket 의 Buffer 에 **_**비우는**_** 동작이 일어나게 된다.**
5. 프로세스에서 File 을 확인할 수 있다.&#x20;

#### 4. 대표적인 TCP 장애는 무엇일까?

* 네트워크 수준의 장애
  * Loss Segment : 데이터 유실 (원인은 모른다,,)
  * Retransmision : 수신측의 ACK 와 송신측의 Retransmision 이 겹쳐 ACK 가 중복된다. \
    \-> 일정 수준을 운영체제 TCP 수준에서 보완을 해준다.&#x20;
  * Out of order : Segment 의 순서가 맞지 않을 때\
    \-> 일정 수준을 운영체제 TCP 수준에서 보완을 해준다.&#x20;
* 프로세스 수준의 장애
  * Zero Window : **Socket 의 Buffer(Window Size) 의 남은 공간이 Zero 가 될 때**\
    **->  **_**네트워크와 프로세스의 속도차이 때문에 발생**_
