---
description: HTTP 헤더를 이해해보자
---

# HTTP Header

## 일반 헤더

* ### 표현(최근에는 엔티티라는 개념 대신 표현이라는 개념을 사용한다 / RFC7230\~)
  * Content-Type : 표현의 데이터의 형식 -> text/html, application/json, image/png
  * Content-Encodong : 표현 데이터의 압축 방식 -> gzip, deflate, identity
  * Content-Language : 표현 데이터의 자연 언어 -> ko, en, en-US
  * Content-Length : 표현 데이터의 길이 -> 바이트 단위
*   ### 콘텐츠 협상(요청시에만 사용한다!)

    \-> 0\~1 사이에 q 우선순위를 사용해야 하며, 클수록 높은 우선순위를 갖는다.\
    \-> 구체적인 것이 우선한다.

    * Accept : 클라이언트가 선호하는 미디어 타입 전달&#x20;
    * Accept-Charset : 클라이언트가 선호하는 문자 인코딩&#x20;
    * Accept-Encoding : 클라이언트가 선호하는 압축 인코딩
    * Accept-Language : 클라이언트가 선호나는 자연 언어
* ### 전송 방식
  * 단순 전송 -> 일반적인 전송방식
  * 압축 전송&#x20;
    * 압축을 해 전송하는데, Content-Encoding 헤더값을 넣어주어야 한다.
  * 분할 전송
    * 말 그대로 분할하여 전송하는 것이다.&#x20;
    * Transfer-Encoding : chunked 헤더 값을 넣어주어야 하고, Content-Length(크기가 가변적이다) 를 지정할 수 없다. &#x20;
  * 범위 전송&#x20;
    * Content-Range 헤더값을 통해서 범위 만큼의 데이터만 보낼 수 있다.
* ### 일반 정보
  * Referer : 이전 요청의 URI 를 알려준다 -> 유입 경로를 분석이 가능하다(요청에서 사용)
  * User-Agent : 클라이언트 애플리케이션 정보(웹 브라우저 정보 등등..)
  * Server : 요청을 처리하는 Origin 서버의 소프트웨어 정보(응답에서 사용)
  * Date : 메시지가 발생한 날짜와 시간(응답에서 사용)
* ### 특별한 정보
  * **Host : 요청에서 사용하는 필수헤더이다!!!** -> 하나의 서버가 여러 도메인을 처리해야 할 때
  * Location : 리다이렉션 해야하는 대상 리소스를 나타내는 헤더 정보, 201(Created) 에서 요청에 의해 생성된 리소스를 뜻하기도 한다.
  * Allow : 허용 가능한 HTTP 메서드
  * Retry-After : 503 오류 시 클라이언트가 다음 요청까지 기다려야 하는 시간을 나타낸다.
* ### 인증&#x20;
  * Authorization : 클라이언트 인증 정보를 서버에 전달
  * www-Authenticate : 리소스 접근시 필요한 인증 방법 정의
* ### 쿠키
  * 쿠키 사용 이전에는 모든 요청에 사용자의 정보를 보냈다. -> 개발도 힘들고 보안의 문제가 있다.
  * 서버가 유저의 정보를 담은 쿠키를 클라이언트에게 보내준다. -> 웹 브라우저 쿠키 저장소에 저장한다.
  * 해당 서버에 요청을 보낼 때마다 무조건적으로 쿠키를 요청과 함께 보내준다. -> 서버에게 사용자 정보를 보낼 수 있다.&#x20;
  * 쿠키방식도 보안의 문제가 있기 때문에, 세션과 관련된 키를 쿠키에 담에서 클라이언트에게 보내준다.
  * 보안에 민감한 데이터를 저장해두면 안된다!
  * 생명주기를 지정해야 한다. (세션쿠키, 영속쿠키)
  * 도메인, 경로, 보안을 지정할 수 있다.

## 캐시와 조건부 요청

* ### 캐시 기본 동작
  * 캐시가 없다면 데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운받아야 하기 때문에, 느린 사용자 경험을 하게된다.
  * 캐시를 사용하게 되면, 브라우저에 응답 결과를 캐시에 저장하게 된다. ex) cache-control: max-age=60\
    \-> 캐시가 유효한 시간 동안은 캐시 저장소에서 자료를 가져올 수 있다. &#x20;
  * 캐시 유효시간이 지나 다시 서버에 요청하게 될 때, 서버에서 기존 데이터를 변경하지 않는다면 불필요한 네트워크 리소스를 소모한다고 볼 수 있다. 아래 방법을 통해서 개선할 수 있다.
* ### 검증 헤더와 조건부 요청1(Last-Modified)
  * Last-Modified : 해당 데이터가 마지막으로 수정 된 시간
  * 서버가 헤더에 Last-Modified 정보와 함께 클라이언트에게 보내준다.
  * 클라이언트는 캐시 유효시간이 지나고 Last-Modified 정보를 함께 서버에게 요청한다.
  * 서버는 Last-Modified 를 통해서 정보가 변경되지 않았다면, Message Body 가 없이 304(Not Modified) 메시지를 클라이언트에게 전달한다.
  * 클라이언트는 응답 결과를 재사용한다. (Message Body 만큼의 리소스를 절약했다, 매우 실용적이다)
* ### 검증 헤더와 조건부 요청2(ETag)
  * 데이터를 수정해서 날짜가 다르지만, 컨텐츠는 같은 경우 / 서버에서 별도의 캐시 로직을 관리하고 싶은 경우
  * 해당 경우에 Last-Modified 가 아닌, ETag 를 사용할 수 있다.
  * 캐시용 데이터에 임의의 고유한 버전 이름을 달아둔다. -> 데이터가 변경되면 버전 이름을 바꾸어서 변경한다.
  * 캐시 제어 로직을 서버에서 완전하게 관리할 수 있다!!
* ### 캐시와 조건부 요청 헤더
  * Cache-Control : max-age
  * Cache-Control : no-cache -> 데이터는 캐시해도 되지만, 항상 원서버에 검증하고 사용해라,,
  * Cache-Control : no-store  -> 데이터에 민감한 정보가 있으므로 저장하면 안됨(메모리에서 사용하고 최대한 빨리 삭제)
  * Cache-Control : must-revalidate -> 캐시 만료 후 최초 조회시 반드시 원 서버에 검증해야 함
    * 순간적으로 프록시 캐시와 원 서버의 네트워크가 단절 될 때, 프록시 서버에서 오류 보다는 오래된 데이터를 보내 200 응답 코드를 받을수도 있다.&#x20;
    * 이러한 장애 순간에 504(Gateway Timeout) 응답 코드를 보내준다!&#x20;
  * If-Match, If-None-Match : ETag 값 사용
  * If-Modified-Since, If-Unmodified-Since : Last-Modified 값 사용
* ### 프록시 캐시
  * 예를 들어, 미국 원서버에 접근하려면 너무 오래 걸린다. 때문에, 한국 어딘가에 프록시 캐시 서버를 넣어서 클라이언트가 미국 원서버와 네트워킹 할 때  리소스를 절약한다.
* ### 캐시 무효화
  * Cache-Control: no-cache, no-store, must-revalidate (모든 요소를 빼먹지 말고 넣어주어야 한다!)
  * 해당 헤더값을 통해서 확실하게 캐시를 무효화 시킬 수 있다.&#x20;
