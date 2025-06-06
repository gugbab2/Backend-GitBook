# IPSec VPN 과 터널링 개념

## IPSec(IP Security)

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-16 21.54.49.png" alt=""><figcaption></figcaption></figure>

* **IPSec 은 네트워크에서의 안전한 연결을 설정하기 위한 통신 규칙 또는 프로토콜 세트이다.**
* IPSec 은 네트워크 계층에 보안 서비스를 제공하며, 패킷 단위에 적용된다.&#x20;
* IPSec 은 현재 사용 중인 IPv4, IPv6 를 모두 지원한다.&#x20;
* **IPSec 은 GtoG(Gateway to Gateway) VPN 구현을 위해서 현재 가장 많이 사용되고 있는 방식이다.** \
  **-> IPSec 에서 핵심은 터널링이다.** \
  **-> 터널링은 쉽게 암호화해서 알아볼 수 없게 한다는 것이다.**&#x20;
* IPSec 은 L3(IP 수준)의 보안을 제공한다. \
  -> 따라서 어플리케이션에 대한 의존성이 없고, IP 기반 통신을 모두 보호할 수 있다는 장점이 있다.&#x20;

## IPSec Protocol

* ISAKMP
  * 보안 협상 및 암호화 키들을 관리하는 매커니즘을 제공한다.&#x20;
* IP AH(Authentication Header)
  * AH 는 데이터의 원본 인증 및 **무결성** ~~재연공격 방지 기능을 제공한다.~~ \
    -> 무결성은 Hash 를 사용
* IP ESP(Encapsulation Security Payload)
  * ESP 는 데이터의 기밀성, 원본 인증 및 **기밀성** ~~및 재연공격 방지 기능을 제공한다.~~ \
    -> 기밀성은 PKI 를 사용

## VPN GtoG

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-16 22.06.50.png" alt=""><figcaption></figcaption></figure>

### 다음의 상황을 가정하자

#### 서울 본사의 네트워크 환경

* 폐쇄망에 DB 서버를 구성하고, 내부망에 호스트들만 DB 서버에 접근할 수 있도록 한다.&#x20;
* 내부망에서는 인터넷에 접근할 수 없다.

#### 부산 지사의 호스트에서 VPN 을 통해 서울 본사의 DB 서버를 접속하려면 어떻게 해야 하는가?

* 부산지사와 서울본사 SG(Security Gateway) 간의 터널링 작업을 선행한다. \
  -> VPN GtoG&#x20;
* **VPN 통신을 위한 패킷의 흐름은 다음과 같다.**&#x20;
  * **부산지사 3.3.3.10 에서 서울본사 5.5.5.100 을 접근하기 위한 패킷을 생성하고 요청을 보낸다.**&#x20;
  * **해당 패킷이 3.3.3.1 SG 에 도착했을 때 해당 패킷 전체를 암호화하고 앞에 출발지를 3.3.3.1(부산지사 SG) 목적지를 5.5.5.1(서울본사 SG) 로 설정한 헤더를 추가한 패킷으로 변경한 후 요청을 보낸다.** \
    **-> 기존 패킷의 내용을 암호화 할 수 있지만, SG 출발지/목적지 정보를 담은 헤더까지 암호화 할 수 없다.**&#x20;
  * **5.5.5.1(서울본사 SG) 로 패킷이 도착 후 앞에 헤더를 버리고 SG 암호화 설정에 따라 복호화를 진행한다.**&#x20;
  * **이후 기존 목적지인 5.5.5.100 서버로 접근한다.** \
    **-> 위 상황에서 5.5.5.100 서버 입장에서는 서울본사 네트워크나, 부산지사 네트워크나 다를바가 없다.**&#x20;
