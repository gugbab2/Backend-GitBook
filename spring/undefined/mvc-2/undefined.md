# 예외 처리와 오류 페이지

## 1. 서블릿 예외 처리 - 시작&#x20;

스프링이 아닌 순서 서블릿 컨테이너는 예외를 어떻게 처리하는지 알아보자.&#x20;

#### 서블릿은 다음 2가지 방식으로 예외 처리를 지원한다.&#x20;

* `Exception(예외)`&#x20;
* `response.sendError(HTTP 상태 코드, 오류 메시지)`&#x20;

### Exception(예외)&#x20;

#### 자바 직접 실행&#x20;

자바의 메인 메서드를 직접 실행하는 경우 `main` 이라는 이름의 쓰레드가 실행된다. \
실행 도중 예외를 잡지 못하고 처음 실행한 `main()` 메서드를 넘어서 예외가 던져지면, 예외 정보를 남기고 해당 쓰레드는 종료된다.&#x20;

#### 웹 애플리케이션&#x20;

웹 애플리케이션은 사용자 요청별로 별도의 쓰레드가 할당되고, 서블릿 컨테이너 안에서 실행된다.\
애플리케이션에서 예외가 발생했는데, 어디선가 `try-catch` 로 예외를 잡아서 처리하면 아무런 문제가 없다. \
그런데 만약에 애플리케이션에서 예외를 잡지 못하고 서블릿 밖으로 예외가 전달되면 어떻게 동작할까?&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 14.05.44.png" alt=""><figcaption></figcaption></figure>

결국 톰캣 같은 WAS 까지 예외가 전달된다. WAS 는 예외가 올라오면 어떻게 처리해야 할까?&#x20;

먼저 스프링 부트가 제공하는 기본 예외 페이지가 있는데 이건 꺼두자(나중에 설명하겠다)&#x20;

**application.properties**

```properties
server.error.whitelabel.enabled=false
```

#### ServletExController - 서블릿 예외 컨트롤러

```java
package hello.exception.servlet;

import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;

@Slf4j
@Controller
public class ServletExController {

    @GetMapping("/error-ex")
    public void errorEx() {
        throw new RuntimeException("예외 발생!");
    }
    
}
```

실행해보면 tomcat(WAS) 에서 기본적으로 제공하는 오류 화면을 볼 수 있다.&#x20;

```html
HTTP Status 500 – Internal Server Error
```

웹 브라우저에서 개발자 모드로 확인해보면 HTTP 상태 코드가 500으로 보인다. \
`Exception` 의 경우 서버 내부에서 처리할 수 없는 오류가 발생한 것으로 생각해서 HTTP 상태 코드 500을 반환한다.&#x20;

이번에는 아무 사이트나 호출해보면 tomcat(WAS)이 기본적으로 제공하는 404 오류 화면을 확인할 수 있다.

`http://localhost:8080/no-page`

```html
HTTP Status 404 – Not Found
```

### response.sendError(HTTP 상태 코드, 오류 메시지)&#x20;

오류가 발생했을 때 `HttpServletResponse` 가 제공하는 `sendError` 라는 메서드를 사용해도 된다. 이것을 호출 \
한다고 당장 예외가 발생하는 것은 아니지만, 서블릿 컨테이너(WAS)에게 오류가 발생했다는 점을 전달할 수 있다.

이 메서드를 사용하면 HTTP 상태 코드와 오류 메시지도 추가할 수 있다.

* `response.sendError(HTTP 상태 코드)`
* `response.sendError(HTTP 상태 코드, 오류 메시지)`&#x20;

#### ServletExController - 추가&#x20;

```java
@GetMapping("/error-404")
public void error404(HttpServletResponse response) throws IOException {
    response.sendError(404, "404 오류!");
}

@GetMapping("/error-500")
public void error500(HttpServletResponse response) throws IOException {
    response.sendError(500);
}
```

sendError 흐름&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 16.56.59.png" alt=""><figcaption></figcaption></figure>

`response.sendError()` 를 호출하면 `response` 내부에는 오류가 발생했다는 상태를 저장해둔다. 그리고 서블릿 컨테이너(WAS)는 고객에게 응답 전에 `response` 에 `sendError()` 가 호출되었는지 확인한다. 그리고 호출 되었다면 설정한 오류 코드에 맞추어 기본 오류 페이지를 보여준다.

