# POJO(Plain Old Java Object)

스프링 백엔드 개발자를 위한 관점으로 POJO(Plain Old Java Object) 를 정리해보자.&#x20;

### POJO 란 무엇인가?  <a href="#pojo-20-eb-9e-80-20-eb-ac-b4-ec-97-87-ec-9d-b8-ea-b0-80-3f-c2-a0-1" id="pojo-20-eb-9e-80-20-eb-ac-b4-ec-97-87-ec-9d-b8-ea-b0-80-3f-c2-a0-1"></a>

* POJO는 "Plain Old Java Object"의 약자로, 특정 프레임워크나 기술에 종속되지 않은 순수한 자바 객체를 의미한다. 스프링 개발에서 POJO는 매우 중요한 개념이다. 스프링 프레임워크 자체가 POJO 중심의 개발을 강력하게 지원하기 때문이다.
* 자바의 기본에 충실: 특정 규약(예: 특정 클래스 상속, 특정 인터페이스 구현 강제) 없이, 자바 언어 명세에 따라 작성된 객체이다.
* 독립성: 객체가 특정 프레임워크 API나 환경에 직접적으로 의존하지 않아, 객체 자체의 생명주기나 동작 방식이 프레임워크에 의해 크게 좌우되지 않는다.
* POJO는 객체지향의 핵심 원칙, 특히 객체 간의 명확한 책임 분담을 프레임워크의 간섭 없이 지켜나갈 수 있도록 돕는 중요한 접근 방식 또는 스타일이라고 할 수 있다.&#x20;

### POJO 의 등장 배경  <a href="#pojo-20-ec-9d-98-20-eb-93-b1-ec-9e-a5-20-eb-b0-b0-ea-b2-bd-c2-a0-1-1" id="pojo-20-ec-9d-98-20-eb-93-b1-ec-9e-a5-20-eb-b0-b0-ea-b2-bd-c2-a0-1-1"></a>

POJO라는 용어는 2000년대 초반에 등장했다. 당시 엔터프라이즈 자바 개발 환경은 EJB(Enterprise JavaBeans) 같이 무겁고 침투적인(invasive) 프레임워크가 주를 이루었다.

* 과거 프레임워크의 문제점:
  * 높은 결합도: 개발자는 비즈니스 로직을 구현하기 위해 프레임워크에 특화된 클래스(예: javax.ejb.SessionBean)를 상속받거나, 특정 인터페이스(예: javax.ejb.EntityBean)를 구현해야 했다. 이로 인해 비즈니스 로직이 담긴 객체가 프레임워크 기술에 깊숙이 결합되었다.
  * 테스트의 어려움: 객체가 프레임워크 환경에 강하게 의존했기 때문에, EJB 컨테이너와 같은 특정 환경 없이는 단위 테스트를 수행하기 매우 어려웠다. 이는 개발 생산성을 저해하고 코드 품질 관리를 어렵게 만들었다.
  * 과도한 복잡성: 프레임워크를 사용하기 위해 많은 설정과 규약을 따라야 했고, 이는 개발의 복잡성을 증가시켰다.
* POJO로의 전환: 이러한 문제들에 대한 반작용으로, "단순한 자바 객체로도 충분히 엔터프라이즈 애플리케이션을 개발할 수 있어야 한다"는 생각이 확산되었다. 즉, 개발자가 비즈니스 로직에만 집중하고, 객체가 특정 기술의 복잡성으로부터 자유로워져야 한다는 요구가 POJO라는 개념을 탄생시켰다.

### 왜 스프링 개발에서 POJO 가 중요한가?  <a href="#ec-99-9c-20-ec-8a-a4-ed-94-84-eb-a7-81-20-ea-b0-9c-eb-b0-9c-ec-97-90-ec-84-9c-20pojo-20-ea-b0-80-20" id="ec-99-9c-20-ec-8a-a4-ed-94-84-eb-a7-81-20-ea-b0-9c-eb-b0-9c-ec-97-90-ec-84-9c-20pojo-20-ea-b0-80-20"></a>

스프링의 핵심 철학은 POJO의 등장 배경과 맞닿아 있다. 스프링은 "비침투적(non-invasive)" 프로그래밍 모델을 제공하여, 개발자가 비즈니스 로직에 집중할 수 있도록 돕는다. POJO는 이러한 철학을 가능하게 하는 핵심 요소이다.

