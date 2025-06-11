# 스프링 MVC - 기본 기능1

## 1. 요청 매핑&#x20;

#### MappingController

```java
package hello.springmvc.basic.requestmapping;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import org.springframework.web.bind.annotation.*;

@RestController
public class MappingController {
    
    private Logger log = LoggerFactory.getLogger(getClass());
    
    /**
    * 기본 요청
    * 둘다 허용 /hello-basic, /hello-basic/
    * HTTP 메서드 모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE
    */
    @RequestMapping("/hello-basic")
    public String helloBasic() {
        log.info("helloBasic");
        return "ok";
    }
}
```

#### 매핑 정보

* `@RestController`
  * `@Controller` 는 반환 값이 `String` 이면 뷰 이름으로 인식된다. 그래서 뷰를 찾고 뷰가 랜더링 된다.&#x20;
  *   `@RestController` 는 반환 값으로 뷰를 찾는 것이 아니라, **HTTP 메시지 바디에 바로 입력**한다. 따라서

      실행 결과로 ok 메세지를 받을 수 있다. `@ResponseBody` 와 관련이 있는데, 뒤에서 더 자세히 설명한다.
* `@RequestMapping("/hello-basic")`
  * `/hello-basic` URL 호출이 오면 이 메서드가 실행되도록 매핑한다.
  * 대부분의 속성을 `배열[]` 로 제공하므로 다중 설정이 가능하다. `{"/hello-basic", "/hello-go"}`

#### HTTP 메서드&#x20;

`@RequestMapping` 에 `method` 속성으로 HTTP 메서드를 지정하지 않으면 HTTP 메서드와 무관하게 호출된다.\
모두 허용 GET, HEAD, POST, PUT, PATCH, DELETE

#### HTTP 메서드 매핑&#x20;

```java
/**
* method 특정 HTTP 메서드 요청만 허용
* GET, HEAD, POST, PUT, PATCH, DELETE
*/
@RequestMapping(value = "/mapping-get-v1", method = RequestMethod.GET)
public String mappingGetV1() {
    log.info("mappingGetV1");
    return "ok";
}
```

#### HTTP 메서드 매핑 축약&#x20;

```java
/**
* 편리한 축약 애노테이션 (코드보기)
* @GetMapping
* @PostMapping
* @PutMapping
* @DeleteMapping
* @PatchMapping
*/
@GetMapping(value = "/mapping-get-v2")
public String mappingGetV2() {
    log.info("mapping-get-v2");
    return "ok";
}
```

#### PathVariable(경로 변수) 사용&#x20;

```java
/**
* PathVariable 사용
* 변수명이 같으면 생략 가능
* @PathVariable("userId") String userId -> @PathVariable String userId
*/
@GetMapping("/mapping/{userId}")
public String mappingPath(@PathVariable("userId") String data) {
    log.info("mappingPath userId={}", data);
    return "ok";
}
```

최근 HTTP API 는 다음과 같이 리소스 경로에 식별자를 넣는 스타일을 선호한다.&#x20;

* `/mapping/userA`&#x20;
* `/users/1`
* `@PathVariable` 의 이름과 파라미터 이름이 같으면 생략할 수 있다.

#### 특정 파라미터 조건 매핑&#x20;

```java
/**
* 파라미터로 추가 매핑
* params="mode",
* params="!mode"
* params="mode=debug"
* params="mode!=debug" (! = )
* params = {"mode=debug","data=good"}
*/
@GetMapping(value = "/mapping-param", params = "mode=debug")
public String mappingParam() {
    log.info("mappingParam");
    return "ok";
}
```

#### 특정 헤더 조건 매핑&#x20;

```java
/**
* 특정 헤더로 추가 매핑
* headers="mode",
* headers="!mode"
* headers="mode=debug"
* headers="mode!=debug" (! = )
*/
@GetMapping(value = "/mapping-header", headers = "mode=debug")
public String mappingHeader() {
    log.info("mappingHeader");
    return "ok";
}
```

#### 미디어 타입 조건 매핑 - HTTP 요청 Content-Type, consume

