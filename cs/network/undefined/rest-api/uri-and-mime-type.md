# URI & MIME type

## [URI](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics\_of\_HTTP/Identifying\_resources\_on\_the\_Web)(Uniform Resource Identifier)

\-> 리소스를 식별하는 방법(식별자의 역할)\
\-> URI 는 대부분 URL 을 의미한다.(동의어에 가깝다)

* #### URI 를 나누는 방법
  * URN : 리소스의 유니크한 이름, 사실상 안쓰임
  * URL : 리소스의 위치(유니크), 위치 변경에 취약하다;;

## [MIME Type](https://developer.mozilla.org/ko/docs/Web/HTTP/Basics\_of\_HTTP/MIME\_types)(Content Type, Media Type)

\-> 가장 범용적으로 사용되는 타입\
\-> Content의 종류를 표현. \<type>/\<subtype>의 형태로 쓴다. HTTP Headers에 Content-Type 속성으로 전달함. [IANA](https://www.iana.org/assignments/media-types/media-types.xhtml)에 등록된 목록을 참고하자.

1. `text/plain` ⇒ E-mail에서 자주 사용.
2. `text/html` ⇒ 일반적인 웹 문서. HTML 문서.
3. `text/css`
4. `text/javascript`
5. `application/xml` ⇒ 범용. 자기서술적이기 상대적으로 어렵다.
6. `application/atom+xml`
7. `application/json` ⇒ 범용. 자기서술적이기 굉장히 어렵다.
8. `application/dns+json`
