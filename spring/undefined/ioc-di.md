# IoC / DI

> ### 라이브러리와 프레임워크의 차이
>
> #### 라이브러리
>
> * 라이브러리는 특정 기능을 수행하는 함수, 클래스, 또는 메서드들의 집합으로, 개발자가 필요할 때 호출하여 하용하는 도구이다.&#x20;
> * **개발자가 제어권을 가진다.**&#x20;
> * **개발시 필요한 도구를 제공한다.**&#x20;
>
> #### 프레임워크
>
> * 애플리케이션의 구조와 동작 방식을 정의하고, 개발자가 해당 구조 안에서 코드를 작성하도록 설계된 도구이다.&#x20;
> * **프레임워크가 제어권을 가진다.**&#x20;
> * **애플리케이션의 뼈대를 제공한다.**&#x20;

## 1. IoC(Inversion of Control)

* 제어의 역전이라는 의미로, **스프링에서 오브젝트(빈)의 생성과 의존 관계 설정, 사용, 제거 등의 작업을 코드 대신 스프링 컨테이너가 담당한다.**
  * 이를 스프링 컨테이너가 오브젝트에 대한 제어권을 가지고 있다고 해서 IoC 라고 부른다.\
    &#xNAN;**(스프링 자체가  라이브러리가 아닌 프레임워크이기 때문)**
* 따라서, 스프링 컨테이너를 IoC 컨테이너라고 부른다.
  * 오브젝트(빈)을 IoC 컨테이너에서 관리하므로, 개발자는 비지니스 로직에 집중할 수 있다.
  * 대신, 어떻게 빈이 프레임워크에서 관리되는지는 알아야겠지?&#x20;

### 1-1. IoC 컨테이너란?

* 스프링에서는 IoC 를 담당하는 컨테이너를 빈 팩토리, DI 컨테이너, 애플리케이션 컨텍스트 라고 다양하게 부른다.
  * 오브젝트 생성과 오브젝트 사이의 런타임 관계를 설정하는 **DI 관점**으로 바라볼 때, 컨테이너를 **빈 팩토리, 또는 DI 컨테이너**라 부른다.
  * 그러나 스프링 컨테이너는 **단순한 DI 작업보다 더 많은 일을 하는데**, DI 를 위한 빈 팩토리의 여러가지 기능을 추가한 것을 **애플리케이션 컨텍스트**라고 부른다.
* 정리하면, 애플리케이션 컨텍스트는 그 자체로 IoC와 DI 그 이상의 기능을 가졌다고 생각하면 된다.

### 1-2. 빈 팩토리와 애플리케이션 컨텍스트의 관계

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

#### 빈 팩토리

* 스프링 컨테이너의 최상위 인터페이스
* 스프링 빈을 관리하고 조회하는 역할을 담당한다.
* 대표적으로 `getBean()` 메서도를 제공한다.

#### 애플리케이션 컨텍스트

* 애플리케이션 컨텍스트는 빈 팩토리의 기능을 모두 상속받아서 사용한다.
* 때문에 빈 팩토리의 없는 다음의 기능들을 가지고 있다.
  * 메시지 소스를 활용한 국제화 기능
  * 환경변수
  * 애플리케이션 이벤트
  * 편리한 리소스 조회
  * ...

### 1-3. 설정 메타 정보

* IoC 컨테이너의 가장 기초적인 역할은 오브젝트를 생성하고 관리하는 것이다.
  * 스프링 컨테이너가 관리하는 이러한 오브젝트를 빈이라고 한다.
* 스프링 컨테이너는 자바코드(생성자, 수정자...), XML, Groovy 등 다양한 설정 정보를 받아들일 수 있도록 유연하게 설계되어 있다.

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### 1-4. 스프링 빈 설정 메타 정보(BeanDefinition)

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

* 스프링이 이러한 다양한 설정을 제공하는데 있어 그 중심에는 `BeanDefinition` 이라는 추상화가 있다.
* 쉽게 말하면, 설정정보(XML, 자바코드, 어노테이션) 를 읽어서 `BeanDefinition` 을 만든다.
  * 따라서 스프링 컨테이너는 오직 `BeanDefinition` 만 알면 된다.
  * 이상적인  OOP 가 지켜진 것을 확인할 수 있다.&#x20;

## 2. DI(의존관계 주입)

### 2-1. 의존관계(Dependency)

* 프로그래밍에서 의존이란 A 객체를 수정할 때 B 의 기능이 추가되거나 변경되면 두 객체는 서로 의존관계에 있다고 볼 수 있다.
* **객체지향 프로그래밍에서는 구체화된 객체를 의존하는 것이 아닌, 인터페이스를 의존하게 되면, 확장성 있는 의존관계를 맺을 수 있다.**

### 2-2. 의존관계 주입(Dependency Injection)

* 의존 관계를 외부에서 결정하는 것을 DI 라고 한다.
  * **DI 를 통해서 객체간의 의존성이 줄어든다.**
  * 메서드 매개변수를 통해서 의존관계를 주입한다. \
    -> **요구사항이 변경되었을 때 구현체만 변경하면 된다.**
