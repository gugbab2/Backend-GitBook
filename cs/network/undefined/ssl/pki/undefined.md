# 인터넷을 위한 비대칭키 체계

## 대칭키 인터넷 환경 문제&#x20;

* 대칭키 체계에서 키는 중요한 개인정보인데, 이 정보를 어떻게 안전하게 보낼 것인가?
* **하지만, 대칭키를 인터넷 환경에서 일반적인 송수신으로는 안전하게 전달할 수 없다..**
  * 통신의 효율을 위해서 대칭키를 사용해 데이터를 암복호화 해야 한다..
* **이를 해결하기 위한 방식이 비대칭키 방식이다.** &#x20;

## 인터넷을 위한 비대칭키 체계

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-19 14.25.57.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트와 서버 모두 각각의 비대칭키(공개키, 비공개키) 를 만든다.&#x20;
2. 클라이언트 서버 모두 각각의 공개키를 공유한다.&#x20;
3. 공유된 공개키를 기반으로 암호화를 진행하고,&#x20;
4. 암호화된 데이터를 Tunnel 을 통해 송수신한다.&#x20;
   1. 해당 방식으로 암호화 된 TCP/IP 세션을 Tunnel 이라고 한다.&#x20;
5. 클라이언트 서버 모두 각각의 비공개키로 복호화를 진행한다.&#x20;

## 효율 극대화를 위한 혼합(대칭키, 비대칭키) 활용

* **하지만 비대칭키 방식은 컴퓨팅 파워를 상당히 소모하기에 효율적인 방식이라고 볼 수는 없다.**&#x20;
* **때문에, 대칭키와 비대칭키를 혼합한 방식을 사용하게 되며, 해당 방식은 SSL 인증의 기반이다.**&#x20;
  * 여기서 공유되는 대칭키를 Session Key 라고 부른다!

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-19 14.40.00.png" alt=""><figcaption></figcaption></figure>

1. 서버는 비대칭키(공개키, 비공개키) 를 만든다.&#x20;
2. 서버의 공개키를 기반으로 클라이언트는 대칭키를 암호화한다.&#x20;
3. 암호화된 대칭키를 Tunnel 을 통해 송수신한다.&#x20;
4. 서버는 공개키로 암호화 된 대칭키를 비공개키를 사용해 복호화한다.&#x20;
5. 클라이언트 서버 모두 공유된 대칭키를 기반으로 암복호화해 통신한다.

## 비대칭키 체계의 문제점&#x20;

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-19 14.47.26.png" alt=""><figcaption></figcaption></figure>

* 만약 서버의 공개키를 해커가 가로채 해커의 공개키로 클라이언트에게 전달해준다면,
* 클라이언트는 해커의 공개키로 대칭키를 암호화해 서버에게 요청을 보내게 된다.&#x20;
* 요청을 보내는 중간에 해커는 암호화 된 대칭키를 확인하고 서버의 공개키로 다시 암호화 후 서버에 전달해준다.&#x20;
* 위의 상황이 완성된다면, **해커는 중간에서 클라이언트와 서버의 모든 송수신 정보를 도청할 수 있다!**