```java
/**
* Content-Type 헤더 기반 추가 매핑 Media Type
* consumes="application/json"
* consumes="!application/json"
* consumes="application/*"
* consumes="*\/*"
* MediaType.APPLICATION_JSON_VALUE
*/
@PostMapping(value = "/mapping-consume", consumes = "application/json")
public String mappingConsumes() {
    log.info("mappingConsumes");
    return "ok";
}
```

#### 미디어 타입 조건 매핑 - HTTP 요청 Accept, produce

```java
/**
* Accept 헤더 기반 Media Type
* produces = "text/html"
* produces = "!text/html"
* produces = "text/*"
* produces = "*\/*"
*/
@PostMapping(value = "/mapping-produce", produces = "text/html")
public String mappingProduces() {
    log.info("mappingProduces");
    return "ok";
}
```

## 2. 요청 매핑 - API 예시&#x20;

회원 관리를 HTTP API 로 만든다 생각하고 매핑을 어떻게 하는지 알아보자.&#x20;

#### 회원 관리 API&#x20;

* 회원 목록 조회: GET `/users`
* 회원 등록: POST `/users`
* 회원 조회: GET `/users/{userId}`
* 회원 수정: PATCH `/users/{userId}`
* 회원 삭제: DELETE `/users/{userId}`&#x20;

#### MappingClassController

다른 예제와 HTTP API 경로가 겹칠 수 있기에 앞에 `/mapping` 을 붙였다.&#x20;

```java
package hello.springmvc.basic.requestmapping;

import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/mapping/users")
public class MappingClassController {

    @GetMapping
    public String user() {
        return "get user";
    }

    @PostMapping
    public String addUser() {
        return "post user";
    }

    @GetMapping("/{userId}")
    public String findUser(@PathVariable String userId) {
        return "get userId=" + userId;
    }

    @PatchMapping("/{userId}")
    public String updateUser(@PathVariable String userId) {
        return "update userId=" + userId;
    }

    @DeleteMapping("/{userId}")
    public String deleteUser(@PathVariable String userId) {
        return "delete userId=" + userId;
    }
}
```

매핑 방법을 이해했으니, 이제부터 HTTP 요청을 보내는 데이터들을 스프링 MVC 로 어떻게 조회하는지 알아보자.&#x20;

## 3. HTTP 요청 - 기본, 헤더 조회&#x20;

애노테이션 기반의 스프링 컨트롤러는 다양한 파라미터를 지원한다.&#x20;

#### RequestHeaderController&#x20;

```java
package hello.springmvc.basic.request;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpMethod;
import org.springframework.util.MultiValueMap;
import org.springframework.web.bind.annotation.CookieValue;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.Locale;

@Slf4j    // 로그 객체 생성 코드를 컴파일 타임에 생성한다. 
@RestController
public class RequestHeaderController {

    @RequestMapping("/headers")
    public String headers(HttpServletRequest request,
                          HttpServletResponse response,
                          HttpMethod httpMethod,
                          Locale locale,
                          @RequestHeader MultiValueMap<String, String> headerMap,
                          @RequestHeader("host") String host,
                          @CookieValue(value = "myCookie", required = false) String cookie
    ){
        log.info("request={}", request);
        log.info("response={}", response);
        log.info("httpMethod={}", httpMethod);
        log.info("locale={}", locale);
        log.info("headerMap={}", headerMap);
        log.info("header host={}", host);
        log.info("myCookie={}", cookie);

        return "ok";
    }
}
```

* `HttpServletRequest`
* `HttpServletResponse`
* `HttpMethod` : HTTP 메서드를 조회한다. `org.springframework.http.HttpMethod`
* `Locale` : Locale 정보를 조회한다.
* `@RequestHeader MultiValueMap<String, String> headerMap`&#x20;
  * 모든 HTTP 헤더를 MultiValueMap 형식으로 조회한다.
* `@RequestHeader("host") String host`&#x20;
  * 특정 HTTP 헤더를 조회한다.
  * 속성
    * 필수 값 여부: `required`
    * 기본 값 속성: `defaultValue`&#x20;
