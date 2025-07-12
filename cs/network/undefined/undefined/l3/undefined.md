# 패킷의 생성과 전달 및 계층별 데이터 단위

## 패킷의 생성과 전달

<figure><img src="../../../../../.gitbook/assets/image (7) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

1. 프로세스에서 데이터를 인터페이스(File)에 Write 해준다.
   1. TCP/IP 프로토콜에서는 인터페이스가 Socket 이다.
   2. TCP/IP 프로토콜에서는 Write 가 Send 이다.
   3. **여기서 Socket 은 User mode 프로세스에서 Kernel mode 프로토콜을 접근할 수 있는 추상화한 인터페이스이지, 데이터 단위가 아니다!**&#x20;
2. TCP 로 전달되게 되면, Segment 형태(TCP Header + Data) 화 한다.
3. 아래 계층으로 가면 갈 수록 앞에 Header 를 붙이는 형태로 가공이 되고, 결국 적으로 L2 Frame 의 형태로 가공이 된다. (Encapsulation)
4. 결국, L2 Frame 의 형태로 이더넷을 타고 L2 Access Switch -> Router -> Internet 으로 통신되는 구조이다.

## 계층별 데이터 단위

<figure><img src="../../../../../.gitbook/assets/image (8) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* L1 \~ L2 : Ethernet Frame
* L3 : Packet
  * MTU : Maximum Target Unit(1500 byte)
* L4 : Segment
  * MSS : Maximun Sagment Size(1460 byte)
* L5 이상(Socket) : Stream
  * 시작은 있지만, 끝은 정의할 수 없다.
  * 운영체제 입장에서는 끝을 알 수 없는 큰 데이터 덩어리이다.
* **주의할 점!**
  * **Stream 이 MSS 의 크기로 넘어갈 때, MSS 단위로 분할한다.**
  * **이 후 MTU 단위에 맞추어 인터넷에서 통신한다.**
