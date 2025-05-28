---
description: 프
---

# 빈 스코프

## 1. 빈 스코프란?

* 스코프는 빈이 존재할 수 있는 범위를 의미하며, 스프링은 다음과 같은 다양한 스코프를 지원한다.
  * 싱글톤 : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
  * 프로토타입 : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
  * 웹 관련 스코프
    * request : 웹 요청이 들어오고 나갈때까지 유지되는 스코프
    * session
    * application

## 2. 프로토타입 스코프

* 항상 같은 인스턴스를 제공하는 싱글톤 스코프 빈과 다르게, 프로토타입 스코프 빈은 항상 새로운 인스턴스를 생성해서 반환한다.
* 그 흐름은 다음과 같다.
  * 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.\
    -> 요청하기 전까지는 빈을 생성하지 않는다.
  * 스프링 컨테이너는 이 시점에 **프로토타입 빈을 생성하고, 필요한 의존관계를 주입**한다.\
    -> **초기화까지 처리한다.**
  * 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.
  * 이후 스프링 컨테이너는 프로토타입을 관리하지 않고 프로토타입 빈을 요청시 새로운 인스턴스를 반환한다.\
    -> 때문에, @PreDestroy 같은 종료 메서드가 호출되지 않는다.

### 2-1. 싱글톤 빈과의 차이점

* **싱글톤 빈은 스프링 컨테이너 생성 시점에 초기화 메서드가 실행 되지만,**\
  **프로토타입 스코프 빈은 스프링 컨테이너에서 빈을 조회할 때 생성되고, 초기화 메서드도 실행된다.**
* **싱글톤 빈은 스프링 컨테이너가 관리하기 때문에, 스프링 컨테이너가 종료될 때 빈의 종료 메서드가 실행되지만,**\
  **프로토타입 빈은 스프링 컨테이너가 생성과 의존관계 주입 그리고 초기화까지만 관여하고 더는 관리하지 않는다.**\
  &#xNAN;**-> 때문에, @PreDestroy 같은 종료 메서드가 호출되지 않는다.**\
  &#xNAN;**-> 프로토타입 빈은 프로토타입 빈을 조회한 클라이언트가 직접 관리해야 한다.**

### 2-2. 싱글톤 빈과 함께 사용했을 때 문제점

* 두 빈의 라이프사이클이 다르다는 점을 위에서 확인할 수 있다.\
  때문에, 요청때마다 새로운 빈을 생성하기 원했던 프로토타입 빈이 싱글톤 빈에 종속되어지는 상황에서는 싱글톤 빈의 라이프사이클에 따라서 하나의 인스턴스가 공유되게 된다.\
  -> **싱글톤 빈 생성시 이미 주입이 끝난 상태이다.**
* 이때 Provider 를 사용해 문제를 해결할 수 있다.\
  -> **위와 같은 상황에서 스프링의 어플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트가 어려워진다.**\
  &#xNAN;**-> 지금 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 기능! DL 정도의 기능을 제공하는 무언가가 필요하다.**

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;
  
public int logic() {
      PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
      prototypeBean.addCount();
      int count = prototypeBean.getCount();
      return count;
}
```

> **Dependency Lookup(DL)**
>
> * 의존관계를 외부에서 주입받는 것이 아닌, 직접 필요한 의존관계를 찾는 것을 의존관계 탐색(DL)이라고 한다.

### 2-3. 프로토타입 빈을 언제 사용하는가?

* 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 된다!
* 그런데 실무에서는 싱글톤 빈으로 대부분의 문제를 해결할 수 있기에 프로토타입 빈을 사용하는 경우는 드물다..

## 3. 웹 스코프

* 웹 스코프는 웹 환경에서만 동작하고,
* 프로토타입 스코프와는 다르게 스프링이 해당 스코프의 종료시점까지 관리한다.\
  -> 따라서 종료 메서드가 호출된다.

### 3-1. 웹 스코프 종류

* request : HTTP 요청 하나가 들어오고 나갈 때까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
* session : HTTP Session 과 동일한 생명주기를 갖는 스코프
* application : 서블릿 컨텍스트(ServletContext)와 동일한 생명주기를 가지는 스코프
* websocket : 웹 소켓과 동일한 생명주기를 갖는 스코프

### 3-2. request 스코프

<figure><img src="../../.gitbook/assets/image (38).png" alt=""><figcaption></figcaption></figure>

> 스프링 부트는 웹 라이브러리가 없으면 우리가 지금까지 학습한 `AnnotationConfigApplicationContext`을 기반으로 애플리케이션을 구동한다. 웹 라이브러리가 추가되면 웹과 관련된 추가 설정과 환경들이 필요하므로 `AnnotationConfigServletWebServerApplicationContext` 를 기반으로 애플리케이션을 구동한다.

```java
@Component
@Scope(value = "request")
public class MyLogger {

  private String uuid;
  private String requestURL;
  
  public void setRequestURL(String requestURL) {
   this.requestURL = requestURL;
  }
  public void log(String message) {
   System.out.println("[" + uuid + "]" + "[" + requestURL + "] " + message);
  }
 
  @PostConstruct
  public void init() {
   uuid = UUID.randomUUID().toString();
   System.out.println("[" + uuid + "] request scope bean create:" + this);
  }
  @PreDestroy
  public void close() {
   System.out.println("[" + uuid + "] request scope bean close:" + this);
  }
}
```

```java
@Controller
@RequiredArgsConstructor
public class LogDemoController {
  private final LogDemoService logDemoService;
  private final MyLogger myLogger;
 
  @RequestMapping("log-demo")
  @ResponseBody
  public String logDemo(HttpServletRequest request) {
   String requestURL = request.getRequestURL().toString();
   myLogger.setRequestURL(requestURL);
   myLogger.log("controller test");
   logDemoService.logic("testId");
   return "OK";
  }
}
```

```java
@Service
@RequiredArgsConstructor
public class LogDemoService {
  private final ObjectProvider<MyLogger> myLoggerProvider;
  public void logic(String id) {
    MyLogger myLogger = myLoggerProvider.getObject();
    myLogger.log("service id = " + id);
  }
}

```

* **위의 코드를 실행했을 때, 다음과 같이 우리가 기대했던 값이 나오지 않는 것을 확인할 수 있다...**\
  &#xNAN;**-> 그 이유는 스프링 애플리케이션을 실행하는 시점에 싱글톤 빈은 생성해서 주입이 가능하지만, request 스코프 빈은 아직 생성되지 않았기 때문이다..(이 빈은 실제 고객의 요청이 와야 생성할 수 있다)**

<pre><code><strong>Error creating bean with name 'myLogger': Scope 'request' is not active for the 
</strong>current thread; consider defining a scoped proxy for this bean if you intend to 
refer to it from a singleton;
</code></pre>

* **때문에, ObjectProvider 를 사용했을 때! 우리가 원하는 아래의 결과를 얻을 수 있다.**

```
[d06b992f...] request scope bean create
[d06b992f...][http://localhost:8080/log-demo] controller test
[d06b992f...][http://localhost:8080/log-demo] service id = testId
[d06b992f...] request scope bean close
```
