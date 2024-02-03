# Restricted Cone, Port Restricted Cone

## (IP)Restricted Cone

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-14 08.24.53.png" alt=""><figcaption></figcaption></figure>

* Restricted Cone 방식은 NAT-table 레코드 등록 시 Remote IP 를 등록해,
* Inbound 시 특정 IP 의 요청만 들어올 수 있도록 한다.&#x20;

## Port Restricted Cone

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-14 08.35.29.png" alt=""><figcaption></figcaption></figure>

* Restricted Cone 방식은 NAT-table 레코드 등록 시 Remote IP, Remote Port 를 등록해,
* Inbound 시 특정 IP, Port 의 요청만 들어올 수 있도록 한다.&#x20;
* Symmetric NAT 방식과 비슷하지만, **각각의 호스트마다 다른 Port 를 부여하지 않는다는 점에서 차이를 보인다.** \
  **-> External Port 자원을 아끼는 방식이다. (방식의 철학이 다를 뿐 프로세스는 비슷하다)**
