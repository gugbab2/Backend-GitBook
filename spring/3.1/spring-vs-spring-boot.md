# Spring vs Spring Boot

## 1. 스프링은 무엇인가? 스프링은 어떤 중요한 문제를 다루고 있는가?

* 스프링 프레임워크는 자바 생태계에서 가장 대중적인 개발 프레임워크로,
* 의존성 주입(DI), 제어의 역전(IoC) 은 스프링에서 가장 중요한 특징이다. \
  \-> 이들을 통해서 객체의 결합도를 낮추는 방식으로 어플리케이션을 개발할 수 있다.
* 이런 개발방식으로 개발한 응용 프로그램은 단위테스트가 용이하기 때문에, 보다 높은 퀄리티의 프로그램을 개발할 수 있다.&#x20;

### 1-1. DI 없는 예제&#x20;

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

* 위 예제를 보면 해당 클래스가 MyService 라는 클래스에 의존하여 결합도가 높은 상태인 것을 확인할 수 있다.&#x20;

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

* 위 예제는 해당 클래스가 MyService 클래스에 대한 책임을 없애, 결합도가 낮아진 상태를 볼 수 있다.&#x20;
* 간단히 말해, 스프링 프레임워크는 웹 어플리케이션을 개발하는데 결합도를 낮추는 방향의 개발방법을 제공한다고 말할 수 있다.&#x20;

## 2. 스프링이 이리도 좋은데, 스프링 부트는 왜 필요한가?

* 기존의 스프링은 DI와 IoC 를 통해서 클래스간의 결합도를 낮추는 방식을 제공했지만, 그 설정에 있어(.xml) 수많은 시간 투자를 감내해야 했다..
* 최소한의  기능으로 아래  Spring MVC 를 사용하여 기본 프로젝트를 설정하는 다음 코드를 보자.

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

* 스프링 부트에서는 AutoConfiguration(자동 구성)이라는 기능을 제공합니다. 이를 통해 과거 수많은 빈설정, 의존성주입 등의 문제를 해결할 수 있다.&#x20;
* 예를 들어, 스프링 부트에서는 데이터베이스 연결 설정을 자동으로 처리할 수 있습니다. 개발자는 데이터베이스 드라이버를 추가한 후, 데이터베이스 URL, 사용자 이름, 암호 등을 구성 파일(application.properties 또는 application.yml)에 작성하면 됩니다. 스프링 부트는 이 정보를 기반으로 데이터베이스 연결을 자동으로 설정합니다.
* AutoConfiguration은 `@EnableAutoConfiguration` 애너테이션을 통해 활성화됩니다. 이 애너테이션은 주로 메인 애플리케이션 클래스에 추가됩니다. 스프링 부트는 자동 구성 클래스들을 스캔하고, 필요한 빈들을 자동으로 구성하여 애플리케이션에 추가합니다.

### 3-2. 의존성 관리의 어려움 : Starter

* 스프링 프레임워크는 개발자가 필요한 의존성들을 하나씩 추가하고 버전을 관리해야 했다. \
  \-> 이는 의존성 충돌이나 버전 관리의 어려움을 초래할 수 있다..
* 스프링부트는 Starter 라는 개념을 도입해서 필요한 의존성을 한번에 가져오고 버전 관리를 자동화한다. \
  \-> 이는 개발자에게 의존성 관리에 대한 부담을 지우고 빠르게 개발에 집중할 수 있다.&#x20;

### 3-3. 내장 서버의 부재

* 기존 스프링 프레임워크에서는 웹 개발시 외부 서버(톰캣)에 애플리케이션을 배포해야했다.\
  \-> 이는 배포와 관리의 번거로움을 초래한다.
* 스프링 부트는 내장 서버를 기본적으로 제공하여 애플리케이션을 실행하고 배포할 수 있도록 도와준다.