* 낮은 결합도(Decoupling): 비즈니스 로직을 담은 POJO는 스프링 프레임워크 코드로부터 분리된다. 이를 통해 시스템의 다른 부분에 영향을 주지 않고 특정 부분을 수정하거나 확장하기 용이해진다.
* 향상된 테스트 용이성: POJO는 스프링 컨테이너나 복잡한 설정 없이도 일반적인 JUnit 같은 도구를 사용하여 쉽게 단위 테스트를 할 수 있다. 이는 개발 속도와 코드 품질을 크게 향상시킨다.
* 코드의 단순성 및 가독성: POJO는 필요한 비즈니스 로직과 상태만을 가지므로 코드가 단순하고 이해하기 쉽다. 개발자는 프레임워크의 복잡한 내부 구조보다는 실제 구현하려는 기능에 더 집중할 수 있다.
* 유연성 및 재사용성: 특정 기술이나 환경에 묶여있지 않아, 필요에 따라 다른 환경이나 프레임워크로 이전하거나 핵심 로직을 재사용하기 유리하다.

### 스프링에서 POJO 는 어떻게 활용되는가?  <a href="#ec-8a-a4-ed-94-84-eb-a7-81-ec-97-90-ec-84-9c-20pojo-20-eb-8a-94-20-ec-96-b4-eb-96-bb-ea-b2-8c-20-ed" id="ec-8a-a4-ed-94-84-eb-a7-81-ec-97-90-ec-84-9c-20pojo-20-eb-8a-94-20-ec-96-b4-eb-96-bb-ea-b2-8c-20-ed"></a>

스프링은 POJO를 사용하여 애플리케이션의 주요 구성 요소를 만든다.

* 빈(Beans)으로서의 POJO: 스프링 컨테이너는 POJO로 작성된 클래스들을 빈(Bean)으로 관리한다. 이 빈들은 애플리케이션의 핵심 로직을 수행하는 서비스 객체, 데이터 접근 객체(DAO), 도메인 객체 등이 될 수 있다.
* 의존성 주입 (Dependency Injection - DI): 스프링은 DI를 통해 POJO 객체들이 필요로 하는 다른 객체(의존성)들을 외부에서 주입해준다. POJO 자체는 의존성을 직접 생성하거나 검색할 필요가 없어 코드의 결합도가 낮아지고 유연성이 증가한다.

```java
// OrderService는 ProductRepository에 의존하지만, 직접 생성하지 않는다 (스프링이 주입).
public class OrderService { // POJO
    private final ProductRepository productRepository; // POJO (or Interface)

    public OrderService(ProductRepository productRepository) { // 생성자를 통한 DI
        this.productRepository = productRepository;
    }
    // 비즈니스 로직
}
```

* 관점 지향 프로그래밍 (Aspect-Oriented Programming - AOP): 트랜잭션 관리, 보안, 로깅과 같은 \*\*횡단 관심사(cross-cutting concerns)\*\*를 POJO 코드의 변경 없이 적용할 수 있게 한다. AOP를 통해 POJO는 순수하게 비즈니스 로직에만 집중할 수 있다.
* 애노테이션과 POJO:
  * 스프링에서 사용하는 @Component, @Service, @Repository, @Autowired, @Transactional 등의 애노테이션은 POJO에 메타데이터를 추가하여 스프링 컨테이너가 해당 객체를 어떻게 처리해야 할지 알려준다.
  * 이러한 애노테이션들은 객체에게 특정 클래스 상속이나 인터페이스 구현을 강제하지 않으므로, POJO의 본질을 크게 해치지 않는다. 오히려 POJO가 스프링의 강력한 기능을 손쉽게 활용할 수 있도록 돕는 역할을 한다. 애노테이션이 붙은 클래스도 여전히 핵심 로직은 순수 자바로 유지되며 테스트 가능하다.

### 스프링 백엔드 개발자를 위한 POJO 실천 가이드 <a href="#ec-8a-a4-ed-94-84-eb-a7-81-20-eb-b0-b1-ec-97-94-eb-93-9c-20-ea-b0-9c-eb-b0-9c-ec-9e-90-eb-a5-bc-20-e" id="ec-8a-a4-ed-94-84-eb-a7-81-20-eb-b0-b1-ec-97-94-eb-93-9c-20-ea-b0-9c-eb-b0-9c-ec-9e-90-eb-a5-bc-20-e"></a>

1. 핵심 비즈니스 로직은 POJO로 작성한다: 서비스 계층, 도메인 모델 등은 특정 프레임워크에 대한 의존 없이 순수 자바 객체로 구현하는 것을 목표로 한다.
2. 인터페이스를 활용한 설계: 구체 클래스보다는 인터페이스에 의존하게 만들어 유연성을 높이고 POJO 원칙을 지키는 데 도움이 된다.
3. DI 적극 활용: 객체 생성과 의존성 관리는 스프링 컨테이너에 맡기고, POJO는 필요한 의존성을 주입받아 사용하도록 한다.
4. 단위 테스트 생활화: POJO로 작성된 코드는 테스트하기 쉬우므로, 견고한 애플리케이션을 위해 단위 테스트를 꾸준히 작성한다.
