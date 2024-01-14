# JSP

## 1. JSP 처리 과정

\-> WAS 는 JSP 페이지에 대한 요청이 들어오면 다음과 같은 처리를 한다.&#x20;

* **JSP에 해당하는 서블릿이 존재하지 않을 시**
  * **JSP 페이지로부터 자바 코드(.java)를 생성한다.**
  * **자바 코드를 컴파일해서 서블릿 클래스(.class) 를 생성한다.**\
    **-> JSP 도 내부적으로는 서블릿으로 돌아간다는 것을 생각해야 한다.**&#x20;
  * 서블릿에 클라이언트 요청을 전달한다.
  * 서블릿이 요청을 처리한 결과를 응답으로 생성한다.&#x20;
  * 응답을 웹 브라우저에 전달한다. &#x20;
* **JSP에 해당하는 서블릿이 존재하는 경우**
  * 서블릿에 클라이언트 요청을 전달한다.
  * 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
  * 응답을 웹 브라우저에 전송한다.&#x20;

## 2. 출력 버퍼와 응답

* JSP 페이지는 응답 결과를 곧바로 웹 브라우저에 전송하지 않는다. \
  \-> 대신 **`출력 버퍼(Buffer)`**라고 불리는 곳에 임시로 응답 결과를 저장했다가 **`한번에 웹 브라우저에 전송한다.`**
* 이유는 다음과 같다.&#x20;
  * **성능 향상 : 작은 단위로 데이터를 전송하는 것이 아닌, 한번에 큰 단위로 데이터를 전송할 수 있다.**&#x20;
  * **에러 페이지 처리기능이 가능 : JSP 페이지가 생성한 내용이 있더라도, 버퍼에 저장된 데이터가 웹 브라우저로 전송되기 이전까지 버퍼에 저장된 내용을 지우고 새로운 내용을 전달할 수 있다.** \
    **-> ex) JSP 실행 과정에서 에러가 발생하면, 지금까지 생성한 내용을 버퍼에서 지우고 에러 화면을 출력할 수 있다.**
  * **버퍼가 다 차기 전에 헤더 정보를 변경 가능**\
    **-> HTTP 프로토콜의 구조 상 응답 데이터를 보내기 이전에 응답 헤더값을 먼저 보내야 한다. (상태코드 등..)**\
    **-> 버퍼의 값이 전송된다면 헤더값을 변경해도 적용되지 않는다.**  &#x20;
* 기본적으로 적용되어 있는 `autoFlush=true` 설정을 통해 **버퍼에 담겨 있는 값들이 자동적으로 Flush 된다.**

## # 주의할 점

* JSP 파일을 저장할 때 사용하는 인코딩 타입과, contentType 의 charset 이 일치해야 정상적인 데이터를 확인할 수 있다. \
  **-> pageEncoding : 페이지가 작성될 때 사용되는 문자 인코딩**\
  **-> chatset : 웹 브라우저가 서버로부터 받는 응답을 어떻게 처리할 지**\
  \-> pageEncoding 속성을 가지고 있다면, 파일을 읽어올 때 속성값을 캐릭터 셋으로 사용한다.&#x20;
* 톰캣과 같은 컨테이너는 JSP 코드를 분석하는 과정에서 어떤 인코딩을 이용해서 코드를 작성했는지 검사하며, 그 결과로 선택한 캐릭터 셋을 이용해서 JSP 페이지의 문자를 읽어오게 된다.&#x20;