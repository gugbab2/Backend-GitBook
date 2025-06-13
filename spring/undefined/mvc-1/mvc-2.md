# 스프링 MVC - 기본 기능2

## 7. HTTP 요청 메시지 - 단순 텍스트&#x20;

서블릿에서 학습한 내용을 확인해보자.&#x20;

* HTTP message body 에 데이터를 직접 담아서 요청&#x20;
  * HTTP API에서 주로 사용, JSON, XML, TEXT
  * 데이터 형식은 주로 JSON 사용
  * POST, PUT, PATCH

요청 파라미터와 다르게, HTTP 메시지 바디를 통해 데이터가 직접 넘어오는 경우는 `@RequestParam`, `@ModelAttribute` 를 사용할 수 없다. (물론 HTML Form 형식으로 전달되는 경우는 요청 파라미터로 인정된다.)

#### RequestBodyStringController&#x20;

```java
package hello.springmvc.basic.request;

import jakarta.servlet.ServletInputStream;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import java.io.IOException;
import java.io.InputStream;
import java.io.Writer;
import java.nio.charset.StandardCharsets;

@Slf4j
@Controller
public class RequestBodyStringController {

    @PostMapping("/request-body-string-v1")
    public void requestBodyString(HttpServletRequest request, HttpServletResponse response) throws IOException {

        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);

        response.getWriter().write("ok");
    }
}
```

#### Input, Output 스트림, Reader - requestBodyStringV2&#x20;

```java
@PostMapping("/request-body-string-v2")
public void requestBodyStringV2(InputStream inputStream, Writer responseWriter) throws IOException {

    String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);
    log.info("messageBody={}", messageBody);
    responseWriter.write("ok");
}
```

**스프링 MVC는 다음 파라미터를 지원한다**

* `InputStream(Reader)` : HTTP 요청 메시지 바디의 내용을 직접 조회
* `OutputStream(Writer)` : HTTP 응답 메시지의 바디에 직접 결과 출력

#### HttpEntity - requestBodyStringV3

```java
@PostMapping("/request-body-string-v3")
public HttpEntity requestBodyStringV3(HttpEntity<String> httpEntity) throws IOException {

    String messageBody = httpEntity.getBody();
    log.info("messageBody={}", messageBody);

    return new HttpEntity<>("ok");
}
```

**스프링 MVC 는 다음 파라미터를 지원한다.**&#x20;

* **`HttpEntity`**: HTTP header, body 정보를 편리하게 조회
  * 메시지 바디 정보를 직접 조회
  * 요청 파라미터를 조회하는 기능과 관계 없음 `@RequestParam` X, `@ModelAttribute` X
* **`HttpEntity`는 응답에도 사용 가능**
  * 메시지 바디 정보 직접 반환
  * 헤더 정보 포함 가능
  * view 조회X

`HttpEntity` 를 상속받은 다음 객체들도 같은 기능을 제공한다.

* `RequestEntity`
  * HttpMethod, url 정보가 추가, 요청에서 사용
* `ResponseEntity`
  * HTTP 상태 코드 설정 가능, 응답에서 사용
  * `return new ResponseEntity<String>("Hello World", responseHeaders, HttpStatus.CREATED)`

> #### 참고&#x20;
>
> 스프링 MVC 내부에서 HTTP 메시지 바디를 읽어서 문자나 객체로 변환해서 전달해주는데, 이 때 HTTP 메시지 컨버터(HttpMessageConverter) 라는 기능을 사용한다.&#x20;

#### @RequestBody - requsetBodyStringV4

```java
@ResponseBody
@PostMapping("/request-body-string-v4")
public String requestBodyStringV4(@RequestBody String messageBody) throws IOException {

    log.info("messageBody={}", messageBody);

    return "ok";
}
```

**@RequestBody**&#x20;

`@RequestBody` 를 사용하면 HTTP 메시지 바디 정보를 편리하게 조회할 수 있다. 참고로 헤더 정보가 필요하다면 `HttpEntity` 를 사용하거나 `@RequestHeader` 를 사용하면 된다.\
이렇게 메시지 바디를 직접 조회하는 기능은 요청 파라미터를 조회하는 `@RequestParam`, `@ModelAttribute` 와는 전혀 관계가 없다.

**요청 파라미터 vs HTTP 메시지 바디**&#x20;

* 요청 파라미터를 조회하는 기능 : `@RequestParam`, `@ModelAttribute`&#x20;
* HTTP 메시지 바디를 직접 조회하는 기능: `@RequestBody`

