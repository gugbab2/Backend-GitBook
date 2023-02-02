# Spring Web MVC

## [Spring](https://docs.spring.io/spring-framework/docs/current/reference/html/overview.html#overview) 의 핵심

* Spring Framework는 모듈로 나뉩니다. 애플리케이션은 필요한 모듈을 선택할 수 있습니다. \
  **중심에는 구성 모델 및 의존성주입(Dependency Injection) 메커니즘을 포함하는 코어 컨테이너의 모듈이 있습니다.**\
  ****-> 생성자를 통한 DI 로 외부의 객체 내부 정보를 노출시키지 않아서, 의미 있는 캡슐화를 한 객체를 만들 수 있다.
* 그 외에도 Spring Framework는 메시징, 트랜잭션 데이터 및 지속성, 웹을 비롯한 다양한 애플리케이션 아키텍처에 대한 기본 지원을 제공 \
  \-> **웹 개발 시 필요한 수많은 기능 제공**
* ### 디자인 철학
  * 모든 수준에서 선택권을 제공합니다. \
    **Spring을 사용하면 가능한 한 늦게 디자인 결정을 연기할 수 있습니다.** \
    ****예를 들어 코드를 변경하지 않고 구성을 통해 지속성 공급자를 전환할 수 있습니다.\
    \->  EX) 현재 사용하고 있는 Mysql 을 요구사항에 의해서 MongoDB 로 변경하더라도 내부 코드 수정 없이 변경이 가능하다.(다형성!)\
    \-> **요구사항 수용에 최적!**
  * 다양한 관점을 수용합니다. \
    Spring은 유연성을 수용하며 작업 수행 방법에 대해 독선적이지 않습니다. 다양한 관점에서 다양한 애플리케이션 요구 사항을 지원합니다.
  * 강력한 이전 버전과의 호환성을 유지합니다. \
    Spring의 진화는 버전 간에 주요 변경 사항을 거의 강제하지 않도록 신중하게 관리되었습니다. Spring은 Spring에 의존하는 애플리케이션 및 라이브러리의 유지 관리를 용이하게 하기 위해 엄선된 범위의 JDK 버전 및 타사 라이브러리를 지원합니다.
  * API 디자인에 관심을 가져라. \
    Spring 팀은 직관적이고 여러 버전과 여러 해에 걸쳐 유지되는 API를 만들기 위해 많은 생각과 시간을 투자합니다.
  * 코드 품질에 대한 높은 기준을 설정하십시오. \
    Spring Framework는 의미 있고 최신이며 정확한 javadoc(문서화?)을 강조합니다. 패키지 간의 순환 종속성이 없는 깨끗한 코드 구조를 주장할 수 있는 극소수의 프로젝트 중 하나입니다.

## Spring Boot

## Spring initializer

## Web Server와 Web Application Server(WAS)

* ### Tomcat

## Model-View-Controller(MVC) 아키텍처 패턴

## 관심사의 분리(Seperation of Concern)

## Spring MVC

## Java Annotation

## Spring Annotation

* ### @RestController
  * #### @Controller
  * #### @ResponseBody
* ### @GetMapping
  * #### @RequestMapping
