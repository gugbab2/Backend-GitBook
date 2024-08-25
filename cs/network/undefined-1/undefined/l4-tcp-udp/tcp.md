# TCP 연결 및 상태 변화

## TCP 연결 과정(3-way handshaking)

\-> 논리적인 과정이다! (물리적인 과정이 아니니 헷갈리지 말자!)

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-06 11.49.15.png" alt=""><figcaption></figcaption></figure>

* 3-way handshaking 에서 통신 단위는 Segment 인데, 일반적인 Segment 가 아닌 IP, TCP 헤더만(Payload 가 없다) 담겨있는 관리 목적의 Segment 이다.
* 3-way handshaking 에는 시간차가 존재하는데, 클라이언트가 ACK 를 받은 시점과, 서버가 ACK 를 받은 시점이 다르다는 것이다.\
  \-> Established 시점이 다르다.\
  **-> 회사 전문통신을 생각해볼 때 netstat 명령어로 서버와 클라이언트 네트워크 상태를 살펴볼 때, 서버와 클라이언트 모두 Established 상태여야만 한다.**&#x20;

#### 연결 과정을 살펴보자.

1. 클라이언트가 서버에 요청을 하기 전 Sequence Number(랜덤값) 를 생성하고, 서버에 요청(SYN)을 보낸다.
2. 서버는 클라이언트의 요청을 받고, 다음의 과정을 수행한다.
   1. Sequence Number 를 생성하고(랜덤값),
   2. 클라이언트에 요청에 대한 응답(ACK) 을 전달한다.\
      \-> 클라이언트 Sequence Number + 1
   3. 클라이언트에게 요청(SYN) 을 보낸다.
3. 클라이언트는 서버에 요청을 받고 응답(ACK) 을 전달한다.\
   \-> 서버의 Sequence Number + 1

#### 상태변화

* 클라이언트
  * SYN\_SEND -> ESTABLISHED
* 서버
  * LISTEN -> SYS\_RCVD -> ESTABLISHED

#### 3-way handshaking 의 결과

* 클라이언트, 서버의 Sequence Number 교환
* 정책 교환
  * 클라이언트, 서버의 MSS(Maximum Segment Size) 교환\
    \-> MSS 가 다르다면 큰쪽이 낮은쪽에 맞춘다.