* `@CookieValue(value = "myCookie", required = false) String cookie`&#x20;
  * 특정 쿠키를 조회한다.
  * 속성
    * 필수 값 여부: `required`
    * 기본 값: `defaultValue`&#x20;

#### MultiValueMap

* MAP과 유사한데, 하나의 키에 여러 값을 받을 수 있다.
* HTTP header, HTTP 쿼리 파라미터와 같이 하나의 키에 여러 값을 받을 때 사용한다.
  * `keyA=value1&keyA=value2`

```java
MultiValueMap<String, String> map = new LinkedMultiValueMap();
map.add("keyA", "value1");
map.add("keyA", "value2");

//[value1,value2]
List<String> values = map.get("keyA");
```

## 4. HTTP 요청 파라미터 - 쿼리 파라미터, HTML Form&#x20;

### HTTP 요청 데이터 조회 - 개요&#x20;

#### 클라이언트에서 서버로 요청 데이터를 전달할 때는 주로 다음 3가지 방법을 사용한다.&#x20;

* **GET - 쿼리 파라미터**
  * `/url?username=hello&age=20`
  * 메시지 바디 없이, URL의 쿼리 파라미터에 데이터를 포함해서 전달
  * 예) 검색, 필터, 페이징등에서 많이 사용하는 방식
* **POST - HTML Form**
  * `content-type: application/x-www-form-urlencoded`
  * 메시지 바디에 쿼리 파리미터 형식으로 전달 `username=hello&age=20`
  * 예) 회원 가입, 상품 주문, HTML Form 사용
* **HTTP message body**에 데이터를 직접 담아서 요청
  * HTTP API에서 주로 사용, JSON, XML, TEXT
  * 데이터 형식은 주로 JSON 사용
  * POST, PUT, PATCH

### 요청 파라미터 - 쿼리 파라미터, HTML Form&#x20;

`HttpServletRequest` 의 `request.getParameter()` 를 사용하면 다음 두가지 요청 파라미터를 조회할 수 있다.

**GET 쿼리 파리미터 전송 방식이든, POST HTML Form 전송 방식이든 둘다 형식이 같으므로 구분없이 조회할 수 있다.**

이것을 간단히 **요청 파라미터(request parameter) 조회**라 한다.

#### GET, 쿼리 파라미터 전송&#x20;

예시 \
`http://localhost:8080/request-param?username=hello&age=20`&#x20;

#### POST, HTML Form 전송&#x20;

예시

```java
POST /request-param ...
content-type: application/x-www-form-urlencoded

username=hello&age=20
```

#### RequestParamController&#x20;

```java
package hello.springmvc.basic.request;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import java.io.IOException;
import java.util.Map;

@Slf4j
@Controller
public class RequestParamController {

    @RequestMapping("/request-param-v1")
    public void requestParamV1(HttpServletRequest request, HttpServletResponse response) throws IOException {
        String username = request.getParameter("username");
        int age = Integer.parseInt(request.getParameter("age"));
        log.info("username={}, age={}", username, age);

        response.getWriter().write("ok");
    }
}
```

**request.getParamter()**

여기서는 단순히 `HttpServletRequest`가 제공하는 방식으로 요청 파라미터를 조회했다.

## 5. HTTP 요청 파라미터 - @RequestParam

스프링이 제공하는 `@RequestParam` 을 사용하면 요청 파라미터를 매우 편리하게 사용할 수 있다.&#x20;

#### requestParamV2&#x20;

```java
@ResponseBody
@RequestMapping("/request-param-v2")
public String requestParamV2(
        @RequestParam("username") String memberName,
        @RequestParam("age") int memberAge) {
    log.info("username={}, age={}", memberName, memberAge);

    return "ok";
}
```

* `@RequestParam` : 파라미터 이름으로 바인딩
* `@ResponseBody` : View 조회를 무시하고, HTTP message body에 직접 해당 내용 입력

#### @RequestParam의 `name(value)` 속성이 파라미터 이름으로 사용

* `@RequestParam("username") String memberName`
* \=> `request.getParameter("username")`

#### **requestParamV3**

