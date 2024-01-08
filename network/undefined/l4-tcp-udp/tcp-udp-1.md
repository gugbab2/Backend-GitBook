# TCP, UDP 헤더 형식과 게임서버 특징

## TCP 헤더 형식&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-06 15.32.40.png" alt=""><figcaption></figcaption></figure>

* Sequence number 와 ACK number 의 의미가 다른것을 인지하자.\
  \-> 두 숫자가 같을 수는 있다.&#x20;
  * Sequence number : 연결지향에서 패킷의 순서를 파악하기 위함
  * ACK number : SYN 에 대한 응답을 표현하기 위함

## UDP 헤더 형식&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-06 15.45.39.png" alt=""><figcaption></figcaption></figure>

* 패킷의 순서, 혼잡을 제어하지 않기 때문에, 상당히 단순하다.
* 실시간 서비스(유튜브, 게임) 에 경우 사용자의 네트워킹의 상태에 따라 서비스 속도가 달라질 수 있기에, 최대한 단순화 된 UDP 형식이 필요하다.&#x20;
