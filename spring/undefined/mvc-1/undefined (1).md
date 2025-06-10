# 서블릿

## 1. Hello 서블릿&#x20;

### 스프링 부트 서블릿 환경 구성&#x20;

`@ServletComponentScan`&#x20;

스프링 부트는 서블릿을 직접 등록해서 사용할 수 있도록 `@ServletComponentScan` 을 지원한다. 다음과 같이 추가하자.&#x20;

```java
package hello.servlet;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletComponentScan;

@ServletComponentScan	// 서블릿 자동 등록
@SpringBootApplication
public class ServletApplication {

	public static void main(String[] args) {
		SpringApplication.run(ServletApplication.class, args);
	}
}
```

#### 서블릿 등록하기&#x20;

HTTP 요청을 통해 매핑된 URL 이 호출되면 서블릿 컨테이너는 다음 메서드를 실행한다. \
`protected void service(HttpServletRequest request, HttpServletResponse response)`

```java
package hello.servlet.basic;

import jakarta.servlet.ServletException;
import jakarta.servlet.annotation.WebServlet;
import jakarta.servlet.http.HttpServlet;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;

import java.io.IOException;

@WebServlet(name = "helloServlet", urlPatterns = "/hello")
public class HelloServlet extends HttpServlet {

    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {

        System.out.println("HelloServlet.service");
        System.out.println("request = " + request);
        System.out.println("response = " + response);

        String username = request.getParameter("username");
        System.out.println("username = " + username);

        response.setContentType("text/plain");
        response.setCharacterEncoding("utf-8");
        response.getWriter().write("hello " + username);
    }
}
```

### 서블릿 컨테이너 동작 방식 설명&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-09 13.01.18.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-09 13.01.41.png" alt=""><figcaption></figcaption></figure>

## 2. HttpServletRequest &#x20;

#### HttpServletRequest 역할&#x20;

HTTP 요청 메시지를 개발자가 직접 파싱해서 사용해도 되지만, 매우 불편할 것이다. 서블릿은 HTTP 요청 메시지를 편리하게 관리할 수 있도록 개발자 대신 HTTP 요청 메시지를 파싱한다. 그리고 그 결과를 `HttpServletRequest` 객체에 담아서 제공한다.&#x20;

#### 임시 저장소 기능&#x20;

* 해당 HTTP 요청이 시작부터 끝날 때 까지 유지되는 임시 저장소 기능&#x20;
  * 저장 : `request.setAttribute(name, value)`
  * 조회 : `request.getAttribute(name)`

#### 세션 관리 기능&#x20;

* `request.getSession(create : true)`&#x20;

### HttpServletRequest - 기본 사용법&#x20;

필요할 때 찾아보자.&#x20;

### HTTP 요청 데이터 - 개요&#x20;

주로 다음 3가지 방법을 사용한다.&#x20;

* **GET - 쿼리 파라미터**
  * /ur&#x6C;**?username=hello\&age=20**
  * 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  * 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
* **POST - HTML Form**
  * content-type: application/x-www-form-urlencoded
  * 메시지 바디에 쿼리 파리미터 형식으로 전달 username=hello\&age=20
  * 예) 회원 가입, 상품 주문, HTML Form 사용
* **HTTP message body**에 데이터를 직접 담아서 요청
  * HTTP API에서 주로 사용, JSON, XML, TEXT
  * 데이터 형식은 주로 JSON 사용
  * POST, PUT, PATCH&#x20;

## 3. HttpServletResponse&#x20;

필요할 때 찾아보자.&#x20;