**@ResponseBody**

`@ResponseBody` 를 사용하면 응답 결과를 HTTP 메시지 바디에 직접 담아서 전달할 수 있다.\
물론 이 경우에도 view를 사용하지 않는다.

## 8. HTTP 요청 메시지 - JSON&#x20;

#### RequestBodyJsonController

```java
package hello.springmvc.basic.request;

import com.fasterxml.jackson.databind.ObjectMapper;
import hello.springmvc.basic.HelloData;
import jakarta.servlet.ServletInputStream;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpEntity;
import org.springframework.stereotype.Controller;
import org.springframework.util.StreamUtils;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import java.io.IOException;
import java.nio.charset.StandardCharsets;

@Slf4j
@Controller
public class RequestBodyJsonController {

    private ObjectMapper objectMapper = new ObjectMapper();

    @PostMapping("/request-body-json-v1")
    public void requestBodyJsonV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        ServletInputStream inputStream = request.getInputStream();
        String messageBody = StreamUtils.copyToString(inputStream, StandardCharsets.UTF_8);

        log.info("messageBody={}", messageBody);
        HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
        log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

        response.getWriter().write("ok");
    }
}
```

* `HttpServletRequest`를 사용해서 직접 HTTP 메시지 바디에서 데이터를 읽어와서, 문자로 변환한다.
* 문자로 된 JSON 데이터를 Jackson 라이브러리인 `objectMapper` 를 사용해서 자바 객체로 변환한다.

#### requestBodyJsonV2 - @RequestBody 문자 변환

```java
@ResponseBody
@PostMapping("/request-body-json-v2")
public String requestBodyJsonV2(@RequestBody String messageBody) throws IOException {

    log.info("messageBody={}", messageBody);
    HelloData helloData = objectMapper.readValue(messageBody, HelloData.class);
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```

* 이전에 학습했던 `@RequestBody` 를 사용해서 HTTP 메시지에서 데이터를 꺼내고 `messageBody`에 저장한다.
* 문자로 된 JSON 데이터인 `messageBody` 를 `objectMapper` 를 통해서 자바 객체로 변환한다.

#### **requestBodyJsonV3 - @RequestBody 객체 변환**

```java
@ResponseBody
@PostMapping("/request-body-json-v3")
public String requestBodyJsonV3(@RequestBody HelloData helloData) throws IOException {

    log.info("helloData={}", helloData);
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```

**@RequestBody 객체 파라미터**

* `@RequestBody HelloData data`&#x20;
* `@RequestBody` 에 직접 만든 객체를 지정할 수 있다.

`HttpEntity`, `@RequestBody` 를 사용하면 HTTP 메시지 컨버터가 HTTP 메시지 바디의 내용을 우리가 원하는 문자나 객체 등으로 변환해준다.

HTTP 메시지 컨버터는 문자 뿐만 아니라 JSON도 객체로 변환해주는데, 우리가 방금 V2에서 했던 작업을 대신 처리 해준다.

**@RequestBody는 생략 불가능**

스프링은 `@ModelAttribute` , `@RequestParam` 과 같은 해당 애노테이션을 생략시 다음과 같은 규칙을 적용한다.

* `String`, `int`, `Integer` 같은 단순 타입 = `@RequestParam`&#x20;
* 나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)

따라서 이 경우 `HelloData`에 `@RequestBody` 를 생략하면 `@ModelAttribute` 가 적용되어버린다.\
`HelloData data` -> `@ModelAttribute HelloData data`&#x20;

따라서 생략하면 HTTP 메시지 바디가 아니라 요청 파라미터를 처리하게 된다.

#### requestBodyJsonV4 - HttpEntity

```java
@ResponseBody
@PostMapping("/request-body-json-v4")
public String requestBodyJsonV4(HttpEntity<HelloData> data) throws IOException {

    HelloData helloData = data.getBody();
    log.info("helloData={}", helloData);
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());

    return "ok";
}
```

**@ResponseBody**&#x20;

응답의 경우에도 `@ResponseBody` 를 사용하면 해당 객체를 HTTP 메시지 바디에 직접 넣어줄 수 있다.\
물론 이 경우에도 `HttpEntity` 를 사용해도 된다.

* `@RequestBody` 요청
  * JSON 요청 -> HTTP 메시지 컨버터 -> 객체
* `@ResponseBody` 응답&#x20;
  * 객체 -> HTTP 메시지 컨버터 -> JSON 요청&#x20;

