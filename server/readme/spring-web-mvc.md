# Spring Web MVC

## [Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview) 의 핵심

* Spring Framework는 모듈로 나뉩니다. 애플리케이션은 필요한 모듈을 선택할 수 있습니다. \
  **중심에는 구성 모델 및 의존성주입(Dependency Injection) 메커니즘을 포함하는 코어 컨테이너의 모듈이 있습니다.**\
  ****-> **생성자를 통한 DI 로 외부의 객체 내부 정보를 노출시키지 않아서, 의미 있는 캡슐화를 한 객체를 만들 수 있다**.
* 그 외에도 Spring Framework는 메시징, 트랜잭션 데이터 및 지속성, 웹을 비롯한 다양한 애플리케이션 아키텍처에 대한 기본 지원을 제공 \
  \-> **웹 개발 시 필요한 수많은 기능 제공**
* ### 디자인 철학
  * 모든 수준에서 선택권을 제공합니다. \
    **Spring을 사용하면 가능한 한 늦게 디자인 결정을 연기할 수 있습니다.** \
    ****예를 들어 코드를 변경하지 않고 구성을 통해 지속성 공급자를 전환할 수 있습니다.\
    \->  EX) 현재 사용하고 있는 Mysql 을 요구사항에 의해서 MongoDB 로 변경하더라도 내부 코드 수정 없이 변경이 가능하다.(다형성!)\
    \-> **요구사항 수용에 최적!**
  * 다양한 관점을 수용합니다. \
    ~~Spring은 유연성을 수용하며 작업 수행 방법에 대해 독선적이지 않습니다. 다양한 관점에서 다양한 애플리케이션 요구 사항을 지원합니다.~~
  * 강력한 이전 버전과의 호환성을 유지합니다. \
    Spring의 진화는 버전 간에 주요 변경 사항을 거의 강제하지 않도록 신중하게 관리되었습니다. Spring은 Spring에 의존하는 애플리케이션 및 라이브러리의 유지 관리를 용이하게 하기 위해 엄선된 범위의 JDK 버전 및 타사 라이브러리를 지원합니다.
  * API 디자인에 관심을 가져라. \
    ~~Spring 팀은 직관적이고 여러 버전과 여러 해에 걸쳐 유지되는 API를 만들기 위해 많은 생각과 시간을 투자합니다.~~
  * 코드 품질에 대한 높은 기준을 설정하십시오. \
    ~~Spring Framework는 의미 있고 최신이며 정확한 javadoc(문서화?)을 강조합니다. 패키지 간의 순환 종속성이 없는 깨끗한 코드 구조를 주장할 수 있는 극소수의 프로젝트 중 하나입니다.~~

## Spring Boot

* 스프링부트는 스프링을 사용하기 위한 다양한 설정들을 미리 해주어 사용자 입장에서 편리할게 사용할 수 있도록 한 프로젝트이다.

## Spring initializer

* [https://start.spring.io/](https://start.spring.io/)

## Web Server와 Web Application Server(WAS)

*   ### Tomcat



    * 용어 정리\
      \[톰캣] : 톰캣은 WAS 로, WAS는 웹서버와 웹 컨테이너의 결합으로 다양한 기능을 컨테이너에 구현하여 다양한 역할을 수행할 수 있는 서버\
      \[웹 서버] : 웹 상에서 정적인 자료를 처리\
      \[웹 컨테이너] : 웹 컨테이너는 서블릿의 생명주기를 관리하고, 특정 서블릿을 맵핑하며 URL 요청이 올바른 접근 권한을 갖도록 보장한다.\
      \[서블릿] :  클라이언트의 요청을 처리하고, 그 결과를 반환하는 Servlet(HttpServlet) 클래스의 구현 규칙을 지킨 자바 웹 프로그래밍 기술\
      &#x20;\
      **종합하자면!** **클라이언트의 요청이 있을 때 내부의 프로그램(서블릿!)을 통해 결과를 만들어내고 이것을 다시 클라이언트에 전달해주는 역할을 하는 것이 웹 컨테이너**&#x20;
    * 톰캣 6버전 이상부터는 조금씩 웹서버의 기능이 추가되어서 현재는 톰캣만 설치해도 어느 정도의 웹서버 역할이 가능.\
      \-> 스프링부트를 실행했을 때, 내부적으로 임베디드 톰캣이 실행되는 것을 볼 수 있다.
    * 아직도 많은 JSP를 사용하는 웹 프로그래머들은 아파치 + 톰캣을 병행해서 사용. 그 이유는 정적 데이터를 처리할 때 아파치의 성능이 더 좋음.

    <figure><img src="../../.gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

## Model-View-Controller(MVC) 아키텍처 패턴

* Model, View, Controller 의 약자로 하나의 애플리케이션을 각각의 관심사에 따라서 분리하여 로직을 짜는 것을 의미한다.
*   View → 표현(웹페이지)

    Controller → 입력(URL 요청 처리)

    Model → 그 외의 모든 것 (사실상 도메인 모델)

## 관심사의 분리(Seperation of Concern)

* MVC 패턴이 추구하는 것으로 UI 와 비지니스 로직을 분리했다.

## Spring MVC

Spring Web MVC는 Map과 유사한 Model 인터페이스를 제공.

1. [https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/ui/package-summary.html](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/ui/package-summary.html)
2. 애플리케이션 전체가 아니라, 웹과 관련된 부분(UI Layer) 에서만 MVC 개념을 활용할 수 있도록 도움. 관심사의 분리를 더 심화할 기회 제공.

## Java Annotation

* 주석처럼 프로그래밍 언어에 영향을 미치지 않으며 유용한 정보를 제공(설정 정보)..\
  \-> Annotation 이전에 설정들은 대부분 .xml 형태로 했다.
* @Test, @Override, @FunctionalInterface, @Deprecated, @SuppressWarnings 등등..
* 아래와 같이 인터페이스를 만들 수 있다!

## Spring Annotation

* ### @RestController
  * #### @Controller : 컨트롤러의 역할을 수행한다 .
  *   #### @ResponseBody : 해당 어노테이션을 붙여주지 않는다면 스프링 프로젝트 내 뷰 템플릿과 매핑이된다.

      하지만 해다 어노테이션은 말 그대로 ResponseBody 에 값을 넣어 응답을 해준다.
* ### @GetMapping
  * #### @RequestMapping : 요청과 매핑이 되도록 연결해준다.
