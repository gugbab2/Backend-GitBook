---
description: 화
---

# TCP 연결 종료 및 상태 변화

## TCP 연결 종료 과정(4-way handshaking)

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-06 15.10.19.png" alt=""><figcaption></figcaption></figure>

* 특별한 이유가 없다면, 클라이언트가 Active 하다.\
  \-> 클라이언트가 연결을 끊겠다는 요청을 한다!
* Socket 은 CLOSED 상태가 되어야 회수 할 수 있다.\
  \-> 호스트 입장에서 Socket 은 자원이다! (잘 관리되어야 한다)
* **서버 입장에서 연결을 끊게 된다면 TIME\_WAIT 시간을 기다려야 하는 낭비(?) 가 발생할 수 있기 때문에, 서버측에서 연결을 끊는 것이 아닌, 클라이언트 측에서 연결을 끊도록 해야한다.**\
  **-> 때문에, Socket 단 프로토콜(L4 이상) 에서는 서버에서 요청하면, 클라이언트가 연결을 끊는 요청을 하도록 유도 설계가 되어있다.**

#### 연결 종료 과정을 살펴보자

1. 클라이언트가 서버에 연결을 끊자는 의미의 요청(FIN + ACK)을 보낸다.
2. 서버는 클라이언트에게 해당 요청에 대한 응답(ACK)을 보낸다.
3. 서버는 CLOSE\_WAIT 시간을 거친 후 클라이언트에 연결을 끊자는 의미의 요청(FIN + ACK) 을 보낸다.
4. 클라이언트는 해당 요청에 대한 응답(ACK) 을 보낸다.

#### 상태 변화

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-06 15.28.31.png" alt=""><figcaption></figcaption></figure>

* 클라이언트
  * ESTABLISHED -> FIN\_WAIT1 -> FIN\_WAIT2 -> TIME\_WAIT -> CLOSED\
    **-> TIME\_WAIT : TIME\_WAIT 가 발생하는 쪽에서 연결을 끊자는 요청을 한 것이다!**\
    **(서버에서 TIME\_WAIT 가 발생한다면, 일반적인 서버 상황은 아니라고 볼 수 있다)**
* 서버
  * ESTABLISHED -> CLOSE\_WAIT -> LAST\_ACK -> CLOSED