## 9. HTTP 응답 - 정적 리소스, 뷰 템플릿&#x20;

스프링(서버)에서 응답 데이터를 만드는 방법은 크게 3가지이다.

* 정적 리소스
  * 예) 웹 브라우저에 정적인 HTML, css, js를 제공할 때는, **정적 리소스**를 사용한다.
* 뷰 템플릿 사용
  * 예) 웹 브라우저에 동적인 HTML을 제공할 때는 뷰 템플릿을 사용한다.
* HTTP 메시지 사용
  * HTTP API를 제공하는 경우에는 HTML이 아니라 데이터를 전달해야 하므로, HTTP 메시지 바디에\
    JSON 같은 형식으로 데이터를 실어 보낸다.

### 정적 리소스&#x20;

스프링 부트는 클래스패스의 다음 디렉토리에 있는 정적 리소스를 제공한다.\
`/static` , `/public` , `/resources` ,`/META-INF/resources`

`src/main/resources` 는 리소스를 보관하는 곳이고, 또 클래스패스의 시작 경로이다.\
따라서 다음 디렉토리에 리소스를 넣어두면 스프링 부트가 정적 리소스로 서비스를 제공한다.

#### 정적 리소스 경로

`src/main/resources/static`

다음 경로에 파일이 들어있으면\
`src/main/resources/static/basic/hello-form.html`&#x20;

웹 브라우저에서 다음과 같이 실행하면 된다.\
`http://localhost:8080/basic/hello-form.html`&#x20;

정적 리소스는 해당 파일을 변경 없이 그대로 서비스하는 것이다.

### 뷰 템플릿&#x20;

뷰 템플릿을 거쳐서 HTML이 생성되고, 뷰가 응답을 만들어서 전달한다.\
일반적으로 HTML을 동적으로 생성하는 용도로 사용하지만, 다른 것들도 가능하다. 뷰 템플릿이 만들 수 있는 것이라면 뭐든지 가능하다.

#### **뷰 템플릿 경로**

`src/main/resources/templates`

#### ResponseViewController - 뷰 템플릿을 호출하는 컨트롤러

```java
package hello.springmvc.basic.response;

import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

@Controller
public class ResponseViewController {

    @RequestMapping("/response-view-v1")
    public ModelAndView responseViewV1() {
        ModelAndView mv = new ModelAndView("response/hello")
                .addObject("data", "hello!");

        return mv;
    }

    @RequestMapping("/response-view-v2")
    public String responseViewV2(Model model) {
        model.addAttribute("data", "hello!");
        return "response/hello";
    }

    @RequestMapping("/response/hello")
    public void responseViewV3(Model model) {
        model.addAttribute("data", "hello!");
    }
}
```

**String을 반환하는 경우 - View or HTTP 메시지**

`@ResponseBody` 가 없으면 `response/hello` 로 뷰 리졸버가 실행되어서 뷰를 찾고, 렌더링 한다.

`@ResponseBody` 가 있으면 뷰 리졸버를 실행하지 않고, HTTP 메시지 바디에 직접 `response/hello` 라는 문자가 입력된다.

여기서는 뷰의 논리 이름인 `response/hello` 를 반환하면 다음 경로의 뷰 템플릿이 렌더링 되는 것을 확인할 수 있다.

**Void 를 반환하는 경우**

* `@Controller` 를 사용하고, `HttpServletResponse`, `OutputStream(Writer)` 같은 HTTP 메시지 바디를 처리하는 파라미터가 없으면 요청 URL을 참고해서 논리 뷰 이름으로 사용
  * 요청 URL:`/response/hello`&#x20;
  * 실행: `templates/response/hello.html`
* **참고로 이 방식은 명시성이 너무 떨어지고 이렇게 딱 맞는 경우도 많이 없어서, 권장하지 않는다.**

**HTTP 메시지**

`@ResponseBody`, `HttpEntity` 를 사용하면, 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 응답\
데이터를 출력할 수 있다.

## 10. HTTP 응답 - HTTP API, 메시지 바디에 직접 입력&#x20;

#### ResponseBodyController

