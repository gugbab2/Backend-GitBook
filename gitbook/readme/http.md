---
description: HTTP 개요, HTTP 메시지
---

# HTTP 이해

## HTTP(Hypertext Transfer Protocol)

* WEB 상에서 발생하는 통신을 위한 가장  기본적인 프로토콜(규칙,  규약)
* 클라이언트-서버 프로토콜이기도 하다
* HTTP 는 OSI(Open Systems Interconnection) 7계층에서 마지막 7계층에 해당되는 프로토콜로  웹개발자의 입장에서는 2, 3, 4, 7 계층 정보를 이해하고 넘어가자
  * 2계층(데이터 링크 계층) \
    \- MAC Address 라는 물리 주소(유일)를 가지게 되고, 2계층부터 주소체계를 가지게 된다.
  * 3계층(네트워크 계층) \
    \- IP 라는 논리 주소(유일하지  않음)를 가지게 되고, 이는 네트워크 관리자에 의해 관리된다.
  * 4계층(전송  계층) \
    \-  Port 를 가지게 되고, 각 IP 내부 특정 프로세스를 식별한다.\
    \-  일반적으로 TCP, UDP 프로토콜을 사용하며, TCP는 연결지향(올바른 패킷관리) 프로토콜이다.
  * 7계층\
    \- 1\~4계층까지는 운영체제 와 같이 사람이 아닌 곳에서 할당하고 관리하지만, \
    \- 7계층은 사람이 직접 개발하고 관리  해야한다.

## HTTP와 HTTPS의 차이(TLS)

* 위에서 말했듯이, HTTP 는 웹표준 프로토콜이고, HTTPS 는 중간에 TLS(SSL) 를 함께 사용하는 보안이 강화된 프로토콜이다.
* 이를 통해서 웹 상에서, 통신 내부 정보가 평문으로 노출되지 않는다.\
  (와이어샤크와  같은 패킷분석툴을 통해서  패킷정보를 확인 할 수 있다)![](../../.gitbook/assets/image.png)

## 클라이언트-서버 모델

* HTTP 요청 대상을 리소스/서비스 라고 부르며 URL( URI) 을 통해서 요청을 보낸다.
* 클라이언트 -> 요청
* 서버 -> (처리, Optional) -> 응답

## stateless와 stateful

* HTTP 는 각각의 요청이 독립적인 Stateless Protocol 이다.&#x20;
* 때문에, 클라이언트는 항상 자신이 누구인지 서버에 요청을 보낼 때마다 알려주어야 한다.

## HTTP Cookie와 HTTP Session

* Cookie
  * 위와 같이 Stateless 상황에서 자신을 서버에 알리기 위한 정보를 쿠키에 담아서 서버에 전달한다.
  * 요청과 응답을 통해서 계속적으로 쿠키를 주고받게 된다.
* Session
  * 쿠키는 브라우저 개발자 도구 등 쉽게 변조가 가능하여 보안이 취약하다는 단점을 가지고 있다.
  * 이에 대한 해결책으로 나온 세션은, 데이터는 서버에서 관리하고 이를 사용할 수 있는 Key 값을 쿠키에 담아 요청과 응답을 통해 쿠키를 주고받는다.
  * 보통 Key 값으로는 예측할 수 없는 UUID 등을 많이 사용한다.

## HTTP 메시지 구조

* ### HTTP 요청(Reuqest)와 응답(Response
  * #### 요청과 응답의 구조
    * Start line\
      \- 실행되어야 할 요청 OR 요청  수행에 대한 결과 값
    * HTTP headers\
      \- 요청과 응답에 대한 설명 \
      \- Body 에 대한 설명
    * empty line
    * Body\
      \- 크기를 알기가 어렵다. 때문에 헤더 값 중 하나인 Content-Length 를 확인해야 한다.\
      \- 꼭 사람이 읽을 수 있는 텍스트일 필요는 없다. 바이너리 형태도 가능하다.\
      \- 하나가 아니라 여럿일 수 있다. 파일 업로드를 위한 multipart/form-data 가 대표적이다.
  * #### multipart/form-data
    * html \<form> 을 통해 데이터를 서버로 전송할 때 설정하지 않을  시 default 로 Content-Type: application/x-www-form-urlencoded 와 같은 Content-Type 을 사용하고, 전송되는 각각의  데이터들은 HTTP Body 에 '**&**' 로 구분 지어 전송되게 된다. \
      (문자열과 숫자의 조합일 때는 상관없다)
    * 하지만, 파일을 서버로 전송할 때는 바이너리 데이터를 전송해야 하고 수 많은 문자의 조합이 사용되어 '**&**' 로 구분하지 못한다.
    * 이때, multipart/form-data 를 사용하게 되면 임의로 생성되는 바운더리를 통해서 구분되며 임의의 바운더리는 UUID 로 매번 임의로 생성된다.
* ### HTTP 요청 메서드(HTTP request methods)
  * #### 멱등성 : 매번 같은 요청을 보내도 같은 결과를 나타내는 것을 뜻함
  * GET → Read
  * HEAD → GET without body
  * POST → Submit (멱등성X) \
    \- GET 은 URL 을 통해서 요청을 보내 데이터(개인정보)를 평문으로 노출시킬 수도 있지만,\
    \- POST 는 HTTP Body 를 통해서 데이터를 보내기 때문에 한번 감추어준다.(완벽한 건X)
  * PUT → Update (+Create) ⇒ Overwrite!
  * PATCH → Update (멱등성X) => Particial!
  * DELETE → Delete
  * OPTIONS → 리소스와의 통신 옵션을 확인하기 위해 사용
* ### HTTP 응답 상태 코드(HTTP response status code)
  * 1xx → 정보 ⇒ 우리가 직접 쓰는 일은 드믐.
  * 2xx → 성공 ⇒ 200 OK, 201 Created, 204 No Content
  * 3xx → 리다이렉션 \
    \- 304 : 요청된 리소스를 재전송할 필요가 없음을 나타낸다. 캐시된 자원으로의 암묵적인 리디렉션이다.\
    \- 304 Not Modified가 특수한 형태로 자주 보임.
  * 4xx → 클라이언트 쪽 문제 ⇒ 404 Not Found
  * 5xx → 서버 쪽 문제 ⇒ 500 Internal Server Error
