# LAN 과 WAN 의 경계 그리고 Broadcast

## Broadcast (<-> Unicast)

* Broadcast
  * 이슈 : 꼭 필요할때만 사용되어야 하는 상당히 제한적인 네트워킹
  * 때문에, 범위를 제한시키는 것이 상당히 중요하다.
* Broadcast 주소
  * Broadcast 주소라는 매우 특별한 주소가 있다.
  * MAC, IP 모두 존재한다.
  * **MAC 기준으로 이진수 1111 이면 Broadcast 주소이다.**\
    **-> MAC 주소는 48 비트이니, 예를들면 FF-FF-FF-FF-FF-FF 이다.**\
    **-> 목적지 주소가 Broadcast 라면 연결된 모든 End-Point 들이 응답을 받으라는 이야기이다. (그 순간에 전체 네트워크가 통신을 할 수 없다 ..)**\
    **-> 해당 문제 때문에, Broadcast 는 최소화 해야한다! (Broadcast 의 범위를 줄일수도 있다)**
* 그렇다면 목적지 주소는 어디에 담는 것일까?\
  \-> L2 네트워크의 단위인 L2 Frame 을 생각해보자.
  * L2 Frame 에는 맨 앞단에 헤더라는 공간이 있는데,
  * 이곳에 출발지 주소와, 목적지 주소가 담겨져 있다.
  * 우리가 말하는 Broadcast 주소는 헤더 안 목적지에 담기게 된다.

## 네트워크의 규모( LAN 과 WAN 의 경계)

<figure><img src="../../../../../.gitbook/assets/image (14) (1).png" alt=""><figcaption></figcaption></figure>

#### 네트워크 규모를 이해하는 요령(정답은 아니다, 요령이다!)

* LAN 수준 (L2)
  * **Physical 의 수준으로 정의되는 네트워크**
* WAN 수준 (L3 부터!)
  * **Logical(Virtual) 의 수준으로 정의되는 네트워크**
  * 인터넷은 Logical 이다.
