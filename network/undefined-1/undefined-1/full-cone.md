# Full Cone 방식과 내부 네트워크 접속 문제

## Full Cone

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-13 16.57.23.png" alt=""><figcaption></figcaption></figure>

* Symmetric NAT 과 동일하게 Outbound 에서 NAT-Table 에 레코드를 쌓게 되는데, 이때 목적지 IP, Port 를 Any 로 열어 해당 호스트에 대해서는 모든 요청이 허용되도록 하는 방식이다.&#x20;
* **보안성을 떨어질지언정, 속도 측면에서 높은 성능을 보일 수 있다.** \
  **-> 게임, 스트리밍 서비스에서 많이 사용되는 것을 볼 수 있다.**





