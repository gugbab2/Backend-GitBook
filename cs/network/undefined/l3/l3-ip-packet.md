# L3 IP Packet 으로 외워라

## L3 Packet

<figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 패킷은 개념적으로 단위 데이터라고 생각할 수 있다.
* Packet = L3 IP Packet
* 패킷은 Header 와 Payload 로 구분되며( L2 Frame 도 마찬가지이다)
* **최대 크기는 MTU( Maximum Transform Unit) 이다. (매우중요!)**
  * **특별한 이유가 없다면 1500byte 이다.(매우 작은 양)**\
    **-> 최근에 해당 크기로 보낼 수 있는 데이터가 없다..**

<figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 와이어샤크 툴을 사용해 패킷을 확인이 가능하다.

## Encapsulation 과 Decapsulation

<figure><img src="../../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* Encapsulation : 단위화, 보안성 강화\
  \-> 러시아 전통인형 마트료시카를 생각하자\
  **-> 네트워크에서L2 의 payload 는 L3 패킷, L3의 payload 는 L4 의 세그먼트 .. 와 같이 Encapsulation!**
* Decapsulation : Encapsulation 의 반대 개념
