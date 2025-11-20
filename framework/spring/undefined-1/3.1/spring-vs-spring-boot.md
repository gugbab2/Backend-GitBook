# Spring vs Spring Boot

## 1. 스프링은 무엇인가? 스프링은 어떤 중요한 문제를 다루고 있는가?

* 스프링 프레임워크는 자바 생태계에서 가장 대중적인 개발 프레임워크로,
* 의존성 주입(DI), 제어의 역전(IoC) 은 스프링에서 가장 중요한 특징이다.\
  \-> 이들을 통해서 객체의 결합도를 낮추는 방식으로 어플리케이션을 개발할 수 있다.
* 이런 개발방식으로 개발한 응용 프로그램은 단위테스트가 용이하기 때문에, 보다 높은 퀄리티의 프로그램을 개발할 수 있다.

### 1-1. DI 없는 예제

```java
@RestController
public class MyController {
    private MyService service = new MyService();

    @RequestMapping("/welcome")
    public String welcome() {
        return service.retrieveWelcomeMessage();
    }
}
```

* 위 예제를 보면 해당 클래스가 MyService 라는 클래스에 의존하여 결합도가 높은 상태인 것을 확인할 수 있다.

### 1-2. DI 있는 예제

```java
@Component
public class MyService {
    public String retrieveWelcomeMessage(){
       return "Welcome to InnovationM";
    }
}

@RestController
public class MyController {

    @Autowired
    private MyService service;

    @RequestMapping("/welcome")
    public String welcome() {
        return service.retrieveWelcomeMessage();
    }
}
```

* 위 예제는 해당 클래스가 MyService 클래스에 대한 책임을 없애, 결합도가 낮아진 상태를 볼 수 있다.
* 간단히 말해, 스프링 프레임워크는 웹 어플리케이션을 개발하는데 결합도를 낮추는 방향의 개발방법을 제공한다고 말할 수 있다.

## 2. 스프링이 이리도 좋은데, 스프링 부트는 왜 필요한가?

* 기존의 스프링은 DI와 IoC 를 통해서 클래스간의 결합도를 낮추는 방식을 제공했지만, 그 설정에 있어(.xml) 수많은 시간 투자를 감내해야 했다..
* 최소한의 기능으로 아래 Spring MVC 를 사용하여 기본 프로젝트를 설정하는 다음 코드를 보자.

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix">
        <value>/WEB-INF/views/</value>
    </property>
    <property name="suffix">
        <value>.jsp</value>
    </property>
</bean>

<mvc:resources mapping="/webjars/**" location="/webjars/"/>

<servlet>
    <servlet-name>dispatcher</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/my-servlet.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>dispatcher</servlet-name>
    <url-pattern>/</url-pattern>
</servlet-mapping>
```

## 3. 스프링부트는 이러한 문제를 어떻게 해결했는가?

### 3-1. 번거로운 설정 : 의존성 자동설정(AutoConfiguration)

* 스프링부트의 AutoConfiguration 은 Starter 의 의존성의 문제가 해결된 라이브러리를 기반으로 애플리케이션 구성을 자동으로 설정한다.\
  \-> 스프링 부트가 애플리케이션 classpath 를 검사하여 starter에 포함된 라이브러리와 설정 파일을 찾고, 이를 사용하여 필요한 bean 들을 자동으로 생성하고 구성하는 것을 말한다.
* 스프링부트의 최상단에는 @SpringBootApplication 이라는 어노테이션이 있다. 해당 어노테이션은 다음의 어노테이션을 합쳐놓은 것이다.
  * @SpringbootConfiguration
  * @ComponentScan\
    \-> 자신을 root 로 하위 패키지들을 싹 훑어서 @Component 라는 어노테이션을 붙인 클래스들을 찾아서 Bean 으로 등록한다.
  * @EnableAutoConfiguration\
    \-> 해당 에너테이션을 통해서 AutoConfiguration 이 활성화된다.
* 예를 들어 spring-boot-starter-web 을 사용하는 경우 스프링 부트 자동 구성은 다음의 설정을 자동으로 처리한다.
  1. Tomcat 또는 Undertow와 같은 내장 서버의 설정: 스프링 부트는 "spring-boot-starter-web" Starter의 의존성을 기반으로 내장 서버(Tomcat 또는 Undertow)를 자동으로 구성합니다. 따라서 개발자는 별도의 서버 설정이나 배포 파일을 작성할 필요 없이 내장 서버를 사용할 수 있습니다.
  2. DispatcherServlet의 등록: 웹 애플리케이션을 위해 스프링 MVC를 사용하는 경우, "spring-boot-starter-web" Starter는 DispatcherServlet을 자동으로 등록합니다. 이를 통해 HTTP 요청을 처리하는 컨트롤러와 뷰를 매핑할 수 있습니다.
  3. WebMvcConfigurer의 설정: "spring-boot-starter-web" Starter는 WebMvcConfigurer를 구현한 Bean들을 검색하여 자동으로 설정합니다. 이를 통해 개발자는 추가적인 설정 없이도 웹 애플리케이션의 인터셉터, 리소스 핸들러, 포매터 등을 등록할 수 있습니다.
  4. Spring Boot의 기본 설정: 스프링 부트의 자동 구성은 기본적으로 여러 가지 기능과 설정들을 제공합니다. 예를 들어, 자동으로 설정되는 로깅 설정, 자동으로 설정되는 스프링 시큐리티 설정 등이 있습니다.

### 3-2. 의존성 관리의 어려움 : Starter

* 스프링 프레임워크는 개발자가 필요한 의존성들을 하나씩 추가하고 버전을 관리해야 했다.\
  \-> 이는 의존성 충돌이나 버전 관리의 어려움을 초래할 수 있다..
* 스프링부트는 Starter 라는 개념을 도입해서 필요한 의존성을 한번에 가져오고 버전 관리를 자동화한다.\
  \-> 이는 개발자에게 의존성 관리에 대한 부담을 지우고 빠르게 개발에 집중할 수 있다.
* 예를 들어 spring-boot-starter-jpa 를 의존성 추가했을 때 아래의 일을 해준다.
  * spring-app, spring-jdbc 등의 의존성을 걸어준다.
  * classpath 를 뒤져서 어떤 Database 를 사용하는지 파악하고, 자동으로 entityManager 를 구성해준다.
  * 해당 모듈들 설정에 필요한 properties 설정을 제공한다.

### 3-3. 내장 서버의 부재

* 기존 스프링 프레임워크에서는 웹 개발시 외부 서버(톰캣)에 애플리케이션을 배포해야했다.\
  \-> 이는 배포와 관리의 번거로움을 초래한다.
* 스프링 부트는 내장 서버를 기본적으로 제공하여 애플리케이션을 실행하고 배포할 수 있도록 도와준다.
