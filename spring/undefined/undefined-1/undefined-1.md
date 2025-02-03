# 예제를 통한 스프링 핵심 원리 이해

## 1. 주문 도메인 설계&#x20;

* 주문 도메인 협력, 역할, 책임&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 주문 도메인 전체&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 주문 클래스 다이어그램&#x20;

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 주문 객체 다이어그램&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2. 주문 도메인 개발&#x20;

```java
public class OrderServiceImpl implements OrderService {
//  private final DiscountPolicy discountPolicy = new FixDiscountPolicy();
    private final DiscountPolicy discountPolicy = new RateDiscountPolicy();
}
```

* 위처럼 코드를 작성하면, OCP/DIP 원칙에 어긋난다.&#x20;
  * 기능을 확장하려면 클라이언트 코드에 영향을 주기 때문에, OCP 를 위반하고&#x20;
  * 주문 서비스 클라이언트 `OrderServiceImpl` 은 `DiscountPolicy` 인터페이스를 의존하는 것과 동시에 `FixDiscountPolicy` 혹은 `RateDiscountPolicy` 에도 의존으로 하고 있기 때문에, DIP 를 위반한다.&#x20;
* 현재 구현한 모델의 실제 모습은 다음과 같다.&#x20;

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 즉, 위 코드를 인터페이스에만 의존하도록 설계를 변경해야 한다.&#x20;
  * 그러나 아래 경우에는 구현체가 없기 때문에, `nullPointException` 이 발생한다.&#x20;
  * 따라서 누군가가 클라이언트인 `OrderServiceImpl` 에 `DiscountPolicy` 의 구현 객체를 대신 생성하고 주입해주어야 한다.&#x20;

```java
public class OrderServiceImpl implements OrderService {
    private final DiscountPolicy discountPolicy;
}
```

### AppConfig 등장&#x20;

* `AppConfig` 는 애플리케이션의 전체 동작 방식을 구성하기 위한 객체를 생성하고, 연결하는 책임을 가지는 설정 클래스이다.&#x20;
* 이는 클라이언트인 `OrderServiceImpl` 에 구현하지 않은 이유는, SRP 원칙을 지키기 위해서이다.&#x20;
  * `OrderServiceImpl` 은 `DiscountPolicy` 의 구현 객체를 가지고 주문 로직을 수행해야 한다.&#x20;
  * 여기에 추가적으로 각 인터페이스에 어떤 구현 객체가 들어와야 하는지 정하는 다른 책임이 더해진다면, 클라이언트는 점점 복잡해진다.&#x20;
  * 따라서, 각각의 책임을 확실히 분리하기 위해 `AppConfig` 를 별도로 만드는 것이다.&#x20;
* 아래 코드에서는 `OrderServiceImpl` 은 구현 클래스를 의존하지 않는다.&#x20;
  * `OrderServiceImpl` 입장에서 생성자를 통해 어떤 구현 객체가 주입될지 알 수 없다.&#x20;
  * 이는 오직 외부(`AppConfig`) 에서 결정한다.&#x20;