```java
package hello.springmvc.basic.response;

import hello.springmvc.basic.HelloData;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.bind.annotation.ResponseStatus;

import java.io.IOException;

@Slf4j
@Controller
public class ResponseBodyController {

    @GetMapping("/response-body-string-v1")
    public void responseBodyV1(HttpServletResponse response) throws IOException {
        response.getWriter().write("ok");
    }

    @GetMapping("/response-body-string-v2")
    public ResponseEntity<String> responseBodyV2() throws IOException {
        return new ResponseEntity<>("ok", HttpStatus.OK);
    }

    @ResponseBody
    @GetMapping("/response-body-string-v3")
    public String responseBodyV3() throws IOException {
        return "ok";
    }

    @GetMapping("/response-body-json-v1")
    public ResponseEntity<HelloData> responseBodyJsonV1() throws IOException {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return new ResponseEntity<>(helloData, HttpStatus.OK);
    }

    @ResponseStatus(HttpStatus.OK)
    @ResponseBody
    @GetMapping("/response-body-json-v2")
    public HelloData responseBodyJsonV2() throws IOException {
        HelloData helloData = new HelloData();
        helloData.setUsername("userA");
        helloData.setAge(20);

        return helloData;
    }
}
```

**responseBodyV1**

* 서블릿을 직접 다룰 때 처럼 `HttpServletResponse` 객체를 통해서 HTTP 메시지 바디에 직접 `ok` 응답 메시지를 전달한다.\
  `response.getWriter().write("ok")`&#x20;

**responseBodyV2**

* `ResponseEntity` 엔티티는 `HttpEntity` 를 상속 받았는데, HttpEntity는 HTTP 메시지의 헤더, 바디 정보를 가지고 있다. `ResponseEntity` 는 여기에 더해서 HTTP 응답 코드를 설정할 수 있다.
* `HttpStatus.CREATED`로 변경하면 201 응답이 나가는 것을 확인할 수 있다.

**responseBodyV3**

* `@ResponseBody` 를 사용하면 view를 사용하지 않고, HTTP 메시지 컨버터를 통해서 HTTP 메시지를 직접 입력할 수 있다. `ResponseEntity` 도 동일한 방식으로 동작한다.

**responseBodyJsonV1**

* `ResponseEntity` 를 반환한다. HTTP 메시지 컨버터를 통해서 JSON 형식으로 변환되어서 반환된다.

**responseBodyJsonV2**

* `ResponseEntity` 는 HTTP 응답 코드를 설정할 수 있는데, `@ResponseBody` 를 사용하면 이런 것을 설정하기 까다롭다.
* `@ResponseStatus(HttpStatus.OK)` 애노테이션을 사용하면 응답 코드도 설정할 수 있다.
* 물론 애노테이션이기 때문에 응답 코드를 동적으로 변경할 수는 없다. 프로그램 조건에 따라서 동적으로 변경하려면 `ResponseEntity` 를 사용하면 된다.

#### @RestContoller&#x20;

`@Controller` 대신에 `@RestController` 애노테이션을 사용하면, 해당 컨트롤러에 모두 `@ResponseBody` 가 적용되는 효과가 있다. 따라서 뷰 템플릿을 사용하는 것이 아니라, HTTP 메시지 바디에 직접 데이터를 입력한다. 이름 그대로 Rest API(HTTP API)를 만들 때 사용하는 컨트롤러이다.

## 11. HTTP 메시지 컨버터&#x20;

뷰 템플릿으로 HTML을 생성해서 응답하는 것이 아니라, HTTP API처럼 JSON 데이터를 HTTP 메시지 바디에서 직접 읽거나 쓰는 경우 HTTP 메시지 컨버터를 사용하면 편리하다.

#### @ResponseBody 사용 원리&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-10 21.00.34.png" alt=""><figcaption></figcaption></figure>

* `@ResponseBody` 를 사용
  * HTTP의 BODY에 문자 내용을 직접 반환
  * `viewResolver` 대신에 `HttpMessageConverter` 가 동작
  * 기본 문자처리: `StringHttpMessageConverter`
  * 기본 객체처리: `MappingJackson2HttpMessageConverter`&#x20;
  * `byte` 처리 등등 기타 여러 `HttpMessageConverter`가 기본으로 등록되어 있음

#### 스프링 MVC 는 다음의 경우에 HTTP 메시지 컨버터를 적용한다.&#x20;

* HTTP 요청: `@RequestBody`, `HttpEntity(RequestEntity)`&#x20;
* HTTP 응답: `@ResponseBody`, `HttpEntity(ResponseEntity)`

### HTTP 메시지 컨버터 인터페이스&#x20;

`org.springframework.http.converter.HttpMessageConverter`

