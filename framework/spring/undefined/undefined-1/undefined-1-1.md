# 싱글톤 컨테이너

## 1. 웹 애플리케이션과 싱글톤&#x20;

이전에 만든 스프링 없는 순수한 DI 컨테이너인 `AppConfig` 는 요청할 때마다 객체를 새로 생성한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-28 08.24.28.png" alt=""><figcaption></figcaption></figure>

**문제는 보통의 웹 애플리케이션은 여러 고객이 동시에 요청한다.**&#x20;

* **고객 트래픽이 초당 100이 나오면, 초당 100개 객체가 생성/소멸되는 꼴이다.**&#x20;

이 문제를 해결하기 위해서 스프링 컨테이너는 해당 객체가 딱 1개만 생성되고 공유하도록 설계한 싱글톤 패턴을 이용한다.

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

#### 싱글톤 패턴의 문제점

* 싱글톤 패턴을 구현하는 코드 자체가 많아진다.
* 의존관계상 클라이언트가 구체 클래스에 의존한다.&#x20;
  * 추상화에 의존하지 않는다면 DIP/OCP 원칙을 위반한다.&#x20;
* 테스트하기 어렵다.
  * 전역 상태를 가지기에 각 테스트마다 독립성이 회손된다.&#x20;
* 내부 속성을 변경하거나, 초기화하기 어렵다.
* `private` 생성자로 자식 클래스를 만들기가 어렵다.
* **결론적으로 유연성이 떨어지기에 안티패턴으로 불린다.**

## 3. 싱글톤 컨테이너

스프링 컨테이너는 싱글톤 패턴의 문제점을 해결하며 객체 인스턴스를 싱글톤으로 관리한다.&#x20;

#### 싱글톤 컨테이너

* **스프링 컨테이너는 싱글톤 패턴을 적용하지 않아도, 객체 인스턴스를 싱글톤으로 관리한다.**&#x20;
* 스프링 컨테이너는 싱글톤 컨테이너의 역할을 한다. 이렇게 싱글톤 객체를 생성하고 관리하는 기능을 싱글톤 레지스트리라고 한다.&#x20;
* 스프링 컨테이너의 이런 기능 덕분에 싱글톤 패턴의 모든 단점은 해결하면서 객체를 싱글톤으로 유지할 수 있다.&#x20;
  * 싱글톤 패턴을 위한 지저분한 코드가 들어가지 않아도 된다.&#x20;
  * DIP/OCP, 테스트코드 작성, `private` 생성자로부터 자유롭게 싱글톤을 사용할 수 있다.&#x20;

#### 스프링 컨테이너를 사용하는 테스트 코드

```java
@Test
@DisplayName("스프링 컨테이너와 싱글톤")
void singletonContainer() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    MemberService memberService1 = ac.getBean("memberService", MemberService.class);
    MemberService memberService2 = ac.getBean("memberService", MemberService.class);

    System.out.println("memberService1 = " + memberService1);
    System.out.println("memberService2 = " + memberService2);

    assertThat(memberService1).isSameAs(memberService2);
}
```

#### 싱글톤 컨테이너 적용 후&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-28 08.31.26.png" alt=""><figcaption></figcaption></figure>

스프링 컨테이너 덕분에 고객의 요청이 올 때 마다 객체를 생성하는 것이 아니라, 이미 만들어진 객체를 공유해서 효율적으로 재사용할 수 있다.&#x20;

## 4. 싱글톤 방식의 주의점&#x20;

* 싱글톤 패턴이든,  스프링 같은 싱글톤 컨테이너를 사용하든, 객체 인스턴스를 하나만 생성해서 공유하는 싱글톤 방식은 여러 클라이언트가 하나의 같은 객체 인스턴스를 공유하기 때문에, 싱글톤 객체는 상태를 유지(stateful)하게 설계하면 안된다.&#x20;
* **무상태(stateless) 로 설계해야 한다.**&#x20;
  * 특정 클라이언트에 의존적인 필드가 있으면 안된다.&#x20;
  * 특정 클라이언트가 값을변경할 수 있는 필드가 있으면 안된다.&#x20;
  * 가급적 읽기만 가능해야 한다.&#x20;
  * 필드 대신에 자바에서 공유되지 않는, 지역변수, 파라미터, `ThreadLocal` 등을 사용해야 한다.&#x20;
