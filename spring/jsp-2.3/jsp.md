# JSP

## 1. JSP 란?&#x20;

* JSP는 동적인 웹서버에서 동적인 페이지를 만들어 주는 서버 사이드 스크립트 언어다.
*   설명만 들으면 서블릿(Servlet)과 같은 기능을 하는 것처럼 보인다.

    (실제로 기능적으로는 굉장히 비슷하다)
* 하지만 아주 큰 차이가 있는데 작성하는 언어의 기반이 다르다는 것이다.
* 서블릿이 클래스의 형태를 띄고 있고 자바의 형태를 온전히 가져가는데 반해
* JSP는 HTML 코드를 기반으로 그 사이에 자바코드를 삽입하는 식으로 만든다.

### JSP 등장 배경

* 아래와 같은 HTML 코드가 있다고 생각하자&#x20;

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>
	<form action="TestServlet" method="get">
		이름: <input type="text" name="name"><br><br>
		나이: <input type="text" name="age"><br><br>
		사는곳: <input type="text" name="place"><br><br>
		<input type="submit" value="전송">
	</form>
</body>
</html>
```

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 17.04.17.png" alt="" width="446"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 17.04.26.png" alt="" width="453"><figcaption></figcaption></figure>

* 위와 같은 결과를 출력하는 코드를 서블릿에서 작성하게 되면 아래와 같다.\
  (너무 복잡하다.. )&#x20;

```java
@SuppressWarnings("serial")
@WebServlet("/TestServlet")
public class TestServlet extends HttpServlet {

	protected void doGet(HttpServletRequest request, HttpServletResponse response) 
    	throws ServletException, IOException {
        
		response.setContentType("text/html; charset=UTF-8");
		PrintWriter out = response.getWriter(); 
		
		String name = request.getParameter("name");
		String age = request.getParameter("age");
		String place = request.getParameter("place");
		
		out.println("<html>");
		out.println("<head>");
		out.println("</head>");
		out.println("<body>");
		out.println("<p>Hello Java!</p><br>");
		out.println("<p>당신의 이름은 "+name+"입니다.</p><br>");
		out.println("<p>당신의 나이는 "+age+"입니다.</p><br>");
		out.println("<p>당신의 사는곳은 "+place+"입니다.</p><br>");
		out.println("</body>");
		out.println("</html>");
	}

}
```

* 위 서블릿 코드를 JSP 로 작성하게 되면 아래와 같다. \
  (가독성이 좋다!)

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<body>

<%! String name; String age; String place; %>

<%
name = request.getParameter("name");
age = request.getParameter("age");
place = request.getParameter("place");
%>

<p> 당신의 이름은 <strong><%= name %></strong>입니다. </p>
<p> 당신의 나이는 <strong><%= age %></strong>입니다. </p>
<p> 당신의 사는 곳은 <strong><%= place %></strong>입니다. </p>
</body>
</html>
```

## 2. JSP 처리 과정

\-> WAS 는 JSP 페이지에 대한 요청이 들어오면 다음과 같은 처리를 한다.

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 17.09.06.png" alt=""><figcaption></figcaption></figure>

* **JSP에 해당하는 서블릿이 존재하지 않을 시**
  * **JSP 페이지로부터 자바 코드(.java)를 생성한다.**
  * **자바 코드를 컴파일해서 서블릿 클래스(.class) 를 생성한다.**\
    **-> JSP 도 내부적으로는 서블릿으로 돌아간다는 것을 생각해야 한다.**
  * 서블릿에 클라이언트 요청을 전달한다.
  * 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
  * 응답을 웹 브라우저에 전달한다.
* **JSP에 해당하는 서블릿이 존재하는 경우**
  * 서블릿에 클라이언트 요청을 전달한다.
  * 서블릿이 요청을 처리한 결과를 응답으로 생성한다.
  * 응답을 웹 브라우저에 전송한다.

## 3. 출력 버퍼와 응답

* JSP 페이지는 응답 결과를 곧바로 웹 브라우저에 전송하지 않는다.\
  \-> 대신 `출력 버퍼(Buffer)`라고 불리는 곳에 임시로 응답 결과를 저장했다가 버퍼가 다 차면, **`한번에 웹 브라우저에 전송한다.`**
* 이유는 다음과 같다.
  * **성능 향상 : 작은 단위로 데이터를 전송하는 것이 아닌, 한번에 큰 단위로 데이터를 전송할 수 있다.**
  * **에러 페이지 처리 기능이 가능 : JSP 페이지가 생성한 내용이 있더라도, 버퍼에 저장된 데이터가 웹 브라우저로 전송되기 이전까지 버퍼에 저장된 내용을 지우고 새로운 내용을 전달할 수 있다.**
    * **예를 들어, JSP 실행 과정에서 에러가 발생하면, 지금까지 생성한 내용을 버퍼에서 지우고 에러 화면을 출력할 수 있다.**
  * **버퍼가 다 차기 전에 헤더 정보를 변경 가능**
    * **HTTP 프로토콜의 구조 상 응답 데이터를 보내기 이전에 응답 헤더값을 먼저 보내야 한다. (상태코드 등..)**
    * **버퍼의 값이 전송된다면 헤더값을 변경해도 적용되지 않는다.**
* 기본적으로 적용되어 있는 `autoFlush=true` 설정을 통해 **버퍼에 담겨 있는 값들이 자동적으로 Flush 된다.**

## # 주의할 점

* JSP 파일을 저장할 때 사용하는 인코딩 타입과, contentType 의 charset 이 일치해야 정상적인 데이터를 확인할 수 있다.
  * pageEncoding : 페이지가 작성될 때 사용되는 문자 인코딩
  * chatset : 웹 브라우저가 서버로부터 받는 응답을 어떻게 처리할 지
  * pageEncoding 속성을 가지고 있다면, 파일을 읽어올 때 속성값을 캐릭터 셋으로 사용한다.
* **톰캣과 같은 웹 컨테이너는 JSP 코드를 분석하는 과정에서 어떤 인코딩을 이용해서 코드를 작성했는지 검사하며, 그 결과로 선택한 캐릭터 셋을 이용해서 JSP 페이지의 문자를 읽어오게 된다.**
