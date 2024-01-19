# Symmetric NAT

## Symmetric NAT 동작 순서

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.20.27.png" alt=""><figcaption></figcaption></figure>

* 위 그림과 같이 상황을 가정해보자

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.26.35.png" alt=""><figcaption></figcaption></figure>

* 호스트에서 요청이 나가고, 공유기에 클라이언트의 요청이 나갈 때, IP 와 Port 를 변조해 나가게 된다. (Outbound)
* 이 때, 웹서버 관점에서는 3.3.3.3:23000 이 접근했다고 생각하게 된다. \
  \-> 웹서버 입장에서는 TCP 연결이, 3.3.3.3:23000 과 연결되었다고 생각하게 된다.&#x20;
* **공유기는 Outbound 시점에 NAT-Table 에 해당 네트워킹 정보를 추가한다.** \
  **-> Inbound 에는 추가되지 않는다!!!**\
  **-> NAT table 에는 각각의 호스트마다 다른 포트가 주어진다.**&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.34.18.png" alt=""><figcaption></figcaption></figure>

* **서버는 공유기를 목적지로 SYN + ACK 를 보내게 되는데,**&#x20;
* **이 때, NAT-Table 의 정보(공유기 포트, 목적지 IP, 목적지 포트) 를 기준으로 Serch 해 사설 IP 정보를 얻어낸다.**&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.41.26.png" alt=""><figcaption></figcaption></figure>

* Search 된 정보를 기준으로 호스트에게 응답이 전해지게 된다.&#x20;

## Symmetric NAT 특징

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.44.05.png" alt=""><figcaption></figcaption></figure>

* **2개의 호스트가 같은 도메인에 각각 1번의 요청을 한다고 했을 때, 서버 입장에서는 하나의 출발지에서 2번 요청이 온것으로 이해한다.**&#x20;

#### **단점.. (게임에 한정)**

* **게임에서는 P2P(Pear to Pear) 통신을 하게 되는데, 이때 서버를 거치지 않는다! (속도때문에!)**\
  **-> Symmetric NAT 방식에서는 사용할 수 없는 방식이다. (NAT-Table 레코드가 없기 때문에..)**