정리

서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 사용자가 보기에 불편하다. 의미 있는 오류 화면을 제공해보자.&#x20;

## 2. 서블릿 예외 처리 - 오류 화면 제공&#x20;

서블릿 컨테이너가 제공하는 기본 예외 처리 화면은 고객 친화적이지 않다. 서블릿이 제공하는 오류 화면 기능을 사용해보자.&#x20;

서블릿은 `Exception`(예외) 가 발생해서 서블릿이 밖으로 전달되거나 또는 `response.sendError()` 가 호출 되었을 때 각각의 상황에 맞춘 오류 처리 기능을 제공한다.&#x20;

이 기능을 사용하면 친절한 오류 처리 화면을 준비해서 고객에게 보여줄 수 있다.

지금은 스프링 부트를 통해서 서블릿 컨테이너를 실행하기 때문에, 스프링 부트가 제공하는 기능을 사용해서 서블릿 오류 페이지를 등록하면 된다.

#### 서블릿 오류 페이지 등록&#x20;

```java
package hello.exception;

import org.springframework.boot.web.server.ConfigurableWebServerFactory;
import org.springframework.boot.web.server.ErrorPage;
import org.springframework.boot.web.server.WebServerFactoryCustomizer;
import org.springframework.http.HttpStatus;
import org.springframework.stereotype.Component;

@Component
public class WebServerCustomerizer implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {

    @Override
    public void customize(ConfigurableWebServerFactory factory) {

        ErrorPage errorPage404 = new ErrorPage(HttpStatus.NOT_FOUND, "/error-page/404");
        ErrorPage errorPage500 = new ErrorPage(HttpStatus.INTERNAL_SERVER_ERROR, "/error-page/500");

        ErrorPage errorPageEx = new ErrorPage(RuntimeException.class, "/error-page/500");

        factory.addErrorPages(errorPage404, errorPage500, errorPageEx);
    }
}
```

500 예외가 서버 내부에서 발생한 오류라는 뜻을 포함하고 있기 때문에 여기서는 예외가 발생한 경우도 500 오류 화면으로 처리했다.

오류 페이지는 예외를 다룰 때 해당 예외와 그 자식 타입의 오류를 함께 처리한다. 예를 들어서 위의 경우 `RuntimeException` 은 물론이고 `RuntimeException`의 자식도 함께 처리한다.

오류가 발생했을 때 처리할 수 있는 컨트롤러가 필요하다. 예를 들어서 `RuntimeException` 예외가 발생하면 `errorPageEx` 에서 지정한 `/error-page/500` 이 호출된다.

```java
package hello.exception.servlet;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Slf4j
@Controller
public class ErrorPageController {

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.error("errorPage404");
        printErrorInfo(request);
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage500");
        printErrorInfo(request);
        return "error-page/500";
    }
}
```

#### 오류 처리 뷰&#x20;

~~오류 처리 뷰 구현은 중요한 내용이 아니기에 패스..~~

## 3. 서블릿 예외 처리 - 오류 페이지 작동 원리&#x20;

서블릿은 `Exception` (예외)가 발생해서 서블릿 밖으로 전달되거나 또는 `response.sendError()` 가 호출 되었을 때 설정된 오류 페이지를 찾는다.

#### 예외 발생 흐름&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 17.26.09.png" alt=""><figcaption></figcaption></figure>

#### sendError 흐름&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 17.26.32.png" alt=""><figcaption></figcaption></figure>

WAS 는 해당 예외를 처리하는 오류 페이지 정보를 확인한다. \
`new ErrorPage(RuntimeException.class, "/error-page/500")`

예를 들어서 `RuntimeException` 예외가 WAS까지 전달되면, WAS는 오류 페이지 정보를 확인한다. 확인해보니`RuntimeException` 의 오류 페이지로 `/error-page/500` 이 지정되어 있다. WAS는 오류 페이지를 출력하기 위해 `/error-page/500` 를 다시 요청한다.

#### 오류 페이지 요청 흐름&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 17.27.44.png" alt=""><figcaption></figcaption></figure>

#### 예외 발생과 오류 페이지 요청 흐름&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 17.28.01.png" alt=""><figcaption></figcaption></figure>

