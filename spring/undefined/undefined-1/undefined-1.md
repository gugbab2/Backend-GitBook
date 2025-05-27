# 스프링 컨테이너와 스프링 빈

## 1. 스프링 컨테이너 생성

스프링 컨테이너 생성되는 과정을 알아보자.

```java
ApplicationContext applicationContext = 
                new AnnotationConfigApplicationContext(AppConfig.class); 
```

* `ApplicationContext` 를 스프링 컨테이너라고 한다.&#x20;
* `ApplicationContext` 는 인터페이스이다.&#x20;
* XML 기반 혹은 어노테이션 기반의 설정 클래스 방식 중 선택할 수 있다.&#x20;
* `AnnotationConfigApplicationContext` 는 `ApplicationContext` 인터페이스의 구현체이다.&#x20;

### 스프링 컨테이너의 생성 과정&#x20;

#### 1. 스프링 컨테이너 생성

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.53.46.png" alt=""><figcaption></figcaption></figure>

* `new AnnotationConfigApplicationContext(AppConfig.class)`
* 스프링 컨테이너를 생성할 때는 구성 정보를 지정해주어야 한다.&#x20;
* 여기서는 `AppConfig.class` 를 구성 정보로 지정했다. &#x20;

#### 2. 스프링 빈 등록&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.54.40.png" alt=""><figcaption></figcaption></figure>

* 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보(`AppConfig.class`) 를 사용해서 스프링 빈을 등록한다.&#x20;
* 빈 이름의 디폴트 값은 메서드 이름이다.
  * 빈 이름을 직접 부여할 수도 있다.&#x20;
  * `@Bean(name="orderService")` 으로 직접 설정 가능&#x20;

> #### 주의 : 빈 이름은 항상 다른 이름을 부여해야 한다.&#x20;
>
> #### 같은 이름을 부여하면, 다른 빈이 무시되거나, 기존 빈을 덮어버리거나 설정에 따라 오류가 발생한다.&#x20;

#### 3. 스프링 빈 의존관계 설정 - 준비

<figure><img src="../../../.gitbook/assets/스크린샷 2025-05-27 15.35.21.png" alt=""><figcaption></figcaption></figure>

#### 4. 스프링 빈 의존관계 설정 - 완료&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 19.56.12.png" alt=""><figcaption></figcaption></figure>

* 스프링 컨테이너는 설정 클래스 정보(`AppConfig.class`) 를 참고해서 의존 관계를 주입(DI) 한다.&#x20;
* 싱글톤 컨테이너로, 단순히 자바 코드를 호출하는 것과는 차이가 있다.&#x20;

> #### 참고&#x20;
>
> 스프링은 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다. 그런데 이렇게 자바 코드로 스프링 빈을 등록하면 생성자를 호출하면서 의존관계 주입도 한번에 처리된다. 여기서는 이해를 돕기 위해 개념적으로 나누어 설명했다. 자세한 내용은 의존관계 자동 주입에서 다시 설명하겠다.

## 2. 컨테이너에 등록된 모든 빈 조회&#x20;

#### 스프링 컨테이너에 실제 스프링 빈들이 잘 등록 되었는지 확인해보자.&#x20;

```java
package hello.core.beanfind;

import hello.core.AppConfig;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class ApplicationContextInfoTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("모든 빈 출략하기")
    void findAllBean() {
        // 스프링에 등록된 모든 빈 이름 조회
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();    
        for (String beanDefinitionName : beanDefinitionNames) {
            // 빈 이름으로 빈 객체(인스턴스)를 조회한다.
            Object bean = ac.getBean(beanDefinitionName);
            System.out.println("bean = " + bean);
        }
    }
    
    @Test
    @DisplayName("애플리케이션 빈 출력하기")
    void findApplicationBean() {
        String[] beanDefinitionNames = ac.getBeanDefinitionNames();
        for (String beanDefinitionName : beanDefinitionNames) {
            BeanDefinition beanDefinition = ac.getBeanDefinition(beanDefinitionName);
            
            // 스프링이 내부에서 사용하는 빈은 제외하고, 내가 등록한 빈만 출력
            if (beanDefinition.getRole() == BeanDefinition.ROLE_APPLICATION) {
                Object bean = ac.getBean(beanDefinitionName);
                System.out.println("bean = " + bean);
            }
        }
    }
}
```

## 3. 스프링 빈 조회 - 기본&#x20;

스프링 컨테이너에서 스프링 빈을 찾는 가장 기본적인 조회 방법&#x20;

* `ac.getBean(빈이름, 타입);`
* `ac.getBean(타입);`

> #### 주의!
>
> 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정해야 한다. \
> `NoSuchBeanDefinitionException: No bean named 'xxxxx' available`

```java
package hello.core.beanfind;

import hello.core.AppConfig;
import hello.core.member.MemberService;
import hello.core.member.MemberServiceImpl;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoSuchBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

public class ApplicationContextBasicFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(AppConfig.class);

    @Test
    @DisplayName("빈 이름으로 조회")
    void findBeanByName() {
        MemberService memberService = ac.getBean("memberService", MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("이름 없이 타임으로만 조회")
    void findBeanByType() {
        MemberService memberService = ac.getBean(MemberService.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("구체 타입으로 조회")
    void findBeanByName2() {
        MemberService memberService = ac.getBean("memberService", MemberServiceImpl.class);
        assertThat(memberService).isInstanceOf(MemberServiceImpl.class);
    }

    @Test
    @DisplayName("빈 이름으로 조회X")
    void findBeanByNameX() {
//        ac.getBean("xxxxx", MemberService.class);
        assertThrows(NoSuchBeanDefinitionException.class,
                () -> ac.getBean("xxxxx", MemberService.class));
    }
}
```

