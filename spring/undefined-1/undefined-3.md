# 의존관계 자동 주입

## 1. 다양한 의존관계 주입 방법

#### 의존관계 주입은 크게 4가지 방법이 있다.&#x20;

* 생성자 주입
* 수정자 주입
* 필드 주입
* 일반 메서드 주입&#x20;

### 생성자 주입

* 이름 그대로 생성자를 통한 의존 관계 주입 방법이다.
* 특징
  * 생성자 호출시점에 딱 1번만 호출되는 것이 보장된다.\
    -> 프로젝트 올라갈 때 스프링 컨테이너의 등록 시!
  * **불변, 필수** 의존관계에 사용

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    @Autowired
    public OrderServiceImpl(MemberRepository memberRepository, 
                            DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

**중요! 생성자가 딱 1개만 있으면 @Autowired를 생략해도 자동 주입 된다.** 물론 스프링 빈에만 해당한다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private final MemberRepository memberRepository;
    private final DiscountPolicy discountPolicy;
    
    public OrderServiceImpl(MemberRepository memberRepository, 
                            DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

### 수정자 주입

* setter라 불리는 필드의 값을 변경하는 수정자 메서드를 통해서 의존관계를 주입하는 방법이다.
* 특징
  * **선택, 변경** 가능성이 있는 의존관계에 사용된다.
  * 자바빈 프로퍼티 규약의 수정자 메서드 방식을 사용하는 방법이다.

```java
@Component
public class OrderServiceImpl implements OrderService {

    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
    
    @Autowired
    public void setMemberRepository(MemberRepository memberRepository) {
        this.memberRepository = memberRepository;
    }
    
    @Autowired
    public void setDiscountPolicy(DiscountPolicy discountPolicy) {
        this.discountPolicy = discountPolicy;
    }
}
```

> 참고: `@Autowired` 의 기본 동작은 주입할 대상이 없으면 오류가 발생한다. 주입할 대상이 없어도 동작하게 하려면 `@Autowired(required = false)` 로 지정하면 된다.

### **필드 주입**

* 이름 그대로 필드에 바로 주입하는 방법이다.
* 특징
  * 코드가 간결해서 개발자들을 유혹, 하지만.. 외부에서 변경이 불가능해서 테스트하기 힘들다는 치명적인 단점이 있다! (Mock 객체 생성 불가)&#x20;
  * DI 프레임워크(스프링) 이 없다면 아무것도 할수가 없다.
  * 사용하지 말자!
    * 애플리케이션의 실제 코드와 관계 없는 테스트 코드를 작성하기가 매우 어렵다. \
      (필드 주입을 사용할 경우, 필요한 객체를 넣어주는 곳이 없다..)

```java
@Component
public class OrderServiceImpl implements OrderService {

    @Autowired
    private MemberRepository memberRepository;
    @Autowired
    private DiscountPolicy discountPolicy;
}
```

> **참고**: 순수한 자바 테스트 코드에는 당연히 @Autowired가 동작하지 않는다. `@SpringBootTest` 처럼 스프링
>
> 컨테이너를 테스트에 통합한 경우에만 가능하다.

### 일반 메서드 주입&#x20;

* 일반 메서드를 통해서 주입 받을 수 있다.&#x20;
* 특징&#x20;
  * 한번에 여러 필드를 주입 받을 수 있다.
  * 일반적으로 잘 사용하지 않는다.&#x20;

```java
@Component
public class OrderServiceImpl implements OrderService {
    
    private MemberRepository memberRepository;
    private DiscountPolicy discountPolicy;
    
    @Autowired
    public void init(MemberRepository memberRepository, 
                    DiscountPolicy discountPolicy) {
        this.memberRepository = memberRepository;
        this.discountPolicy = discountPolicy;
    }
}
```

## 2. 옵션 처리&#x20;

주입할 스프링 빈이 없어도 동작해야 할 때가 있다.&#x20;

그런데 `@Autowired` 만 사용하면 `required` 옵션의 기본값이 `true` 로 되어 있어서 자동 주입 대상이 없으면 오류가 발생한다.

자동 주입 대상을 옵션으로 처리하는 방법은 다음과 같다.

* `@Autowired(required=false)` : 자동 주입할 대상이 없으면 수정자 메서드 자체가 호출 안됨
* `org.springframework.lang.@Nullable` : 자동 주입할 대상이 없으면 `null`이 입력된다.
* `Optional<>` : 자동 주입할 대상이 없으면 `Optional.empty` 가 입력된다.

```java
//호출 안됨
@Autowired(required = false)
public void setNoBean1(Member member) {
    System.out.println("setNoBean1 = " + member);
}

//null 호출
@Autowired
public void setNoBean2(@Nullable Member member) {
    System.out.println("setNoBean2 = " + member);
}

//Optional.empty 호출
@Autowired(required = false)
public void setNoBean3(Optional<Member> member) {
    System.out.println("setNoBean3 = " + member);
}
```

* **Member는 스프링 빈이 아니다.**
* `setNoBean1()` 은 `@Autowired(required=false)` 이므로 호출 자체가 안된다.

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 12.53.24.png" alt=""><figcaption></figcaption></figure>

## 3. 생성자 주입을 선택해라.

#### 불변

* 대부분의 의존관계 주입은 한번 일어나면 애플리케이션 종료 시점에서 의존관계를 변경할 일이 없다.
  * 오히려 대부분의 의존관계는 변하면 안된다. (불변해야 한다)
* 수정자 주입을 사용하게 되면 `setter` 를 `public` 으로 열어두어야 한다.
* 꼭 만들어두면 누군가가 실수를 저지른다.
  * 변경해서는 안되는 경우 메서드를 열어두는 건 좋은 방법이 아니다.
* 생성자 주입은 생성 시 딱 한번만 호출되기에 이후에 호출될 일이 없다! 따라서 불변하게 설계할 수 있다.&#x20;

#### 누락

* 생성자 주입을 사용하게 되면 의존성이 주입되지 않았을 시 컴파일 오류가 발생해 빠르게 오류를 확인할 수 있다.
  * 생성자 주입을 통해 의존성 객체를 `private final` 로 선언하게 되면 의존성이 설정되지 않았을 때, 컴파일 \
    오류가 발생한다.&#x20;
  * 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류다.

#### final 키워드

* 생성자 주입을 사용하면 필드의 `final` 키워드를 사용할 수 있다.
* 생성자에서 필드를 초기화하기 않으면 컴파일 오류가 발생한다.
* 컴파일 오류는 세상에서 가장 빠르고, 좋은 오류다.

#### 정리&#x20;

* 생성자 주입 방식을 선택하는 이유는 여러가지가 있지만, 프레임워크에 의존하지 않고, **순수한 자바 언어의 특징을 잘 살리는 방법이기도 하다.**&#x20;
* 기본으로 생성자 주입을 사용하고, 필수 값이 아닌 경우에는 수정자 주입 방식을 옵션으로 부여하면 된다. 생성자 주입과 수정자 주입을 동시에 사용할 수 있다.&#x20;
* 항상 생성자 주입을 선택하라! 그리고 가끔 옵션이 필요하면 수정자 주입을 선택해라. 필드 주입은 사용하지 않는게 좋다.

## 4. 조회 빈이 2개 이상시 문제

`@Autowired` 는 타입(type) 으로 조회한다.&#x20;

```java
@Autowired
private DiscountPolicy discountPolicy
```

타입으로 조회하기 때문에, 마치 다음 코드와 유사하게 동작한다. (실제로는 더 많은 기능을 제공한다.)\
`ac.getBean(DiscountPolicy.class)`

스프링 빈 조회에서 학습했듯이 타입으로 조회하면 선택된 빈이 2개 이상일 때 문제가 발생한다.\
`DiscountPolicy` 의 하위 타입인 `FixDiscountPolicy`, `RateDiscountPolicy` 둘다 스프링 빈으로 선언해보자.

```java
@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
public class RateDiscountPolicy implements DiscountPolicy {}
```

그리고 이렇게 의존관계 자동 주입을 실행하면

```java
@Autowired
private DiscountPolicy discountPolicy
```

`NoUniqueBeanDefinitionException` 오류가 발생한다.

```
NoUniqueBeanDefinitionException: No qualifying bean of type
'hello.core.discount.DiscountPolicy' available: expected single matching bean
but found 2: fixDiscountPolicy,rateDiscountPolicy
```

이때 하위 타입으로 지정할 수 도 있지만, **하위 타입으로 지정하는 것은 DIP를 위배하고 유연성이 떨어진다.** \
그리고 이름만 다르고, 완전히 똑같은 타입의 스프링 빈이 2개 있을 때 해결이 안된다.

## 5. @Autowired 필드 명, @Qualifier, @Primary

### @Autowired 필드 명 매칭

`@Autowired` 는 타입 매칭을 시도하고, 이때 여러 빈이 있으면 필드 이름, 파라미터 이름으로 빈 이름을 추가 매칭한다.

#### 기존 코드&#x20;

```java
@Autowired
private DiscountPolicy discountPolicy
```

필드 명을 빈 이름으로 변경

```java
@Autowired
private DiscountPolicy rateDiscountPolicy
```

필드 명이 `rateDiscountPolicy` 이므로 정상 주입된다.

**필드 명 매칭은 먼저 타입 매칭을 시도 하고 그 결과에 여러 빈이 있을 때 추가로 동작하는 기능이다.**

### @Qualifier 사용

`@Qualifier` 는 추가 구분자를 붙여주는 방법이다. 주입시 추가적인 방법을 제공하는 것이지 빈 이름을 변경하는 것은 아니다.

**빈 등록시 @Qualifier를 붙여 준다.**

```java
@Component
@Qualifier("mainDiscountPolicy")
public class RateDiscountPolicy implements DiscountPolicy {}
```

```java
@Component
@Qualifier("fixDiscountPolicy")
public class FixDiscountPolicy implements DiscountPolicy {}
```

**주입시에 @Qualifier를 붙여주고 등록한 이름을 적어준다.**

```java
@Autowired
public OrderServiceImpl(MemberRepository memberRepository,
    @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.memberRepository = memberRepository;
    this.discountPolicy = discountPolicy;
}
```

```java
@Autowired
public DiscountPolicy setDiscountPolicy(
    @Qualifier("mainDiscountPolicy") DiscountPolicy discountPolicy) {
    this.discountPolicy = discountPolicy;
}
```

### @Primary 사용

`@Primary` 는 우선순위를 정하는 방법이다. `@Autowired` 시에 여러 빈이 매칭되면 `@Primary` 가 우선권을 가진다.

`rateDiscountPolicy` 가 우선권을 가지도록 하자.

```java
@Component
@Primary
public class RateDiscountPolicy implements DiscountPolicy {}

@Component
public class FixDiscountPolicy implements DiscountPolicy {}
```

#### @Primary, @Qualifier 활용

코드에서 자주 사용하는 메인 데이터베이스의 커넥션을 획득하는 스프링 빈이 있고, 코드에서 특별한 기능으로 가끔 사용하는 서브 데이터베이스의 커넥션을 획득하는 스프링 빈이 있다고 생각해보자. 메인 데이터베이스의 커넥션을 획득하는 스프링 빈은 `@Primary` 를 적용해서 조회하는 곳에서 `@Qualifier` 지정 없이 편리하게 조회하고, 서브 데이터베이스 커넥션 빈을 획득할 때는 `@Qualifier` 를 지정해서 명시적으로 획득 하는 방식으로 사용하면 코드를 깔끔하게 유지할 수 있다. 물론 이때 메인 데이터베이스의 스프링 빈을 등록할 때 \`@Qualifier\` 를 지정해주는 것은 상관없다.

#### @Primary, @Qualifier 우선순위

`@Primary` 는 기본값 처럼 동작하는 것이고, `@Qualifier` 는 매우 상세하게 동작한다.&#x20;

이런 경우 어떤 것이 우선권을 가져갈까? 스프링은 자동보다는 수동이, 넒은 범위의 선택권 보다는 좁은 범위의 선택권이 우선 순위가 높다. 따라서 여기서도 `@Qualifier` 가 우선권이 높다.

## 6. 실무 운영 기준

#### 편리한 자동 기능을 기본으로 사용하자

결론부터 이야기하면, 스프링이 나오고 시간이 갈 수록 점점 자동을 선호하는 추세다. 스프링은 `@Component` 뿐만 아니라 `@Controller` , `@Service` , `@Repository`처럼 계층에 맞추어 일반적인 애플리케이션 로직을 자동으로 스캔할 수 있도록 지원한다. \
거기에 더해서 최근 스프링 부트는 컴포넌트 스캔을 기본으로 사용하고, 스프링 부트의 다양한 스프링 빈들도 조건이 맞으면 자동으로 등록하도록 설계했다.

설정 정보를 기반으로 애플리케이션을 구성하는 부분과 실제 동작하는 부분을 명확하게 나누는 것이 이상적이지만, 개발자 입장에서 스프링 빈을 하나 등록할 때 `@Component` 만 넣어주면 끝나는 일을 `@Configuration` 설정 정보에 가서 `@Bean` 을 적고, 객체를 생성하고, 주입할 대상을 일일이 적어주는 과정은 상당히 번거롭다.\
또, 관리할 빈이 많아서 설정 정보가 커지면 설정 정보를 관리하는 것 자체가 부담이 된다. 그리고 결정적으로 자동 빈 등록을 사용해도 OCP, DIP 를 지킬 수 있다.

#### 수동 빈 등록은 언제 사용하냐?

* 업무 로직 빈: 웹을 지원하는 컨트롤러, 핵심 비즈니스 로직이 있는 서비스, 데이터 계층의 로직을 처리하는 리포지토리등이 모두 업무 로직이다. 보통 비즈니스 요구사항을 개발할 때 추가되거나 변경된다.
* 기술 지원 빈: 기술적인 문제나 공통 관심사(AOP)를 처리할 때 주로 사용된다. 데이터베이스 연결이나, 공통 로그 처리 처럼 업무 로직을 지원하기 위한 하부 기술이나 공통 기술들이다.
* 업무 로직은 숫자도 매우 많고, 한번 개발해야 하면 컨트롤러, 서비스, 리포지토리 처럼 어느정도 유사한 패턴이 있다. 이런 경우 자동 기능을 적극 사용하는 것이 좋다. 보통 문제가 발생해도 어떤 곳에서 문제가 발생했는지 명확하게 파악하기 쉽다.
* 기술 지원 로직은 업무 로직과 비교해서 그 수가 매우 적고, 보통 애플리케이션 전반에 걸쳐서 광범위하게 영향을 미친다. 그리고 업무 로직은 문제가 발생했을 때 어디가 문제인지 명확하게 잘 드러나지만, 기술 지원 로직은 적용이 잘 되고 있는지 아닌지 조차 파악하기 어려운 경우가 많다. 그래서 이런 기술 지원 로직들은 가급적 수동 빈 등록을 사용해서 명확하게 드러내는 것이 좋다.

**애플리케이션에 광범위하게 영향을 미치는 기술 지원 객체는 수동 빈으로 등록해서 딱! 설정 정보에 바로 나타나게 하는 것이 유지보수 하기 좋다.**

### 비지니스 로직 중 다형성을 적극 활용할 때

다형성을 적극 활용하는 코드를 상상해보자..

여기에 어떤 빈들이 주입될 지, 각 빈들의 이름은 무엇일지 코드만 보고 한번에 쉽게 파악할 수 있을까? 내가 개발했으니 크게 관계가 없지만, 만약 이 코드를 다른 개발자가 개발해서 나에게 준 것이라면 어떨까?

자동 등록을 사용하고 있기 때문에 파악하려면 여러 코드를 찾아봐야 한다.\
이런 경우 수동 빈으로 등록하거나 또는 자동으로하면 특정 패키지에 같이 묶어두는게 좋다!\
-> 핵심은 딱 보고 이해가 되어야 한다!

```java
@Configuration
public class DiscountPolicyConfig {

    @Bean
    public DiscountPolicy rateDiscountPolicy() {
        return new RateDiscountPolicy();
    }
    
    @Bean
    public DiscountPolicy fixDiscountPolicy() {
        return new FixDiscountPolicy();
    }
}
```