```java
package org.springframework.http.converter;

public interface HttpMessageConverter<T> {

    boolean canRead(Class<?> clazz, @Nullable MediaType mediaType);
    boolean canWrite(Class<?> clazz, @Nullable MediaType mediaType);
    
    List<MediaType> getSupportedMediaTypes();
    
    T read(Class<? extends T> clazz, HttpInputMessage inputMessage)
        throws IOException, HttpMessageNotReadableException;
    void write(T t, @Nullable MediaType contentType, HttpOutputMessage outputMessage)
        throws IOException, HttpMessageNotWritableException;
}
```

HTTP 메시지 컨버터는 HTTP 요청, HTTP 응답 둘 다 사용된다.&#x20;

* `canRead()`, `canWrite()` : 메시지 컨버터가 해당 클래스, 미디어타입을 지원하는지 체크&#x20;
* `read()`, `write()` : 메시지 컨버터를 통해서 메시지를 읽고 쓰는 기능

### 스프링 부트 기본 메시지 컨버터&#x20;

```
0 = ByteArrayHttpMessageConverter
1 = StringHttpMessageConverter
2 = MappingJackson2HttpMessageConverter
...
```

스프링 부트는 다양한 메시지 컨버터를 제공하는데, 대상 클래스 타입과 미디어 타입 둘을 체크해서 사용여부를 결정한다. 만약 만족하지 않으면 다음 메시지 컨버터로 우선순위가 넘어간다.&#x20;

몇가지 주요한 메시지 컨버터를 알아보자.&#x20;

* `ByteArrayHttpMessageConverter` : `byte[]` 데이터를 처리한다.
  * 클래스 타입: `byte[]`, 미디어타입: `*/*`
  * 요청 예) `@RequestBody byte[] data`
  * 응답 예) `@ResponseBody return byte[]` 쓰기 미디어타입 `application/octet-stream`
* `StringHttpMessageConverter` : `String` 문자로 데이터를 처리한다.
  * 클래스 타입: `String` , 미디어타입: `*/*`
  * 요청 예) `@RequestBody String data`
  * 응답 예) `@ResponseBody return "ok"` 쓰기 미디어타입 `text/plain`
* `MappingJackson2HttpMessageConverter` : `application/json`
  * 클래스 타입: 객체 또는 `HashMap`, 미디어타입 `application/json` 관련
  * 요청 예) `@RequestBody HelloData data`
  * 응답 예) `@ResponseBody return helloData` 쓰기 미디어타입 `application/json` 관련

**StringHttpMessageConverter**

```
content-type: application/json

@RequestMapping
void hello(@RequestBody String data) {}
```

**MappingJackson2HttpMessageConverter**

```
content-type: application/json

@RequestMapping
void hello(@RequestBody HelloData data) {}
```

**?**

```
content-type: text/html

@RequestMapping
void hello(@RequestBody HelloData data) {}
```

#### HTTP 요청 데이터 읽기&#x20;

* HTTP 요청이 오고, 컨트롤러에서 `@RequestBody`, `HttpEntity` 파라미터를 사용한다.
* 메시지 컨버터가 메시지를 읽을 수 있는지 확인하기 위해 `canRead()` 를 호출한다.
  * 대상 클래스 타입을 지원하는가.
    * 예) `@RequestBody` 의 대상 클래스 (`byte[]`, `String`, `HelloData` )
  * HTTP 요청의 Content-Type 미디어 타입을 지원하는가.
    * 예) `text/plain`, `application/json`, `*/*`
* `canRead()` 조건을 만족하면 `read()` 를 호출해서 객체 생성하고, 반환한다.

#### HTTP 응답 데이터 생성&#x20;

* 컨트롤러에서 `@ResponseBody`, `HttpEntity` 로 값이 반환된다.
* 메시지 컨버터가 메시지를 쓸 수 있는지 확인하기 위해 `canWrite()` 를 호출한다.
  * 대상 클래스 타입을 지원하는가.
    * 예) return의 대상 클래스 (`byte[]`, `String`, `HelloData` )
  * HTTP 요청의 Accept 미디어 타입을 지원하는가. (더 정확히는 `@RequestMapping` 의 `produces` )
    * 예) `text/plain`, `application/json`, `*/*`
* `canWrite()` 조건을 만족하면 `write()` 를 호출해서 HTTP 응답 메시지 바디에 데이터를 생성한다.

## 12. 요청 매핑 헨들러 어댑터 구조&#x20;

그렇다면 HTTP 메시지 컨버터는 스프링 MVC 어디쯤에서 사용되는 것일까?&#x20;