```java
@ResponseBody
@RequestMapping("/request-param-v3")
public String requestParamV3(
        @RequestParam String username,
        @RequestParam int age) {
    log.info("username={}, age={}", username, age);

    return "ok";
}
```

* HTTP 파라미터 이름이 변수 이름과 같으면 `@RequestParam(name="xx")` 생략 가능

#### requestParamV4

```java
@ResponseBody
@RequestMapping("/request-param-v4")
public String requestParamV4(String username, int age) {
    log.info("username={}, age={}", username, age);

    return "ok";
}
```

* `String`, `int`, `Integer` 등의 단순 타입이면 `@RequestParam` 도 생략 가능

> #### 참고
>
> 이렇게 애노테이션을 완전히 생략해도 되는데, 너무 없는 것도 약간 과하다는 주관적 생각이 있다.
>
> `@RequestParam` 이 있으면 명확하게 요청 파리미터에서 데이터를 읽는 다는 것을 알 수 있다.

> #### 주의 - 스프링 부터 3.2 파라미터 이름 인식 문제&#x20;
>
> 다음 예외가 발생하면 해당 내용을 참고하자.
>
> ```
> java.lang.IllegalArgumentException: Name for argument of type [java.lang.String]
> not specified, and parameter name information not found in class file either.
> ```
>
> 주로 다음 두 애노테이션에서 문제가 발생한다.\
> `@RequestParam`, `@PathVariable`
>
> #### @RequestParam 관련&#x20;
>
> ```java
> //애노테이션에 username이라는 이름이 명확하게 있다. 문제 없이 작동한다.
> @RequestMapping("/request")
> public String request(@RequestParam("username") String username) {
>     ...
> }
>
> //애노테이션에 이름이 없다. -parameters 옵션 필요
> @RequestMapping("/request")
> public String request(@RequestParam String username) {
>     ...
> }
>
> //애노테이션도 없고 이름도 없다. -parameters 옵션 필요
> @RequestMapping("/request")
> public String request(String username) {
>     ...
> }
> ```
>
> #### **@PathVariable 관련(이후에 학습한다)**
>
> ```java
> //애노테이션에 userId라는 이름이 명확하게 있다. 문제 없이 작동한다.
> public String mappingPath(@PathVariable("userId") String userId) {
>     ...
> }
>
> //애노테이션에 이름이 없다. -parameters 옵션 필요
> public String mappingPath(@PathVariable String userId) {
>     ...
> }
> ```
>
> #### 해결 방안1 (권장)&#x20;
>
> 애노테이션에 이름을 생략하지 않고 다음과 같이 이름을 항상 적어준다. **이 방법을 권장한다.**
>
> * `@RequestParam("username") String username`
> * `@PathVariable("userId") String userId`
>
> #### 해결 방안 2,3&#x20;
>
> ~~pass ...~~&#x20;
>
> #### 문제 원인&#x20;
>
> 참고로 이 문제는 Build, Execution, Deployment -> Build Tools -> Gradle에서 Build and run using를 IntelliJ IDEA로 선택한 경우에만 발생한다. Gradle로 선택한 경우에는 Gradle이 컴파일 시점에 해당 옵션을 자동으로 적용해준다.
>
> 자바를 컴파일할 때 매개변수 이름을 읽을 수 있도록 남겨두어야 사용할 수 있다. 컴파일 시점에 `-parameters` 옵션을 사용하면 매개변수 이름을 사용할 수 있게 남겨둔다.
>
> 스프링 부트 3.2 전까지는 바이트코드를 파싱해서 매개변수 이름을 추론하려고 시도했다. 하지만 스프링 부트 3.2 부터는 이런 시도를 하지 않는다.

#### 파라미터 필수 여부 - requestParamRequired

```java
@ResponseBody
@RequestMapping("/request-param-required")
public String requestParamRequired(
        @RequestParam(required = true) String username,        // 요청값이 없다면 400 에러 
        @RequestParam(required = false) Integer age) {
    log.info("username={}, age={}", username, age);

    return "ok";
}
```

주의 - 파라미터 이름만 사용 \
`/request-param-required?username=`&#x20;

* 파라미터 이름만 있고 값이 없는 경우 빈문자로 통과