* 스프링에서는 외부의 대상이 IoC 컨테이너가 되어, 빈을 알아서 주입해 준다.
  * IoC 는 프레임워크의 특징이다.&#x20;

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

### 2-3. 의존관계 주입의 방법&#x20;

* XML 등을 사용한 의존관계 주입도 있지만, 현재는 사용하고 있지 않다고 보는게 맞다!
* 래거시 프로젝트에서만 XML 을 통한 의존성 주입의 코드를 볼 수 있을 것이다.&#x20;

#### 필드 주입

* 의존성을 주입받을 필드에 `@Autowired` 를 붙이는 방식&#x20;

```java
@Component
class UserService {
    @Autowired
    private UserRepository userRepository;
}
```

* 장점&#x20;
  * 간단하고 코드가 짧아짐&#x20;
  * 테스트 및 빠른 개발 단계에서 빠르게 작성 가능&#x20;
* 단점&#x20;
  * **테스트 어려움 : 필드가 `private` 이므로 `Mock` 객체를 직접 주입하기가 어렵다.**&#x20;
  * **강한 결합 : 의존성을 외부에서 변경할 수 없기에 의존성이 떨어짐**&#x20;
  * 권장되지 않음 : 스프링 커뮤니티에서 권장하지 않는 방법이다.&#x20;

#### 수정자 주입

* `@Autowired` 를 사용하여 수정자 메서드를 통해 의존성을 주입받는 방식

```java
@Component
class UserService {
    private UserRepository userRepository;

    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

* 장점&#x20;
  * 유연성 : 의존성을 필요에 따라 변경하거나 주입하지 않을 수도 있다.&#x20;
  * 테스트 편리성 : `setter` 메서드를 통해서 쉽게 `Mock` 객체 주입 가능.
* 단점&#x20;
  * **의존성이 필수인 경우에도 명시적으로 표시되지 않음 (컴파일 타임에 체크 불가)**&#x20;
  * **생성자 주입보다 DI 강제성이 낮다.**&#x20;

#### 생성자 주입

* 생성자를 통해 의존성을 주입받는 방식으로, 의존성을 생성자 매개변수로 전달받는다.&#x20;

```java
@Component
class UserService {
    private final UserRepository userRepository;

    @Autowired
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

* 장점
  * 불변성 : `final` 키워드 사용 가능, 의존성이 변경되지 않음을 보장&#x20;
  * 의존성 강제 : 객체 생성 시 필요한 의존성을 강제하므로,  누락될 가능성이 없다.&#x20;
  * 테스트 용이 : 생성자를 통해 의존성을 명시적으로 주입하므로, `Mock` 객체 사용이 쉽다.&#x20;
  * 권장되는 방식 : 스프링 커뮤니티에서 권장하는 방식이다.&#x20;
* 단점
  * **아래와 같이 의존성이 많을 경우 코드가 길어질 수 있다.**&#x20;
  * **Lombok 의 `@RequiredArgsConstructor` 를 사용하면 이 단점을 극복할 수 있다.**&#x20;

```java
@Component
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    private final UserService userService;
    private final DiscountService discountService;

    // 생성자 주입
    @Autowired
    public OrderService(PaymentService paymentService,
                        InventoryService inventoryService,
                        NotificationService notificationService,
                        UserService userService,
                        DiscountService discountService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
        this.notificationService = notificationService;
        this.userService = userService;
        this.discountService = discountService;
    }
}

// ---------------------------------------------------------------------

@RequiredArgsConstructor
@Component
public class OrderService {
    private final PaymentService paymentService;
    private final InventoryService inventoryService;
    private final NotificationService notificationService;
    private final UserService userService;
    private final DiscountService discountService;
}
```

### 2-4. 순환 참조(Circular Reference)&#x20;

* 스프링 순환 참조랑 서로 다른 빈들이 서로 참조를 맞물리게 주입되면서 생기는 현상이다.&#x20;
  * BeanA 에서 BeanB 를 참조하게 되는데, BeanB 에서 BeanA 를 참조해야 하는 경우 순환참조 문제가 생긴다.&#x20;
* 즉, 스프링에서 어떤 스프링 빈을 먼저 만들어야 할 지 결정할 수 없게 되는 상황이라 할 수 있다.
  * 이 순환 참조 문제도 DI 를 하는 방법 3가지 상황에서 발생할 수 있다.&#x20;

#### 순환 참조가 발생하는 3가지 케이스&#x20;

* **생성자 주입 방식**&#x20;
  * **어플리케이션 구동시,** 스프링 컨테이너는 BeanA 를 생성하기 위해 BeanB 를 주입해주어야 하기 때문에 BeanB 를 찾을것이다. &#x20;
  * 근데, BeanB 를 생성하려 하니 BeanA 가 필요해서 BeanA 를 찾게 되며 무한 반복이 생기게 된다.&#x20;

```java
@Component
public class BeanA {
   private BeanB beanB;
   
   @Autowired
   public void BeanA(BeanB beanB){
      this.beanB = beanB;
   }
}

@Component
public class BeanB {
   private BeanA beanA;
   
   @Autowired
   public void BeanB(BeanA beanA){
      this.beanA = beanA;
   }
}
```

<figure><img src="../../.gitbook/assets/image (9) (1) (1).png" alt="" width="548"><figcaption></figcaption></figure>

* 필드, Setter 주입 방식&#x20;
  * 이 두가지 DI 방식은 **어플리케이션 구동 시 DI 를 하지 않는다.**&#x20;
  * **이 두가지 방식은 어플리케이션 구동 시점에서 필요한 의존성이 없다면 `null` 상태로 유지하고 실제로 사용하는 시점에 주입하기 때문에,**&#x20;
  * **이 두가지 방식은 모두 순환참조를 일으킬 수 있는 메서드를 호출하는 시점에 순환참조 문제가 발생할 것이다.**&#x20;

<pre class="language-java"><code class="lang-java"><strong>// 필드 주입 방식 
</strong><strong>@Component
</strong>public class BeanA {

   @Autowired
   private BeanB beanB;

   public void run(){
       beanB.run();
   }

   public void call(){
      log.info("called BeanA");
   }
}

@Component
public class BeanB {
   @Autowired
   private BeanA beanA;

