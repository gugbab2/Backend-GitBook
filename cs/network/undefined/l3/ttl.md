# TTL 과 단편화

## TTL(Time To Live)

* Packet 은 인터넷 통신의 단위인데, 목적지까지 도달(수많은 라우터를 거쳐간다)의 실패하는 경우도 있다.\
  \-> 그런 경우, 빠르게 Packet 을 죽여서 네트워크의 부하가 가지 않도록 해야한다.
* 이런 경우 사용되는 것이 TTL 이고, 기본적으로 128 or 255 로 세팅되어 있다.\
  \-> 라우터를 거쳐갈 때(**Hop**) TTL 값이 1이 감소되어 간다.\
  \-> **TTL 이 0 이 되는 순간 목적지를 찾지 못했다고 인식하고, Packet 을 폐기한다.**

## 단편화

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-06 10.45.01.png" alt=""><figcaption></figcaption></figure>

* 단편화는 호스트(스위치, 엔트포인트) 간에 MTU 크기 차이로 발생한다.\
  \-> 보편적으로는 1500byte
* 중간에 MTU 의 크기가 다른 호스트(1400byte) 가 껴들어 간다면, Packet 을 수신하지 못할 수 있기 때문에, 더 작은 크기로 쪼개는데 이 작업을 단편화라고 한다.
* **통상적으로 단편화 된 Packet 의 조립은 수신측 Endpoint(Server) 에서 이루어진다.**\
  **-> 수신측 L3 단에서 조립이 이루어지고 조립된 Packet 에서 Segment 를 끄집어낸다.**

#### **최근 하드웨어에서 대부분의 MTU 는 1500 이하로는 떨어지지 않는데, 언제 떨어질까?**

* 거의 없지만, 네트워크에서 VPN 이 구성되는 경우 발생한다.\
  \-> 이런 경우를 방지하기 위해 다양한 네트워크 기술이 존재한다.