---
description: REST API 제작시 꼭 알고 있어야 할 HMAC 기반 인증
---

# HMAC 기반 인증

## HMAC 개요

* 유명한 서비스의 REST API 를 사용하려면 secret access key id(보안 액세스 키 ID) 와 secret access key(보안 엑세스 키) 를 생성해서 등록해야 사용이 가능한 경우가 많습니다.
* 예로 AWS 는 발급받은 "AWS 보안 액세스 키 ID" 를 Authorization 헤더에 추가해서 보내야 하며 본문은 "보안 엑세스 키" 를 사용해서 HMAC 방식의 인증을 거쳐야 API 사용이 가능합니다.

```
Authorization: AWS AWSAccessKeyId:Signature
```

* 위와 같은 방식은 내부적으로 해시 기반 메시지 인증(HMAC; Hash-based Message Authentication Code))이라는 보안 기술을 기반하고 있다.

## HMAC 개념

* HMAC 은 암호화 해시 함수(MD5, SHA1, SHA256)를 사용하여, 클라이언트가 보낸 메시지를 인증하는 방식으로 가볍고 구현이 용이하며, 속도가 빨라 다양하게 활용되고 있다.\
  -> 특히 REST API 필수 구성 요소로 자리 잡고있다.\
  -> 해시 함수 : 입력에 대해서 유일한 출력을 내는 함수
* **클라이언트**는사용자의 ID 와 같은 민감 정보를 직접 받을 필요가 없이, 사전에 공유한 secret key 와 전송할 message 를 입력 받아서 Hash 기반 MAC 을 생성해서 전송하며
* **서버**는 secret key 와 message 를 기반으로 MAC 를 검증해서, secret key 를 소유한 클라이언트가 보낸 메시지가 맞는지 인증할 수 있다.

## HMAC 동작 원리

<figure><img src="../.gitbook/assets/image (48).png" alt=""><figcaption><p>HMAC 동작 원리</p></figcaption></figure>

1. 해시 생성 : 클라이언트는 key + message 를 HMAC 알고리즘으로 처리해 해시 값을 만들어낸다.
2. 요청 보내기 : 생성된 해시와 message 를 HTTP 요청으로 REST API 서버에 보냅니다.\
   -> **보통의 해시는 HTTP Header 또는 URL 에 포함된다.**\
   &#xNAN;**-> message 또한 민감 정보가 들어있을 경우 별도의 암호화를 사용한 후 보내야만 한다.**
3. 해시 생성 : 서버는 클라이언트에게서 받은 요청 내의 message 와 본인이 가지고 있던 key를 조합하여 HMAC 해쉬 값을 생성합니다.
4. 비교 : 클라이언트에게서 넘어온 해시와 서버에서 생성된 해시가 동일한지 비교한다. 동일하면 인증 성공이다!

## HMAC 을 더 안전하게 사용하는 방법

* **전송시 안전한 채널 사용(HTTPS)**
  * HMAC 값은 secret key 가 없다면 message 의 위변조가 불가능하지만, 원문 message 를 같이 보내야 하므로 보안을 위해 HTTPS 와 같이 안전한 전송 채널을 이용하는 것을 권장한다.\
    -> 최근 HTTPS 사용은 필수적이다..
* **Secret Key 관리**
  * 클라이언트가 전송한 요청은 중간에 해커가 가로챌 수 있지만, secret key 가 없다면 위변조해도 서버의 검증 과정에서 에러가 나게 된다.
  * 반대로 secret key 가 유출된다면 해커가 임의로 위변조할 수 있으므로 secret key 를 안전하게 관리하고 유출 우려가 있을 경우에는 재발급해서 사용해야 한다.
* Replay attack 방지
  * 클라이언트가 전송한 요청은 중간에 해커가 가로채서 replay attack 에 활용할 수 있다.
  * 때문에, message 에 timestamp, serial 등 시퀀스 값을 포함하는 것이 필요하다.