아래 그림에서는 보이지 않는다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-11 09.56.39.png" alt=""><figcaption></figcaption></figure>

모든 비밀은 애노테이션 기반의 컨트롤러, 그러니까 `@RequestMapping` 을 처리하는 핸들러 어댑터인 `RequestMappingHandlerAdapter` (요청 매핑 헨들러 어뎁터)에 있다.

#### RequestMappingHandlerAdapter 동작 방식&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-11 09.57.56.png" alt=""><figcaption></figcaption></figure>

### ArgumentResolver

생각해보면, 애노테이션 기반의 컨트롤러는 매우 다양한 파라미터를 사용할 수 있었다. \
`HttpServletRequest`, `Model` 은 물론이고, `@RequestParam`, `@ModelAttribute` 같은 애노테이션 그리고`@RequestBody`, `HttpEntity` 같은 HTTP 메시지를 처리하는 부분까지 매우 큰 유연함을 보여주었다.

이렇게 파라미터를 유연하게 처리할 수 있는 이유가 바로 `ArgumentResolver` 덕분이다.

애노테이션 기반 컨트롤러를 처리하는 `RequestMappingHandlerAdapter` 는 바로 이 `ArgumentResolver` 를 호출해서 컨트롤러(핸들러)가 필요로 하는 다양한 파라미터의 값(객체)을 생성한다. 그리고 이렇게 파리미터의 값이 모두 준비되면 컨트롤러를 호출하면서 값을 넘겨준다.

스프링은 30개가 넘는 `ArgumentResolver` 를 기본으로 제공한다.\
어떤 종류들이 있는지 살짝 코드로 확인만 해보자.

정확히는 `HandlerMethodArgumentResolver` 인데 줄여서 `ArgumentResolver` 라고 부른다.

```java
public interface HandlerMethodArgumentResolver {

    boolean supportsParameter(MethodParameter parameter);
    
    @Nullable
    Object resolveArgument(MethodParameter parameter, 
                           @Nullable ModelAndViewContainer mavContainer,
                           NativeWebRequest webRequest, 
                           @Nullable WebDataBinderFactory binderFactory) 
    throws Exception;
}
```

#### 동작 방식

`ArgumentResolver` 의 `supportsParameter()` 를 호출해서 해당 파라미터를 지원하는지 체크하고, 지원하면`resolveArgument()` 를 호출해서 실제 객체를 생성한다. 그리고 이렇게 생성된 객체가 컨트롤러 호출시 넘어가는 것이다.

그리고 원한다면 여러분이 직접 이 인터페이스를 확장해서 원하는 `ArgumentResolver` 를 만들 수도 있다. 실제 확장하는 예제는 향후 로그인 처리에서 진행하겠다.

### ReturnValueHandler

`HandlerMethodReturnValueHandler` 를 줄여서 `ReturnValueHandler` 라 부른다.\
`ArgumentResolver` 와 비슷한데, 이것은 응답 값을 변환하고 처리한다.

컨트롤러에서 `String`으로 뷰 이름을 반환해도, 동작하는 이유가 바로 `ReturnValueHandler` 덕분이다.\
어떤 종류들이 있는지 살짝 코드로 확인만 해보자.

스프링은 10여개가 넘는 `ReturnValueHandler` 를 지원한다.\
예) `ModelAndView`, `@ResponseBody`, `HttpEntity`, `String`

### HTTP 메시지 컨버터&#x20;

#### HTTP 메시지 컨버터 위치&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-11 10.05.33.png" alt=""><figcaption></figcaption></figure>

HTTP 메시지 컨버터는 어디쯤 있을까?\
HTTP 메시지 컨버터를 사용하는 `@RequestBody` 도 컨트롤러가 필요로 하는 파라미터의 값에 사용된다.\
`@ResponseBody` 의 경우도 컨트롤러의 반환 값을 이용한다.

**요청의 경우** `@RequestBody`를 처리하는 `ArgumentResolver` 가 있고, `HttpEntity` 를 처리하는 `ArgumentResolver` 가 있다. 이 `ArgumentResolver`들이 HTTP 메시지 컨버터를 사용해서 필요한 객체를 생성하는 것이다. (어떤 종류가 있는지 코드로 살짝 확인해보자)

**응답의 경우** `@ResponseBody`와 `HttpEntity`를 처리하는 `ReturnValueHandler`가 있다. 그리고 여기에서 HTTP 메시지 컨버터를 호출해서 응답 결과를 만든다.
