# Broadcast IP 주소와 Localhost

## Broadcast IP

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-04 22.28.43.png" alt=""><figcaption></figcaption></figure>

* IP(Network ID + Host ID) 에서Host ID 가 이진수로 생각할 때, 모두 1로 채워진다면 Broadcast Ip 이다. \
  \-> 다음IP(192.168.0.1/24) 기준으로 생각해보자\
  (1100  0000 1010 1000 0000 0000 0000 0001 -> 1100  0000 1010 1000 0000 0000 1111 1111)
* 연결된 모든 호스트에 송신이 된다.
* Broadcast 시 잠깐 네트워크가 멈추기 때문에, 최소화해야한다. \
  \-> 연결된 모든 호스트에 송신해야 하기 때문에, 네트워크 장비의 부화가 늘어난다!

## Localhost

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-04 22.36.19.png" alt=""><figcaption></figcaption></figure>

* 자기 자신에게 접속(연결) 해야 할 때, 주체는 Process 이다. \
  \-> 호스트는 주체가 아니다!