**중요한 점은 웹 브라우저(클라이언트)는 서버 내부에서 이런 일이 일어나는지 전혀 모른다는 점이다. 오직 서버 내부에서 오휴 페이지를 찾기 위해 추가적인 호출을 한다.**&#x20;

### 오류 정보 추가&#x20;

WAS 는 오류 페이지를 단순히 요청만 하는 것이 아니라, 오류 정보를 `request` 의 `attribute` 에 추가해서 넘겨준다. \
필요하면 오류 페이지에서 이렇게 전달된 오류 정보를 사용할 수 있다.&#x20;

**ErrorPageController - 오류 출력**

```java
package hello.exception.servlet;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Slf4j
@Controller
public class ErrorPageController {

    //RequestDispatcher 상수로 정의되어 있음
    public static final String ERROR_EXCEPTION = "jakarta.servlet.error.exception";
    public static final String ERROR_EXCEPTION_TYPE = "jakarta.servlet.error.exception_type";
    public static final String ERROR_MESSAGE = "jakarta.servlet.error.message";
    public static final String ERROR_REQUEST_URI = "jakarta.servlet.error.request_uri";
    public static final String ERROR_SERVLET_NAME = "jakarta.servlet.error.servlet_name";
    public static final String ERROR_STATUS_CODE = "jakarta.servlet.error.status_code";

    @RequestMapping("/error-page/404")
    public String errorPage404(HttpServletRequest request, HttpServletResponse response) {
        log.error("errorPage404");
        printErrorInfo(request);
        return "error-page/404";
    }

    @RequestMapping("/error-page/500")
    public String errorPage500(HttpServletRequest request, HttpServletResponse response) {
        log.info("errorPage500");
        printErrorInfo(request);
        return "error-page/500";
    }

    private void printErrorInfo(HttpServletRequest request) {
        log.info("ERROR_EXCEPTION: {}", request.getAttribute(ERROR_EXCEPTION));
        log.info("ERROR_EXCEPTION_TYPE: {}", request.getAttribute(ERROR_EXCEPTION_TYPE));
        log.info("ERROR_MESSAGE: {}", request.getAttribute(ERROR_MESSAGE));
        log.info("ERROR_REQUEST_URI: {}", request.getAttribute(ERROR_REQUEST_URI));
        log.info("ERROR_SERVLET_NAME: {}", request.getAttribute(ERROR_SERVLET_NAME));
        log.info("ERROR_STATUS_CODE: {}", request.getAttribute(ERROR_STATUS_CODE));
        log.info("dispatchType={}", request.getDispatcherType());
    }
}
```

## 4. 서블릿 예외 처리 - 필터&#x20;

#### 목표&#x20;

예외 처리에 필터와 인터셉터 그리고 서블릿이 제공하는 `DispatchType` 이해하기

오류가 발생하면 오류 페이지를 출력하기 위해 WAS 내부에서 다시 한번 호출이 발생한다. 이때 필터, 서블릿, 인터셉터도 모두 다시 호출된다. 그런데 로그인 인증 체크 같은 경우를 생각해보면, 이미 한번 필터나, 인터셉터에서 로그인 체크를 완료했다. 따라서 서버 내부에서 오류 페이지를 호출한다고 해서 해당 필터나 인터셉트가 한번 더 호출되는 것은 매우 비효율적이다.

결국 클라이언트로 부터 발생한 정상 요청인지, 아니면 오류 페이지를 출력하기 위한 내부 요청인지 구분할 수 있어야 한다. 서블릿은 이런 문제를 해결하기 위해 `DispatcherType` 이라는 추가 정보를 제공한다.

### DispatcherType&#x20;

필터는 이런 경우를 위해서 `dispatcherTypes` 라는 옵션을 제공한다.

이전 강의의 마지막에 다음 로그를 추가했다.\
`log.info("dispatchType={}", request.getDispatcherType())`&#x20;

그리고 출력해보면 오류 페이지에서 `dispatchType=ERROR`로 나오는 것을 확인할 수 있다.

고객이 처음 요청하면 `dispatcherType=REQUEST` 이다.\
이렇듯 서블릿 스펙은 실제 고객이 요청한 것인지, 서버가 내부에서 오류 페이지를 요청하는 것인지 `DispatcherType`으로 구분할 수 있는 방법을 제공한다.

