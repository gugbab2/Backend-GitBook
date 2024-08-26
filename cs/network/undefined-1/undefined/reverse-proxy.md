# Reverse Proxy(서버 입장)

## Reverse Proxy

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-13 15.45.20.png" alt=""><figcaption></figcaption></figure>

* 기본적으로 서버를 보호하기 위한 목적의 Proxy 이다. \
  \-> **기존 클라이언트를 위한 Proxy 가 아니기 때문에, Reverse Proxy 라고 부른다.**&#x20;
* 보통의 서버 입장에서 네트워크 구성은 다음과 같다. \
  \-> IPS(방화벽) > 서버&#x20;
* **하지만, HTTPS 프로토콜을 사용하면서, 위의 구성은 암호화된 HTTPS 요청을 판별할 수 없다.**&#x20;
* **때문에, 다음과 같은 구성이 생겨났다.** \
  **-> IPS(방화벽) > SSL accelerator(SSL 암호문 해석용) > WAF(해석 된 평문 데이터 판별용) > 서버** \
  **-> 여기서 SSL accelerator, WAF 용도의 Proxy 서버를 Reverse Proxy 라고 부른다.**\
  **-> 해당 구조에서는 서버 도메인의 IP 는  SSL accelerator IP 이다.**&#x20;
* 최근에는 웹서버의 설정을 통해 Reverse Proxy 용도로 사용한다.&#x20;