   public void run(){
      log.info("Called BeanB");
      beanA.call();
   }
}

// Setter 주입 방식 
@Component
public class BeanA {
   private BeanB beanB;

   @Autowired
   public void setBeanB(BeanB beanB){
      this.beanB = beanB;
   }

   public void run(){
      beanB.run();
   }

   public void call(){
      log.info("called BeanA");
   }
}

@Component
public class BeanB {
   private BeanA beanA;

   @Autowired
   public void setBeanA(BeanA beanA){
      this.beanA = beanA;
   }

   public void run(){
      log.info("Called BeanB");
      beanA.call();
   }
}
</code></pre>

#### 해결책&#x20;

* 결국 순환을 끊으므로써 순환참조 문제를 해결해야 하는데, 스프링에서는 `@Lazy` 라는 애노테이션을 통해 이런 순환참조를 끊을 수 있도록 한다.
* 하지만 스프링 커뮤니티에서는 이러한 방식을 추천하지 않는다. 그 이유는 다음과 같다.&#x20;
  * **설계 결함이 가려짐**&#x20;
    * 순환 참조는 설계상 오류일 가능성이 크다.&#x20;
    * `@Lazy` 를 사용하면 문제가 해결된 것처럼 보이지만, 애플리케이션의 의존성 구조가 비대하고 복잡한 상태로 유지된다.&#x20;
  * **런타임 지연 가능성**&#x20;
    * `Bean` 초기화가 지연되면서 실행 시 성능 저하나 예상치 못한. `LazyInitializationException` 이 발생할 수 있다.&#x20;
    * 특히, 지연 초기화된 Bean 에서 다른 프록시 Bean 을 호출할 때 복잡한 버그기 발생할 가능성이 높아진다.&#x20;
  * **테스트 및 디버깅 어려움**&#x20;
    * 의존성이 지연 초기화되므로 어떤 시점에 Bean 이 생성되었는지 추적하기가 어렵다.&#x20;
    * 순환 참조 문제가 코드 리뷰나 테스트 단계에서 드러나지 않을 수 있다.&#x20;
  * **의존성 설계 복잡화**&#x20;
    * 순환 참조 구조가 유지되면서 의존 관계가 복잡해지고, 코드의 모듈성을 해칩니다.
* **결론적으로 순환참조가 발생하지 않는 구조로 설계를 변경하는 것이 좋다.**&#x20;

```java
@Component
public class BeanA {
   private BeanB beanB;
   
   @Autowired
   public void BeanA(BeanB beanB){
      this.beanB = beanB;
   }
}

@Component
public class BeanB {
   private BeanA beanA;
   
   @Autowired
   public void BeanB(@Lazy BeanA beanA){
      this.beanA = beanA;
   }
}
```

### 2-5. @Autowired

* DI 를 할 때 사용하는 어노테이션으로, 의존 관계의 타입에 해당하는 빈을 찾아 주입하는 역할을 한다.
* 스프링 서버가 올라갈 때 애플리케이션 컨텍스트가 `@Bean`, `@Service`, `@Controller` 등 어노테이션을 이용하여 등록한 빈을 생성하고, `@Autowired` 어노테이션이 붙은 위치에 의존관계 주입을 실행하게 된다.
* 해당 어노테이션에 빈을 주입하는 것은 `BeanPostProcessor` 라는 내용을 찾을 수 있고, 그것의 구현체는 `AutowiredAnnotationBeanPostProcessor` 인 것을 확인할 수 있다.

## 3. 결론(DI 와 IoC 의 차이는?)

* **DI : 의존관계를 어떻게 주입할 것인가?**&#x20;
  * 생성자, 필드, 수정자
* **IoC : 누가 소프트웨어의 제어권을 가지고 있는가?**
  * 스프링 컨테이너가 빈을 생성할 때 Bean 간에 의존관계를 DI 를 통해서 해결한다.
* **DI 는 IoC 를 필수적으로 사용하지 않는다.**
  * 자바 코드(생성자, 수정자) 를 통해서 의존성을 직접 주입할 수도 있다.