`javax.servlet.DispatcherType`

```java
public enum DispatcherType {
    FORWARD,
    INCLUDE,
    REQUEST,
    ASYNC,
    ERROR
}
```

### 필터와 DispatcherType&#x20;

필터와 `DispatcherType`이 어떻게 사용되는지 알아보자.

#### LogFilter - DispatcherType 로그 추가

```java
package hello.exception.filter;

import jakarta.servlet.*;
import jakarta.servlet.http.HttpServletRequest;
import lombok.extern.slf4j.Slf4j;

import java.io.IOException;
import java.util.UUID;

@Slf4j
public class LogFilter implements Filter {

    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        log.info("log filter init");
    }

    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain filterChain) throws IOException, ServletException {
        log.info("log filter doFilter");

        HttpServletRequest httpRequest = (HttpServletRequest) request;
        String requestURI = httpRequest.getRequestURI();

        String uuid = UUID.randomUUID().toString();

        try {
            log.info("REQUEST [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
            filterChain.doFilter(request, response); // 다음 필터로 요청을 전달
        } catch (Exception e) {
            log.info("EXCEPTION [{}]", e.getMessage());
            throw e;
        } finally {
            log.info("RESPONSE [{}][{}][{}]", uuid, request.getDispatcherType(), requestURI);
        }
    }

    @Override
    public void destroy() {
        log.info("log filter destroy");
    }
}
```

#### WebConfig

```java
package hello.exception;

import hello.exception.filter.LogFilter;
import hello.exception.interceptor.LogInterceptor;
import jakarta.servlet.DispatcherType;
import jakarta.servlet.Filter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterRegistrationBean;
    }

}
```

`filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);`

이렇게 두 가지를 모두 넣으면 클라이언트 요청은 물론이고, 오류 페이지 요청에서도 필터가 호출된다.

아무것도 넣지 않으면 기본 값이 `DispatcherType.REQUEST`이다. 즉 클라이언트의 요청이 있는 경우에만 필터가 적용된다. 특별히 오류 페이지 경로도 필터를 적용할 것이 아니면, 기본 값을 그대로 사용하면 된다. \
(물론 오류 페이지 요청 전용 필터를 적용하고 싶으면 `DispatcherType.Error` 만 지정하면 된다)

## 5. 서블릿 예외 처리 - 인터셉터&#x20;

### 인터셉터 중복 호출 제거&#x20;

#### LogInterceptor - DispatcherType 로그 추가&#x20;

```java
package hello.exception.interceptor;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.web.servlet.HandlerInterceptor;
import org.springframework.web.servlet.ModelAndView;

import java.util.UUID;

@Slf4j
public class LogInterceptor implements HandlerInterceptor {

    public static final String LOG_ID = "logId";

    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        String requestURI = request.getRequestURI();
        String uuid = UUID.randomUUID().toString();
        request.setAttribute(LOG_ID, uuid);
        log.info("REQUEST [{}][{}][{}][{}]", uuid, request.getDispatcherType(),
                requestURI, handler);
        return true;
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response,
                           Object handler, ModelAndView modelAndView) throws Exception {
        log.info("postHandle [{}]", modelAndView);
    }

    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                                Object handler, Exception ex) throws Exception {
        String requestURI = request.getRequestURI();
        String logId = (String)request.getAttribute(LOG_ID);
        log.info("RESPONSE [{}][{}][{}]", logId, request.getDispatcherType(), requestURI);

        if (ex != null) {
            log.error("afterCompletion error!!", ex);
        }
    }
}
```

앞서 필터의 경우에는 필터를 등록할 때 어떤 `DispatcherType` 인 경우에 필터를 적용할 지 선택할 수 있었다. 그런데 인터셉터는 서블릿이 제공하는 기능이 아니라 스프링이 제공하는 기능이다. 따라서 `DispatcherType` 과 무관하게 항상 호출된다.

대신에 인터셉터는 다음과 같이 요청 경로에 따라서 추가하거나 제외하기 쉽게 되어 있기 때문에, 이러한 설정을 사용해서 오류 페이지 경로를 `excludePathPatterns` 를 사용해서 빼주면 된다.\
(스프링 인터셉터가 강력한 이유!)&#x20;