* **스프링 빈의 필드에 공유 값을 설정하면 정말 큰 장애가 발생할 수 있다!**

#### 상태를 유지할 경우 발생하는 문제점 예시&#x20;

```java
package hello.core.singleton;

public class StatefulService {

    private int price;

    public void order(String name, int price) {
        System.out.println("name = " + name + " price = " + price);
        this.price = price;
    }

    public int getPrice() {
        return price;
    }
}
```

#### 상태를 유지할 경우 발생하는 문제점 예시&#x20;

```java
package hello.core.singleton;

import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;

import static org.assertj.core.api.Assertions.*;

class StatefulServiceTest {

    @Test
    void statefulServiceSingleton() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);
        StatefulService statefulService1 = ac.getBean(StatefulService.class);
        StatefulService statefulService2 = ac.getBean(StatefulService.class);

        // ThreadA : A사용자 10000원 주문 
        statefulService1.order("userA", 10000);
        // ThreadB : B사용자 20000원 주문 
        statefulService2.order("userB", 20000);
        
        // ThreadA : A사용자 주문 금액 조회 
        int price = statefulService1.getPrice();
        System.out.println("price = " + price);

        assertThat(price).isEqualTo(20000);
    }

    static class TestConfig {

        @Bean
        public StatefulService statefulService() {
            return new StatefulService();
        }
    }
}
```

* 사용자A 주문금액은 10000이 나와야 하는데 20000원이라는 결과가 나왔다.&#x20;
* 실무에서 이런 경우를 종종 보는데, 이로인해 정말 해결하기 어려운 큰 문제들이 터진다..&#x20;
* 진짜 공유필드는 조심해야 한다! 스프링 빈은 항상 무상태(stateless) 로 설계하자.

## 5. @Configuration 과 싱글톤

* 그런데 이상한 점이 있다. 다음 `AppConfig` 코드를 보자.&#x20;

```java
@Configuration
public class AppConfig {

    @Bean
    public MemberService memberService() {
        return new MemberServiceImpl(memberRepository());
    }
    
    @Bean
    public OrderService orderService() {
        return new OrderServiceImpl(
        memberRepository(),
        discountPolicy());
    }
    
    @Bean
    public MemberRepository memberRepository() {
        return new MemoryMemberRepository();
    }
    ...
}
```

* `memberService` 빈을 만드는 코드를 보면 `memberRepository()` 를 호출한다.
  * 이 메서드를 호출하면 `new MemoryMemberRepository()` 를 호출한다.
* `orderService` 빈을 만드는 코드도 동일하게 `memberRepository()` 를 호출한다.
  * 이 메서드를 호출하면 `new MemoryMemberRepository()` 를 호출한다.

결과적으로 각각 다른 2개의 `MemoryMemberRepository` 가 생성되면서 싱글톤이 깨지는 것 처럼 보인다. 스프링 컨테이너는 이 문제를 어떻게 해결할까?

#### 테스트 코드&#x20;

```java
package hello.core.singleton;

import hello.core.AppConfig;
import hello.core.member.MemberRepository;
import hello.core.member.MemberServiceImpl;
import hello.core.order.OrderServiceImpl;
import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import static org.assertj.core.api.Assertions.assertThat;

public class ConfigurationSingletonTest {

    @Test
    void configurationTest() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

        MemberServiceImpl memberService = ac.getBean("memberService", MemberServiceImpl.class);
        OrderServiceImpl orderService = ac.getBean("orderService", OrderServiceImpl.class);
        MemberRepository memberRepository = ac.getBean("memberRepository", MemberRepository.class);

        MemberRepository memberRepository1 = memberService.getMemberRepository();
        MemberRepository memberRepository2 = orderService.getMemberRepository();

        System.out.println("memberRepository1 = " + memberRepository1);
        System.out.println("memberRepository2 = " + memberRepository2);
        System.out.println("memberRepository = " + memberRepository);

        assertThat(memberRepository).isSameAs(memberRepository1);
        assertThat(memberRepository).isSameAs(memberRepository2);
    }
    
    @Test
    void configurationDeep() {
        ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
        AppConfig bean = ac.getBean(AppConfig.class);

        System.out.println("bean = " + bean.getClass());
    }
}
```

