# ARP 과 Ping(RTT : Round Trip Time)

## ARP(Address Resolution Protocol)

-> ARP 에서 Address 는 IP, MAC 주소를 가리킨다.

* ARP 는 IP 주소로 MAC 주소를 알아내려 할 때 활용된다.

#### 그렇다면, IP 주소로 MAC 주소를 알아내야 할 때는 언제일까?

* 보통의 경우 Gateway 의 MAC 주소를 알아내야 할때이다.\
  -> Gateway 의 MAC 주소를 알지 못한다면 인터넷을 할 수 없다!
* 엔드포인트가 네이버의 접근한다고 생각해보자.
  * 네이버와의 통신의 인터넷 구간 바로 L3 구간에서 이루어지기 때문에, MAC 주소는 알 필요가 없다.\
    -> 오직 IP 주소만 알면 된다.
  * 하지만, 자신의 네트워크 Gateway 의 MAC 주소는 알아야만 한다.

#### ARP 동작과정

* 매번 ARP 를 하지는 않고, 한번 ARP 가 이루어졌을 때, Cache 를 해둔다.

1. 엔드포인트는 네이버와 통신하고자 할 때, ARP Request 를 **Broadcast** 를 보내게 된다.\
   -> **우리 네트워크 안에 Gateway 가 있니?**\
   -> DHCP 를 통해, Gateway 주소를 알고 있다.
2. Gateway 는 엔드포인트에게 응답(Reply)을 준다.\
   -> 응답에는 MAC 주소가 포함되어 있다.
3. **엔드포인트는 L2 Frame 을 만들 때, 목적지 주소를 Gateway MAC 주소로 설정해준다.**\
   &#xNAN;**-> 왜냐면, 모든 네트워크 요청을 GW 를 거쳐가야 하기 때문이다!**

## Ping

* Ping 은 특정 Host 에 대한 RTT(Round Trip Time) 을 측정할 목적으로 사용되는 프로그램이다.\
  -> 보통의 멀티 게임에서 플레이어들이 서버와 동기화 상태를 위해서 확인한다.
* ICMP 프로토콜을 사용한다.
* DoS(Denial of Service) 공격용으로 악용되기도 한다.\
  -> 멀티 게임에서 RTT 가 빠른 호스트에서 CPU 사용율을 높여서 무지막지한 Ping 을 보내게 되면, 서버 과부화가 일어나게 된다.
* **RTT** 는 거리에 비례하는 것이 아닌, **회선 속도이다!**