```java
package hello.exception;

import hello.exception.filter.LogFilter;
import hello.exception.interceptor.LogInterceptor;
import jakarta.servlet.DispatcherType;
import jakarta.servlet.Filter;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.InterceptorRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new LogInterceptor())
                .order(1)
                .addPathPatterns("/**")
                .excludePathPatterns("/css/**", "/*.ico", "/error", "/error-page/**");
    }

//    @Bean
    public FilterRegistrationBean logFilter() {
        FilterRegistrationBean<Filter> filterRegistrationBean = new FilterRegistrationBean<>();
        filterRegistrationBean.setFilter(new LogFilter());
        filterRegistrationBean.setOrder(1);
        filterRegistrationBean.addUrlPatterns("/*");
        filterRegistrationBean.setDispatcherTypes(DispatcherType.REQUEST, DispatcherType.ERROR);
        return filterRegistrationBean;
    }

}
```

#### 전체 흐름 정리&#x20;

`/hello` 정상 요청&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-16 17.38.49.png" alt=""><figcaption></figcaption></figure>

`/error-ex` 오류 요청

* 필터는 `DispatchType` 으로 중복 호출 제거 (`dispatchType=REQUEST`)
* 인터셉터는 경로 정보로 중복 호출 제거(`excludePathPatterns("/error-page/**")` )

```
1. WAS(/error-ex, dispatchType=REQUEST) -> 필터 -> 서블릿 -> 인터셉터 -> 컨트롤러
2. WAS(여기까지 전파) <- 필터 <- 서블릿 <- 인터셉터 <- 컨트롤러(예외발생)
3. WAS 오류 페이지 확인
4. WAS(/error-page/500, dispatchType=ERROR) -> 필터(x) -> 서블릿 -> 인터셉터(x) -> 컨트롤러(/error-page/500) -> View
```

## 6. 스프링 부트 - 오류 페이지1&#x20;

지금까지 예외 처리 페이지를 만들기 위해서 다음과 같은 복잡한 과정을 거쳤다.

* `WebServerCustomizer` 를 만들고
* 예외 종류에 따라서 `ErrorPage` 를 추가하고
  * 예외 처리용 컨트롤러 `ErrorPageController` 를 만듬

#### 스프링 부트는 이런 과정을 모두 기본으로 제공한다.&#x20;

* `ErrorPage` 를 자동으로 등록한다. 이때 `/error` 라는 경로로 기본 오류 페이지를 설정한다.
  * `new ErrorPage("/error")`, 상태코드와 예외를 설정하지 않으면 기본 오류 페이지로 사용된다.
  *   서블릿 밖으로 예외가 발생하거나, `response.sendError(...)` 가 호출되면 모든 오류는 `/error` 를

      호출하게 된다.
* `BasicErrorController` 라는 스프링 컨트롤러를 자동으로 등록한다.
  * `ErrorPage` 에서 등록한 `/error` 를 매핑해서 처리하는 컨트롤러다.

#### 주의&#x20;

스프링 부트가 제공하는 기본 오류 메커니즘을 사용하도록 **WebServerCustomizer**에 있는 `@Component` 를 주석 처리하자.&#x20;

#### 개발자는 오류 페이지만 등록

`BasicErrorController` 는 기본적인 로직이 모두 개발되어 있다.

개발자는 오류 페이지 화면만 `BasicErrorController` 가 제공하는 룰과 우선순위에 따라서 등록하면 된다. 정적 HTML이면 정적 리소스, 뷰 템플릿을 사용해서 동적으로 오류 화면을 만들고 싶으면 뷰 템플릿 경로에 오류 페이지 파일application/json을 만들어서 넣어두기만 하면 된다.

뷰 선택 우선순위

`BasicErrorController` 의 처리 순서

1. 뷰 템플릿
   1. `resources/templates/error/500.html`
   2. `resources/templates/error/5xx.html`
2. 2\. 정적 리소스(`static`, `public`)
   1. `resources/static/error/400.html`&#x20;
   2. `resources/static/error/404.html`
   3. `resources/static/error/4xx.html`
3. 적용 대상이 없을 때 뷰 이름(`error`)
   1. `resources/templates/error.html`

~~그 외 내용은 뷰 템플릿 관련 내용이기에 패스..~~