```java
public class AppConfig {
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

```java
public class OrderServiceImpl implements OrderService {
    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    public OrderServiceImpl(MemberRepository memberRepository, DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        ths.discountPolicy = discountPolicy;
    }
}
```

* 별도의 설정 클래스(`AppConfig`) 를 사용하므로, OCP/DIP 원칙을 지키며 기존에 하고자 했던 설계를 했다.&#x20;
* 비즈니스 로직상 `DiscountPolicy` 인터페이스의 구현 객체로 다른 클래스가 추가되어도, 구성 영역인 `AppConfig` 에서 수정하면 클라이언트 영역의 어떠한 코드 변경 없이 확장할 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 3. 의존 관계 주입&#x20;

### IoC(Inversion of Control) : 제어의 역전&#x20;

* 기존 프로그램은 클라이언트 구현 객체가 스스로 필요한 서버 구현 객체를 생성하고, 연결하고, 실행했다.&#x20;
  * 한마디로 구현 객체가 프로그램의 제어 흐름을 스스로 조종했다.&#x20;
* 반면에 `AppConfig` 가 등장한 이후 구현 객체는 자신의 로직을 실행하는 역할만 담당하고, 프로그램의 제어 흐름은 이제 `AppConfig` 가 가져간다.&#x20;
* 이렇 듯 프로그램의 제어 흐름을 직접 제어하는 것이 아니라 외부에서 관리하는 것을 제어의 역전(IoC) 라고 한다.&#x20;

### DI(Dependency Injection) : 의존 관계 주입&#x20;

* DI 개념을 정리하면 아래와 같다.
  * 런타임에 외부에서 실제 구현 객체를 생성하고 클라이언트에 전달해 의존관계를 만들어내는 것을 DI 라고 한다.&#x20;
  * DI 를 사용하면 클라이언트 코드를 변경하지 않고, 클라이언트가 호출하는 대상의 타입 인스턴스를 변경할 수 있다.
  * DI 를 사용하면, 런타임 의존관계를 쉽게 변경할 수 있다.
* `OrderServiceImpl` 은 `DiscountPolicy` 인터페이스에만 의존한다.&#x20;
  * 실제 어떤 구현 객체가 사용될지는 모른다.&#x20;
  * 이러한 의존관계는 **정적인 클래스 의존 관계**와 **실행 시점에 결정되는 동적인 객체 의존 관계**로 분리해서 생각해야 한다.&#x20;
* 정적인 클래스 의존 관계&#x20;
  * 클래스가 사용하는 `import` 코드만 보고 의존 관계를 쉽게 판단할 수 있다.&#x20;
  * 정적인 의존 관계는 애플리케이션을 실행하지 않아도 분석 할 수 있다.&#x20;
    * `OrderServiceImpl` 은 `MemeberRepository` 와 `DiscountPolicy` 에 의존함을 알 수 있는 것처럼 말이다.&#x20;

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 동적인 객체 의존 관계&#x20;
  * 런타임에 실제 생성된 객체 인스턴스의 참조가 연결된 의존 관계이다.&#x20;

<figure><img src="../../../.gitbook/assets/image (7) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### DI 컨테이너&#x20;

* 위 코드에서 `AppConfig` 처럼 객체를 생성하고, 관리하면서 의존 관계를 연결해주는 것을 IoC 컨테이너, DI 컨테이너라고 한다.&#x20;

## 4. 스프링으로 전환&#x20;

* `ApplicationContext` 를 스프링 컨테이너라고 한다.&#x20;
* 기존에는 개발자가 `AppConfig` 를 생성해 직접 객체를 생성하고, DI 를 했지만 이제부터는 스프링 컨테이너를 통해서 사용한다. (이 얼마나 간결한가?!)
* 스프링 컨테이너는 @Configuration 이 붙은 `AppConfig` 를 설정 정보로 사용한다.&#x20;
  * 여기서 `@Bean` 이라고 적힌 메서드를 모두 호출해서 반환된 객체를 스프링 컨테이너에 등록한다.&#x20;
  * 이렇게 스프링 컨테이너에 등록된 객체를 스프링 빈이라고 부른다.&#x20;
* 이전에는 개발자가 필요한 객체를 `AppConfig` 를 사용해서 직접 조회했지만, 이제부터는 스프링 컨테이너를 통해서 필요한 스프링 빈을 찾아야 한다.&#x20;
* 기존에는 개발자가 직접 자바 코드로 모든 것을 했다면, 이제부터는 스프링 컨테이너에 객체를 스프링 빈으로 등록하고, 스프링 컨테이너에서 스프링 빈을 찾아서 사용하도록 변경되었다.&#x20;

```java
@Configuration
public class AppConfig {
    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    
    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
```

```java
public class OrderApp {

    public static void main(String[] args) {
//        AppConfig appConfig = new AppConfig();
//        OrderService orderService = appConfig.orderService();

        ApplicationContext applicationContext = new AnnotationConfigApplicationContext(AppConfig.class);
        
        OrderService orderService = applicationContext.getBean("orderService", OrderService.class);

        Long memberId = 1L;
        Order order = orderService.createOrder(memberId, "itemA", 20000);

        System.out.println("order = " + order);
    }
}
```
