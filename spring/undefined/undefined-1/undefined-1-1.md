# 싱글톤 컨테이너

## 1. 웹 애플리케이션과 싱글톤&#x20;

* 이전에 만든 스프링 없는 순수한 DI 컨테이너인 `AppConfig` 는 요청할 때마다 객체를 새로 생성한다.&#x20;
* **문제는 보통의 웹 애플리케이션은 여러 고객이 동시에 요청한다.**&#x20;
  * **고객 트래픽이 초당 100이 나오면, 초당 100개 객체가 생성/소멸되는 꼴이다.**&#x20;
* 이 문제를 해결하기 위해서 스프링 컨테이너는 해당 객체가 딱 1개만 생성되고 공유하도록 설계한 싱글톤 패턴을 이용한다.

```java
@Test
@DisplayName("스프링 없는 순수한 DI 컨테이너")
void pureContainer() {
    AppConfig appConfig = new AppConfig();
    MemberService memberService1 = appConfig.memberService();
    MemberService memberService2 = appConfig.memberService();

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

    Assertions.assertThat(memberService1).isNotSameAs(memberService2);
}
```

## 2. 싱글톤 패턴

* 클래스의 인스턴스를 딱 1개만 생성되도록 보장하는 디자인 패턴이다.
* `private` 생성자를 통해서 외부에서 `new` 키워드를 통해 객체를 생성하지 못하도록 막는다.

<pre class="language-java"><code class="lang-java">public class SingletonService {

      //1. static 영역에 객체를 딱 1개만 생성해둔다.
      private static final SingletonService instance = new SingletonService();
      
      //2. public으로 열어서 객체 인스턴스가 필요하면 이 static 메서드를 통해서만 조회하도록 허용한다.
      public static SingletonService getInstance() {
          return instance;
      }
      
<strong>      //3. 생성자를 private으로 선언해서 외부에서 new 키워드를 사용한 객체 생성을 못하게 막는다. 
</strong><strong>      private SingletonService() {
</strong>      }      
}
</code></pre>

### 싱글톤 패턴의 문제점

* 싱글톤 패턴을 구현하는 코드 자체가 많아진다.
* 의존관계상 클라이언트가 구체 클래스에 의존해서 DIP/OCP 원칙을 위반한다.&#x20;
  * 클라이언트 입장에서는 확장에 열려있어야 하고 변경에는 닫혀 있어야 한다.
* `private` 생성자로 자식 클래스를 만들기가 어렵다.
* **결론적으로 유연성이 떨어지기에 안티패턴으로 불린다.**

### **싱글톤 패턴의 주의점**&#x20;

* 이 방식을 사용하면 여러 클라이언트가  하나의 같은 객체 인스턴스를 공유하기 때문에, 싱글톤 객체는 상태를 유지하지 않도록(stateless : 불변) 설계해야 한다.&#x20;
  * 특정 클라이언트에 의존적인 필드가 있으면 안된다.&#x20;
  * 특정 클라이언트가 값을 변경할 수 있는 필드가 있으면 안된다.&#x20;
  * 가급적 읽기만 가능해야 한다. (read only)
  * 필드 대신 자바에서 공유되지 않는 지역변수, 파라미터, `ThreadLocal` 등을 사용해야 한다.&#x20;

## 3. 싱글톤 컨테이너

* 스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하며 객체 인스턴스를 싱글톤으로 관리한다.&#x20;
  * **싱글톤 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.**&#x20;
* **DIP/OCP, 테스트코드 작성, `private` 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.**&#x20;
  * **싱글톤 컨테이너는 안티패턴이 아니다!** \
    ~~(어떻게 구현했을까?)~~&#x20;

## 4. @configuration 과 싱글톤&#x20;

* `memberService` 와 `orderService` 빈을 만드는 코드를 보면 각각 `memberRepositor()` 를 호출해서 `new MemoeyMemberRepository()` 가 호출된다.&#x20;
* **결과적으로 다른 2개의 객체 인스턴스가 생성되면서 싱글톤이 깨지는 것처럼 보여지지만, 스프링 컨테이너는 이를 하나의 객체로 유지시킨다.**&#x20;

<pre class="language-java"><code class="lang-java"><strong>@Configuration
</strong>public class AppConfig {
    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository()); // 호출1
    }

    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }

    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(memberRepository(), discountPolicy()); // 호출2(중복?)
    }

    @Bean
    public DiscountPolicy discountPolicy() {
        return new RateDiscountPolicy();
    }
}
</code></pre>

* 직접 테스트 코드를 작성해서 확인해도 `memberRepository1`, `memberRepository2` 는 같은 객체로 확인된다.&#x20;

```java
@Test
void configurationTest() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
    OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
    MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

    MemberRepository memberRepository1 = memberService.getMemberRepository();
    MemberRepository memberRepository2 = orderService.getMemberRepository();

    System.out.println("memberRepository1 = " + memberRepository1);
    System.out.println("memberRepository2 = " + memberRepository2);
    System.out.println("memberRepository = " + memberRepository);

    Assertions.assertThat(memberService.getMemberRepository()).isSameAs(memberRepository);
    Assertions.assertThat(orderService.getMemberRepository()).isSameAs(memberRepository);
}
```

## 5. 싱글톤의 비밀 - 바이트코드 조작&#x20;

* 스프링 컨테이너는 싱글톤 레지스트리로, 스프링 빈이 싱글톤이 되도록 보장해야 한다.&#x20;
* **그러나 자바 코드로 조작하기는 어렵기 때문에, 클래스의 바이트 코드를 조작하는 라이브러리를 사용한다.**&#x20;
  * 순수한 클래스라면 class `hello.core.AppConfig` 가 출력되어야겠지만,&#x20;
  * `@Configuration` 을 적용한 `AppConfig` 는 CGLIB 라는 바이트 코드 조작 라이브러리를 사용해서 `AppConfig` 클래스를 상속받은 임의의 다른 클래스를 만들고, 다른 클래스를 스프링 빈으로 등록한다.&#x20;
  * `@Configuration` 을 적용하지 않는다면 싱글톤이 보장되지 않는다.&#x20;
* **해당 라이브러리는 자바 클래스가 싱글톤으로 보장 되도록 해준다.**&#x20;
  * `@Bean` 이 붙은 메서드마다 이미 스프링 빈이 존재하면, 존재하는 빈을 반환한다.
  * 스프링 빈이 없으면 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.&#x20;

```java
@Test
void configurationDeep() {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean);
    // 출력: bean = class hello.core.AppConfig$$EnhancerBySpringCGLIB$$bd479d70
}
```

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