## 4. 스프링 빈 조회 - 동일한 타입이 둘 이상&#x20;

* 타입으로 조회시 같은 타입의 스프링 빈이 둘 이상이면 오류가 발생한다. 이때는 빈 이름을 지정하자.&#x20;
* `ac.getBeansOfType()` 을 사용하면 해당 타입의 모든 빈을 조회할 수 있다.&#x20;

```java
package hello.core.beanfind;

import hello.core.member.MemberRepository;
import hello.core.member.MemoeryMemberRepository;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

public class ApplicationContextSameBeanFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(SameBeanConfig.class);

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 중복 오류가 발생한다.")
    void findBeanByTypeDuplicate() {
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(MemberRepository.class)
        );
    }

    @Test
    @DisplayName("타입으로 조회시 같은 타입이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByName() {
        MemberRepository memberRepository = ac.getBean("memberRepository1", MemberRepository.class);
        assertThat(memberRepository).isInstanceOf(MemberRepository.class);
    }

    @Test
    @DisplayName("특정 타입 모두 조회하기")
    void findAllBeanByType() {
        Map<String, MemberRepository> beansOfType = ac.getBeansOfType(MemberRepository.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("key : " + key + ", valye : " + beansOfType.get(key));
        }
        System.out.println("beansOfType = " + beansOfType);
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Configuration
    static class SameBeanConfig {

        @Bean
        public MemberRepository memberRepository1() {
            return new MemoeryMemberRepository();
        }

        @Bean
        public MemberRepository memberRepository2() {
            return new MemoeryMemberRepository();
        }
    }
}
```

## 5. 스프링 빈 조회 - 상속관계&#x20;

* **부모 타입으로 빈을 조회하면, 자식 타입들도 함께 조회된다.**
  * **`Object` 타입으로 조회하면, 모든 스프링 빈을 조회하게 된다.**
* 때문에 부모 타입으로 조회시, 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-05-27 15.43.39.png" alt=""><figcaption></figcaption></figure>

```java
package hello.core.beanfind;

import hello.core.discount.DiscountPolicy;
import hello.core.discount.FixDiscountPolicy;
import hello.core.discount.RateDiscountPolicy;
import org.junit.jupiter.api.DisplayName;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.NoUniqueBeanDefinitionException;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.Map;

import static org.assertj.core.api.Assertions.*;
import static org.junit.jupiter.api.Assertions.*;

public class ApplicationContextExtendsFindTest {

    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext(TestConfig.class);

    @Test
    @DisplayName("부모 타입으로 조회시 자식이 둘 이상 있으면, 중복 오류가 발생한다.")
    void findBeanByParentTypeDuplicate() {
//        DiscountPolicy bean = ac.getBean(DiscountPolicy.class);
        assertThrows(NoUniqueBeanDefinitionException.class,
                () -> ac.getBean(DiscountPolicy.class));
    }

    @Test
    @DisplayName("부모 타입으로 조회시 자식이 둘 이상 있으면, 빈 이름을 지정하면 된다.")
    void findBeanByParentTypeBeanName() {
        DiscountPolicy rateDiscountPolicy = ac.getBean("rateDiscountPolicy", DiscountPolicy.class);
        assertThat(rateDiscountPolicy).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("특정 하위 타입으로 조회")
    void findBeanBySubType() {
        RateDiscountPolicy bean = ac.getBean(RateDiscountPolicy.class);
        assertThat(bean).isInstanceOf(RateDiscountPolicy.class);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기")
    void findAllBeanByParentType() {
        Map<String, DiscountPolicy> beansOfType = ac.getBeansOfType(DiscountPolicy.class);
        assertThat(beansOfType.size()).isEqualTo(2);
    }

    @Test
    @DisplayName("부모 타입으로 모두 조회하기 - Object")
    void findAllBeanByObjectType() {
        Map<String, Object> beansOfType = ac.getBeansOfType(Object.class);
        for (String key : beansOfType.keySet()) {
            System.out.println("beansOfType.get(key) = " + beansOfType.get(key));
        }
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
}
```

## 6. BeanFactory 와 ApplicationContext&#x20;

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

## 7. 다양한 설정 형식 지원&#x20;

* 스프링 컨테이너는 다양한 형식의 설정 정보를 받아드릴 수 있게 유연한 설계가 되어있다.&#x20;
  * 자바 코드&#x20;
  * XML
  * Groovy
  * ...&#x20;
  *

## 8. 스프링 빈 설정 메타 정보 - BeanDefinition&#x20;

* 스프링이 지원하는 다양한 설정 방식의 중심에는 **`BeanDefinition` 이라는 추상화**가 있다.
* 쉽게 말해 역할과 구현의 개념으로 명확하게 구분한 것이다.
  * XML 을 읽어서 `BeanDefinition` 을 만들면 된다.
  * 자바 코드를 읽어서 `BeanDefinition` 을 만들면 된다.
  * 스프링 컨테이너는 자바 코드인지, XML인지 몰라도 된다. 오직 `BeanDefinition`만 알면 된다.
* `BeanDefinition` 을 빈 설정 메타정보라 말한다.
* 스프링 컨테이너는 해당 메타정보를 바탕으로 스프링 빈을 생성한다.

<figure><img src="../../../.gitbook/assets/스크린샷 2023-05-29 20.06.33.png" alt=""><figcaption></figcaption></figure>