주의 - 기본형(primitive) 에 null 입력 \
`/request-param`

* `@RequestParam(required = false) int age`
* `null` 을 `int` 에 입력하는 것은 불가능(500 예외 발생)
* 따라서 `null` 을 받을 수 있는 `Integer`로 변경하거나, 또는 다음에 나오는 `defaultValue` 사용

#### 기본 값 적용 - requestParamDefault

```java
@ResponseBody
@RequestMapping("/request-param-default")
public String requestParamDefault(
        @RequestParam(required = true, defaultValue = "guest") String username,
        @RequestParam(required = false, defaultValue = "-1") int age) {
    log.info("username={}, age={}", username, age);

    return "ok";
}
```

* 파라미터에 값이 없는 경우 `defaultValue` 를 사용하면 기본 값을 적용할 수 있다.
* 이미 기본 값이 있기 때문에 `required` 는 의미가 없다.
* `defaultValue` 는 빈 문자의 경우에도 설정한 기본 값이 적용된다.
  * `/request-param-default?username=`

#### 파라미터를 Map 으로 조회하기 - requestParamMap&#x20;

```java
@ResponseBody
@RequestMapping("/request-param-map")
public String requestParamDefault(@RequestParam Map<String, Object> paramMap){
    log.info("username={}, age={}", paramMap.get("username"), paramMap.get("age"));

    return "ok";
}
```

## 6. HTTP 요청 파라미터 - @ModelAttribute&#x20;

실제 개발을 하다 요청 파라미터를 받아서 필요한 객체를 만들고 그 객체에 값을 넣어주어야 한다. 보통 다음과 같이 코드를 작성할 것이다.&#x20;

```java
@RequestParam String username;
@RequestParam int age;
HelloData data = new HelloData();

data.setUsername(username);
data.setAge(age);
```

스프링은 이 과정을 완전히 자동화해주는 `@ModelAttribute` 기능을 제공한다.&#x20;

먼저 요청 파라미터를 바인딩 받을 객체를 만들자.&#x20;

#### HelloData

```java
package hello.springmvc.basic;

import lombok.Data;

@Data // @Getter, @Setter, @ToString, @EqualsAndHashCode, @RequiredArgsConstructor 
public class HelloData {
    private String username;
    private int age;
}
```

#### **@ModelAttribute 적용 - modelAttributeV1**

```java
@ResponseBody
@RequestMapping("/model-attribute-v1")
public String modelAttributeV1(@ModelAttribute HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

마치 마법처럼 `HelloData` 객체가 생성되고, 요청 파라미터의 값도 모두 들어가 있다.

스프링MVC는 `@ModelAttribute` 가 있으면 다음을 실행한다.

* `HelloData` 객체를 생성한다.
* 요청 파라미터의 이름으로 `HelloData` 객체의 프로퍼티를 찾는다. 그리고 해당 프로퍼티의 **setter를 호출해서 파라미터의 값을 입력(바인딩) 한다.**&#x20;
* 예) 파라미터 이름이 `username` 이면 `setUsername()` 메서드를 찾아서 호출하면서 값을 입력한다.

#### 바인딩 오류&#x20;

`age=abc` 처럼 숫자가 들어가야 할 곳에 문자를 넣으면 `BindException` 이 발생한다. 이런 바인딩 오류를 처리하는 방법은 검증 부분에서 다룬다.

#### @ModelAttribute 생략 - modelAttributeV2

```java
@ResponseBody
@RequestMapping("/model-attribute-v2")
public String modelAttributeV2(HelloData helloData) {
    log.info("username={}, age={}", helloData.getUsername(), helloData.getAge());
    return "ok";
}
```

`@ModelAttribute` 는 생략할 수 있다.\
그런데 `@RequestParam` 도 생략할 수 있으니 혼란이 발생할 수 있다.

#### 스프링은 파라미터 애노테이션 생략시 다음과 같은 규칙을 적용한다.

* `String`, `int`, `Integer` 같은 단순 타입 =`@RequestParam`
* 나머지 = `@ModelAttribute` (argument resolver 로 지정해둔 타입 외)
