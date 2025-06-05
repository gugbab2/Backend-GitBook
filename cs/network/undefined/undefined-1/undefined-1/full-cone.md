# Full Cone 방식

## Full Cone

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-13 16.57.23.png" alt=""><figcaption></figcaption></figure>

* Symmetric NAT 과 동일하게 Outbound 에서 NAT-Table 에 레코드를 쌓게 되는데, 이때 목적지 IP, Port 를 Any 로 열어 해당 호스트에 대해서는 모든 요청이 허용되도록 하는 방식이다.&#x20;
* **보안성을 떨어질지언정, 속도 측면에서 높은 성능을 보일 수 있다.** \
  **-> 게임쪽에서는 속도를 향상시키기 위해서 P2P 방식을 사용하는데, (웹서버를 거친다는 것이 속도에 영향을 줌)**\
  **-> 게임 클라이언트가 웹서버에 요청을 할 경우, 웹서버는 P2P 대상 서버에게 클라이언트의 정보를 알려줘 P2P 통신을 할 수 있도록 정보를 제공한다.**&#x20;
* Full Cone 방식의 경우 외부 모든 요청을 통과 시킨다는 보안적 단점이 존재한다. \
  -> 이를 해결하기 위해 나타난 방법이 Restricted Cone, Port Restricted Cone
