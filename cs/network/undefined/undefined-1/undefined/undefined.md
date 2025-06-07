# 세 가지 네트워크 장치 구조

## Inline

* **Packet + Drop/Bypass + Filtering**
  * Packet(MTU) 을 필터링 하는 구조이다.&#x20;
* 장치 :  공유기, 라우터, 패킷 필터링 방화벽, IPS 등..

## Out of path

* **Packet + Read only, Scaner**
  * Packet 을 모니터링 하는 구조이다.&#x20;

## Proxy

* **Socket Stream + Filter**&#x20;
  * Stream 을 필터링 하는 구조이다.&#x20;
  * 필터링 한다는 점에서 Inline 구조와 비슷
