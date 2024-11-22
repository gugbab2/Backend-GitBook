---
description: 빈
---

# 스프링 컨테이너와 스프링 빈

## 1. 스프링 컨테이너 생성 과정

#### 결론적으로 스프링 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다!

### 스프링 컨테이너 생성&#x20;

* `ApplicationContext` 를 스프링 컨테이너라고 한다.&#x20;
* XML 기반 혹은 어노테이션 기반의 설정 클래스 방식 중 선택할 수 있다.&#x20;
* `AnnotationConfigApplicationContext` 는 `ApplicationContext` 인터페이스의 구현체이다.&#x20;

```java
ApplicationContext applicationContext = 
                new AnnotationConfigApplicationContext(AppConfig.class);
```

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.53.46.png" alt=""><figcaption></figcaption></figure>

### 스프링 빈 등록&#x20;

* 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보(`AppConfig.class`) 를 사용해서 스프링 빈을 등록한다.&#x20;
* 빈 이름의 디폴트 값은 메서드 이름이다.&#x20;
  * `@Bean(name="orderService")` 으로 직접 설정 가능&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.54.40.png" alt=""><figcaption></figcaption></figure>

### 스프링 빈 의존 관계 설정&#x20;

* 스프링 컨테이너는 설정 클래스 정보(`AppConfig.class`) 를 참고해서 의존 관계를 주입(DI) 한다.&#x20;
  * 이 때 순환 참조가 일어나지 않도록 설계에 주의해야 한다.&#x20;
* 싱글톤 컨테이너로, 단순히 자바 코드를 호출하는 것과는 차이가 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.56.12.png" alt=""><figcaption></figcaption></figure>

## 2. 스프링 빈 조회&#x20;

### 기본&#x20;

1. `ac.getBean(빈이름, 타입);`
2. `ac.getBean(타입);`

> #### 주의!
>
> 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정행 해야 한다.&#x20;

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

@Test
@DisplayName("빈 이름으로 조회")
void findBeanByName() {
    OrderService orderService = ac.getBean("orderService", OrderService.class);
    Assertions.assertThat(orderService).isInstanceOf(OrderServiceImpl.class);
}

@Test
@DisplayName("이름 없이 타입으로 조회")
void findBeanByType() {
    OrderService orderService = ac.getBean(OrderService.class);
    Assertions.assertThat(orderService).isInstanceOf(OrderServiceImpl.class);
}

@Test
@DisplayName("구체 타입으로 조회")
void findBeanByName2() {
    OrderService orderService = ac.getBean("orderService", OrderServiceImpl.class);
    Assertions.assertThat(orderService).isInstanceOf(OrderServiceImpl.class);
}

@Test
@DisplayName("빈 이름으로 조회X")
void findBeanByNameX() {
//    Object xxxxxx = ac.getBean("XXXXXX");
    org.junit.jupiter.api.Assertions.assertThrows(NoSuchBeanDefinitionException.class,
            () -> ac.getBean("XXXXXX"));
}
```

### 상속관계&#x20;

* **부모 타입으로 빈을 조회하면, 자식 타입들도 함께 조회된다.**&#x20;
  * **`Object` 타입으로 조회하면, 모든 스프링 빈을 조회하게 된다.**&#x20;
* 때문에 부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.&#x20;

```java
AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

@Test
@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 중복 오류가 발생한다")
void findBeanByParentTypeDuplicate() {
    assertThrows(NoUniqueBeanDefinitionException.class,
            () -> ac.getBean(DiscountPolicy.class));
}

@Test
@DisplayName("부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다")
void findBeanByParentTypeBeanName() {
    DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
    assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
}

@Test
@DisplayName("부모 타입으로 모두 조회하기")
void findAllBeanByParentType() {
    Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
    assertThat(beansOfType.size()).isEqualTo(2);
}

@Configuration
static class TestConfig {
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

## 3. BeanFactory 와 ApplicationContext&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.59.24.png" alt="" width="563"><figcaption></figcaption></figure>

### BeanFactory&#x20;

* 스프링 컨테이너의 최상위 인터페이스&#x20;
* 스프링 빈을 관리하고 조회하는 역할을 담당&#x20;
* `getBean()` 을 제공&#x20;
* 위 테스트 코드에서 사용한 대부분 기능을 `BeanFactory` 가 제공&#x20;

### ApplicationContext

* BeanFactory 기능을 모두 상속받아서 제공&#x20;
* 빈 관리 및 조회 기능 뿐만이 아니라, 여러 부가 기능을 제공&#x20;
  * 메시지 소스를 활용한 국제화 기능
  * 환경변수 : 로컬, 개발, 운영 등을 구분해서 처리
  * 애플리케이션 이벤트
  * 편리한 리소스 조회 등 ..

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 20.01.55.png" alt=""><figcaption></figcaption></figure>

## 4. 다양한 설정 형식 지원&#x20;

* 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연한 설계가 되어있다.&#x20;
  * 자바 코드&#x20;
  * XML
  * Groovy
  * ...&#x20;

> #### BeanDefinition&#x20;
>
> * 스프링이 지원하는 다양한 설정 방식의 중심에는 **`BeanDefinition` 이라는 추상화**가 있다.
> * 쉽게 말해 역할과 구현의 개념으로 명확하게 구분한 것이다.
>   * XML 을 읽어서 `BeanDefinition` 을 만들면 된다.
>   * 자바 코드를 읽어서 `BeanDefinition` 을 만들면 된다.
> * `BeanDefinition` 을 빈 설정 메타정보라 말한다.
> * 스프링 컨테이너는 해당 메타정보를 바탕으로 스프링 빈을 생성한다.



<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 20.06.33.png" alt=""><figcaption></figcaption></figure>