* 확인해보면 `memberRepository` 인스턴스는 모두 같은 인스턴스가 공유되어 사용된다.

## 6. @Configuration과 바이트코드 조작의 마법

스프링 컨테이너는 싱글톤 레지스트리다. 따라서 스프링 빈이 싱글톤이 되도록 보장해주어야 한다. 그런데 스프링이 자바 코드까지 어떻게 하기는 어렵다. 위 자바 코드를 보면 분명 `memberRepository` 는 3번 호출되어야 하는 것이 맞다.

그래서 스프링은 클래스의 바이트코드를 조작하는 라이브러리를 사용한다.

모든 비밀은 `@Configuration` 을 적용한 `AppConfig` 에 있다.

```java
@Test
void configurationDeep() {
    ApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);
    AppConfig bean = ac.getBean(AppConfig.class);

    System.out.println("bean = " + bean.getClass());
}
```

* `AppConfig` 스프링 빈을 조회해서 클래스 정보를 출력해보자.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-28 08.47.18.png" alt=""><figcaption></figcaption></figure>

순수한 클래스라면 다음과 같이 출력되어야 한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-28 08.48.31.png" alt=""><figcaption></figcaption></figure>

그런데 예상과는 다르게 클래스 명에 `xxxCGLIB`가 붙으면서 상당히 복잡해진 것을 볼 수 있다. 이것은 내가 만든 클래스가 아니라 스프링이 CGLIB라는 바이트코드 조작 라이브러리를 사용해서 `AppConfig` 클래스를 상속받은 임의의 다른 클래스를 만들고, 그 다른 클래스를 스프링 빈으로 등록한 것이다!

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-28 08.49.04.png" alt=""><figcaption></figcaption></figure>

그 임의의 다른 클래스가 바로 싱글톤이 보장되도록 해준다. 아마도 다음과 같이 바이트 코드를 조작해서 작성되어 있을 것이다. (실제로는 CGLIB의 내부 기술을 사용하는데 매우 복잡하다.)

#### AppConfig@CGLIB 예상 코드

```java
@Bean
public MemberRepository memberRepository() {

    if (memoryMemberRepository가 이미 스프링 컨테이너에 등록되어 있으면?) {
        return 스프링 컨테이너에서 찾아서 반환;
    } else { //스프링 컨테이너에 없으면
        기존 로직을 호출해서 MemoryMemberRepository를 생성하고 스프링 컨테이너에 등록
        return 반환
    }
}
```

* `@Bean`이 붙은 메서드마다 이미 스프링 빈이 존재하면 존재하는 빈을 반환하고, 스프링 빈이 없으면 생성해서 스프링 빈으로 등록하고 반환하는 코드가 동적으로 만들어진다.
* 덕분에 싱글톤이 보장되는 것이다.

### @Configuration 을 적용하지 않고, @Bean 만 적용하면 어떻게 될까?

`@Configuration` 을 붙이면 바이트코드를 조작하는 CGLIB 기술을 사용해서 싱글톤을 보장하지만, \
만약 `@Bean`만 적용하면 어떻게 될까?

* `@Bean`만 사용해도 스프링 빈으로 등록되지만, 싱글톤을 보장하지 않는다.
  * `memberRepository()` 처럼 의존관계 주입이 필요해서 메서드를 직접 호출할 때 싱글톤을 보장하지 않는다.
* 크게 고민할 것이 없다. 스프링 설정 정보는 항상 `@Configuration` 을 사용하자.
