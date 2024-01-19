# DNS

## DNS(Domain Name Server)

* Application 단에서 인프라로 체감할 수 있는 것이 바로 DNS 이다.
* DNS 는 도메인에 해당하는 IP 를 확인해주는 역할을 하는 서버로, 대표적인 DNS IP 는 8.8.8.8(Google) 이 있다.
* **DNS 서버 응답이 느려지면 인터넷 전체가 느려지는 문제가 일어난다.**\
  **-> 결론적으로 DNS 서버에서 IP 를 받아와야 통신이 가능하다.**
* 호스트가 DNS 에 한번이라도 질의를 하면 해당 응답결과를 메모리에 담아두게 되는데, **해당 과정을 DNS Cache** 라고 부른다.
* 엔드포인트 안에는 DNS 의 역할을 하는 hosts 파일이 존재한다.

## DNS 동작 과정

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-06 16.34.14.png" alt=""><figcaption></figcaption></figure>

#### 브라우저에서 www.naver.com 을 입력하는 상황을 생각해보자

* 운영체제 내부에서 IP 설정에서 등록된 DNS 서버에 도메인에 해당하는 IP 주소를 달라고 질의를 보낸다.
* DNS 서버는 응답으로 IP 주소를 보내준다.\
  **-> 응답을 줄 때 항상 같이 주는 데이터가 유효기간이다. (유효기간이 넘어가면 다시 질의해야 한다)**
* IP 주소를 기반으로 네이버 서버에 HTTPS 통신을 보내 접속하게 된다.

#### DNS 가 해킹을 당해 다른 주소를 전달해준다면 어떻게 될까?

* 실제 해킹이 되어 엉뚱한 IP 를 알려준다면, 엄청난 일이 일어날 수 있기 때문에, 아주 강력한 보안이 적용되어 있다.

## Domain Name

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-06 16.29.59.png" alt=""><figcaption></figcaption></figure>

* www 는 naver.com 에 소속되어 있고, www.naver 는 com 에 소속되어 있다.
* www 를 Host Name 이라 부르고,
* naver.com 을 Domain Name 이라 부른다.
* **따라서 www.naver.com 은 naver.com 도메인에 소속 된 www 호스트를 찾으라는 의미로 해석된다.**
