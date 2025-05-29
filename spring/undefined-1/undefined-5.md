# 빈 스코프

## 1. 빈 스코프란?

지금까지 우리는 스프링 빈이 스프링 컨테이너의 시작과 함께 생성되어서 스프링 컨테이너가 종료될 때 까지 유지된다고 학습했다. 이것은 스프링 빈이 기본적으로 싱글톤 스코프로 생성되기 때문이다. 스코프는 번역 그대로 빈이 존재할 수 있는 범위를 뜻한다.&#x20;

#### 스코프는 빈이 존재할 수 있는 범위를 의미하며, 스프링은 다음과 같은 다양한 스코프를 지원한다.

* **싱글톤** : 기본 스코프, 스프링 컨테이너의 시작과 종료까지 유지되는 가장 넓은 범위의 스코프
* **프로토타입** : 스프링 컨테이너는 프로토타입 빈의 생성과 의존관계 주입까지만 관여하고 더는 관리하지 않는 매우 짧은 범위의 스코프
* **웹 관련 스코프**
  * **request** : 웹 요청이 들어오고 나갈때까지 유지되는 스코프이다.&#x20;
  * **session** : 웹 세션이 생성되고 종료될 때 까지 유지되는 스코프이다.&#x20;
  * **application** : 웹의 서블릿 컨텍스트와 같은 범위로 유지되는 스코프이다.&#x20;

빈 스코프는 다음과 같이 지정할 수 있다.&#x20;

#### 컴포넌트 스캔 자동 등록&#x20;

```java
@Scope("prototype")
@Component
public class HelloBean {}
```

#### 수동 등록&#x20;

<pre class="language-java"><code class="lang-java">@Scope("prototype")
@Bean
PrototypeBean HelloBean() {
<strong>    return new HelloBean();
</strong>}
</code></pre>

## 2. 프로토타입 스코프&#x20;

싱글톤 스코프의 빈을 조회하면 스프링 컨테이너는 항상 같은 인스턴스의 스프링 빈을 반환한다. 반면에 프로토타입 스코프를 스프링 컨테이너에 조회하면 스프링 컨테이너는 항상 새로운 인스턴스를 생성해서 반환한다.&#x20;

#### 싱글톤 빈 요청&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 20.46.45.png" alt=""><figcaption></figcaption></figure>

1. 싱글톤 스코프의 빈을 스프링 컨테이너에 요청한다.&#x20;
2. 스프링 컨테이너는 본인이 관리하는 스프링 빈을 반환한다. \
   (프로세스가 올라올 때 스프링 컨테이너가 싱글톤 빈을 생성하고 의존성을 주입한다)
3. 이후에 스프링 컨테이너에 같은 요청이 와도 같은 객체 인스턴스의 스프링 빈을 반환한다.&#x20;

#### 프로토타입 빈 요청1&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 20.47.57.png" alt=""><figcaption></figcaption></figure>

1. 프로토타입 스코프의 빈을 스프링 컨테이너에 요청한다.
2. 스프링 컨테이너는 이 시점에 프로토타입 빈을 생성하고, 필요한 의존관계를 주입한다.&#x20;

#### 프로토타입 빈 요청2

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 20.51.00.png" alt=""><figcaption></figcaption></figure>

3. 스프링 컨테이너는 생성한 프로토타입 빈을 클라이언트에 반환한다.&#x20;
4. 이후에 스프링 컨테이너에 같은 요청이 오면 항상 새로운 프로토타입 빈을 생성해서 반환한다.

#### 정리&#x20;

여기서 **핵심은 스프링 컨테이너는 프로토타입 빈을 생성하고, 의존관계 주입, 초기화까지만 처리한다는 것이다.** 클라이언트에 빈을 반환하고, 이후 스프링 컨테이너는 생성된 프로토타입 빈을 관리하지 않는다. 프로토타입 빈을 관리할 책임은 프로토타입 빈을 받은 클라이언트에 있다. 그래서 `@PreDestroy` 같은 종료 메서드가 호출되지 않는다.

## 3. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 문제점

스프링 컨테이너에 프로토타입 스코프 빈을 요청하면 항상 새로운 객체 인스턴스를 생성해서 반환한다. 하지만 싱글톤 빈과 함께 사용할 때는 의도한 대로 잘 동작하지 않으므로 주의해야 한다.&#x20;

### 프로토타입 빈 직접 요청&#x20;

#### 스프링 컨테이너에 프로토타입 빈 직접 요청1

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 20.55.55.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트 A 는 스프링 컨테이너에 프로토타입 빈을 요청한다.&#x20;
2. 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환한다. 해당 빈의 count 필드 값은 0 이다.&#x20;
3. 클라이언트는 조회한 프로토타입 빈에 `addCount()` 를 호출하면서 count 필드 +1 한다.&#x20;
4. 결과적으로 프로토타입 빈의 count는 1이 된다.

#### 스프링 컨테이너에 프로토타입 빈 직접 요청2

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 20.57.55.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트 B 는 스프링 컨테이너에 프로토타입 빈을 요청한다.&#x20;
2. 스프링 컨테이너는 프로토타입 빈을 새로 생성해서 반환한다. 해당 빈의 count 필드 값은 0 이다.&#x20;
3. 클라이언트는 조회한 프로토타입 빈에 `addCount()` 를 호출하면서 count 필드 +1 한다.&#x20;
4. 결과적으로 프로토타입 빈의 count는 1이 된다.

#### 테스트 코드 &#x20;

```java
public class SingletonWithPrototypeTest1 {

    @Test
    void prototypeFind() {
        AnnotationConfigApplicationContext ac = new
            AnnotationConfigApplicationContext(PrototypeBean.class);
        PrototypeBean prototypeBean1 = ac.getBean(PrototypeBean.class);
        prototypeBean1.addCount();
        assertThat(prototypeBean1.getCount()).isEqualTo(1);
        
        PrototypeBean prototypeBean2 = ac.getBean(PrototypeBean.class);
        prototypeBean2.addCount();
        assertThat(prototypeBean2.getCount()).isEqualTo(1);
    }
    
    @Scope("prototype")
    static class PrototypeBean {
        
        private int count = 0;
        
        public void addCount() {
            count++;
        }
    
        public int getCount() {
            return count;
        }
        
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
        
        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

### 싱글톤 빈에서 프로토타입 빈 사용&#x20;

이번에는 `clientBean` 이라는 싱글톤 빈이 의존관계 주입을 통해서 프로토타입 빈을 주입받아서 사용하는 예를 보자.

#### 싱글톤에서 프로토타입 빈 사용1

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 21.01.30.png" alt=""><figcaption></figcaption></figure>

* `clientBean` 은 싱글톤이므로, 보통 스프링 컨테이너 생성 시점에 함께 생성되고, 의존관계 주입도 발생한다.
* 1\. `clientBean` 은 의존관계 자동 주입을 사용한다. 주입 시점에 스프링 컨테이너에 프로토타입 빈을 요청한다.
* 2\. 스프링 컨테이너는 프로토타입 빈을 생성해서 `clientBean` 에 반환한다. 프로토타입 빈의 count 필드 값은 0이다.
* 이제 `clientBean` 은 프로토타입 빈을 내부 필드에 보관한다. (정확히는 참조값을 보관한다.)

#### 싱글톤에서 프로토타입 빈 사용2&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 21.03.03.png" alt=""><figcaption></figcaption></figure>

* 클라이언트 A는 `clientBean` 을 스프링 컨테이너에 요청해서 받는다.싱글톤이므로 항상 같은 `clientBean` 이 \
  반환된다.
* 3\. 클라이언트 A는 `clientBean.logic()` 을 호출한다.
* 4\. `clientBean` 은 prototypeBean의 `addCount()` 를 호출해서 프로토타입 빈의 count를 증가한다.
* count값이 1이 된다.

#### 싱글톤에서 프로토타입 빈 사용3&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 21.04.28.png" alt=""><figcaption></figcaption></figure>

* 클라이언트 B는 `clientBean` 을 스프링 컨테이너에 요청해서 받는다. 싱글톤이므로 항상 같은 `clientBean` 이 반환된다.
* **여기서 중요한 점이 있는데, clientBean이 내부에 가지고 있는 프로토타입 빈은 이미 과거에 주입이 끝난 빈이다.** \
  **주입 시점에 스프링 컨테이너에 요청해서 프로토타입 빈이 새로 생성이 된 것이지, 사용 할 때마다 새로 생성되는 것은 아니다.**&#x20;
* 5\. 클라이언트 B는 `clientBean.logic()` 을 호출한다.
* 6\. `clientBean` 은 prototypeBean의 `addCount()` 를 호출해서 프로토타입 빈의 count를 증가한다. 원래 count 값이 1이었으므로 2가 된다

#### 테스트 코드&#x20;

```java
package hello.core.scope;

import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Scope;
import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

import static org.assertj.core.api.Assertions.*;

public class SingletonWithPrototypeTest1 {

    @Test
    void singletonClientUsePrototype() {
        AnnotationConfigApplicationContext ac = new
            AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
        
        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);
        
        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(2);
    }
    
    static class ClientBean {
        
        private final PrototypeBean prototypeBean;
        
        @Autowired
        public ClientBean(PrototypeBean prototypeBean) {
            this.prototypeBean = prototypeBean;
        }
        
        public int logic() {
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }
    
    @Scope("prototype")
    static class PrototypeBean {
        
        private int count = 0;
        
        public void addCount() {
            count++;
        }
        
        public int getCount() {
            return count;
        }
        
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
        
        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

스프링은 일반적으로 싱글톤 빈을 사용하므로, 싱글톤 빈이 프로토타입 빈을 사용하게 된다. 그런데 싱글톤 빈은 생성 시점에만 의존관계를 주입을 받기 때문에, 프로토타입 빈이 새로 생성되기는 하지만, 싱글톤 빈과 함께 계속 유지되는 것이 문제다.&#x20;

아마 원하는 것이 이런 것은 아닐 것이다. 프로토타입 빈을 주입 시점에만 새로 생성하는 것이 아니라, 사용할 때 마다 새로 생성해서 사용하는 것을 원할 것이다.

## 4. 프로토타입 스코프 - 싱글톤 빈과 함께 사용시 Provider 로 문제 해결&#x20;

싱글톤 빈과 프로토타입 빈을 함께 사용할 때, 어떻게 하면 사용할 때 마다 항상 새로운 프로토타입 빈을 생성할 수 있을까?

### 스프링 컨테이너에 요청

가장 간단한 방법은 싱글톤 빈이 프로토타입을 사용할 때 마다 스프링 컨테이너에 새로 요청하는 것이다.&#x20;

```java
public class PrototypeProviderTest {

    @Test
    void providerTest() {
        AnnotationConfigApplicationContext ac = new
            AnnotationConfigApplicationContext(ClientBean.class, PrototypeBean.class);
        
        ClientBean clientBean1 = ac.getBean(ClientBean.class);
        int count1 = clientBean1.logic();
        assertThat(count1).isEqualTo(1);
        
        ClientBean clientBean2 = ac.getBean(ClientBean.class);
        int count2 = clientBean2.logic();
        assertThat(count2).isEqualTo(1);
    }
    
    static class ClientBean {
        
        @Autowired
        private ApplicationContext ac;
        
        public int logic() {
            PrototypeBean prototypeBean = ac.getBean(PrototypeBean.class);
            prototypeBean.addCount();
            int count = prototypeBean.getCount();
            return count;
        }
    }
    
    @Scope("prototype")
    static class PrototypeBean {
    
        private int count = 0;
        
        public void addCount() {
            count++;
        }
        
        public int getCount() {
            return count;
        }
        
        @PostConstruct
        public void init() {
            System.out.println("PrototypeBean.init " + this);
        }
        
        @PreDestroy
        public void destroy() {
            System.out.println("PrototypeBean.destroy");
        }
    }
}
```

* 실행해보면 `ac.getBean()` 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.&#x20;
* 의존관계를 외부에서 주입(DI) 받는게 아니라 이렇게 내부에서 직접 필요한 의존관계를 맺는 것을 Dependency Lookup(DL) 의존관계 조회(탐색) 이라고 한다.&#x20;
* 그런데 이렇게 스프링의 애플리케이션 컨텍스트 전체를 주입받게 되면, 스프링 컨테이너에 종속적인 코드가 되고, 단위 테스트도 어려워진다.&#x20;
* 지금 필요한 기능은 지정한 프로토타입 빈을 컨테이너에서 대신 찾아주는 딱! DL 정도의 기능만 제공하는 무언가 있으면 된다.&#x20;

스프링에는 이미 모든게 준비되어 있다.

> #### DL (Dependency Lookup)이란?
>
> DL은 DI (Dependency Injection, 의존성 주입)와 대조되는 개념입니다.
>
> * **DI (의존성 주입)**: 객체가 자신이 필요한 의존성을 직접 생성하거나 찾아오지 않고, **외부(스프링 컨테이너)에서 의존성을 주입해 주는 방식**입니다. 대부분의 스프링 애플리케이션에서 주된 의존성 관리 방식입니다. (`@Autowired`, 생성자 주입 등)
> * **DL (의존성 조회/탐색)**: 객체가 자신이 필요한 의존성을 **직접 스프링 컨테이너에 요청하여 찾아오는 방식**입니다. `ApplicationContext.getBean()` 메서드를 사용하는 것이 대표적인 DL의 예시입니다.

### ObjectFactory, ObjectProvider

지정한 빈을 컨테이너에서 대신 찾아주는 DL 서비스를 제공하는 것이 바로 `ObjectProvider` 이다. 참고로 과거에는`ObjectFactory` 가 있었는데, 여기에 편의 기능을 추가해서 `ObjectProvider` 가 만들어졌다.

```java
@Autowired
private ObjectProvider<PrototypeBean> prototypeBeanProvider;

public int logic() {
    PrototypeBean prototypeBean = prototypeBeanProvider.getObject();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```

*   실행해보면 `prototypeBeanProvider.getObject()` 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것

    을 확인할 수 있다.
*   `ObjectProvider` 의 `getObject()` 를 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환

    한다. (**DL**)
* 스프링이 제공하는 기능을 사용하지만, 기능이 단순하므로 단위테스트를 만들거나, mock 코드를 만들기는 훨씬 쉬워진다.
* `ObjectProvider` 는 지금 딱 필요한 DL 정도의 기능만 제공한다.

#### 특징&#x20;

* `ObjectFactory`: 기능이 단순, 별도의 라이브러리 필요 없음, 스프링에 의존
* `ObjectProvider`: `ObjectFactory` 상속, 옵션, 스트림 처리등 편의 기능이 많고, 별도의 라이브러리 필요 없음, 스프링에 의존

### JSR-330 Provider&#x20;

마지막 방법은 `javax.inject.Provider` 라는 JSR-330 자바 표준을 사용하는 방법이다.\
**스프링 부트 3.0**은 `jakarta.inject.Provider` 사용한다.

이 방법을 사용하려면 다음 라이브러리를 gradle 에 추가해야 한다.

#### 스프링부트 3.0 미만&#x20;

`javax.inject:javax.inject:1` 라이브러리를 gradle에 추가해야 한다.

#### 스프링부트 3.0 이상

`jakarta.inject:jakarta.inject-api:2.0.1` 라이브러리를 gradle에 추가해야 한다.

```java
@Autowired
private Provider<PrototypeBean> provider;

public int logic() {
    PrototypeBean prototypeBean = provider.get();
    prototypeBean.addCount();
    int count = prototypeBean.getCount();
    return count;
}
```

* 실행해보면 `provider.get()` 을 통해서 항상 새로운 프로토타입 빈이 생성되는 것을 확인할 수 있다.
* `provider` 의 `get()` 을 호출하면 내부에서는 스프링 컨테이너를 통해 해당 빈을 찾아서 반환한다. (**DL**)
* 자바 표준이고, 기능이 단순하므로 단위테스트를 만들거나 mock 코드를 만들기는 훨씬 쉬워진다.
* `Provider` 는 지금 딱 필요한 DL 정도의 기능만 제공한다.

#### 특징&#x20;

* `get()` 메서드 하나로 기능이 매우 단순하다.
* 별도의 라이브러리가 필요하다.
* 자바 표준이므로 스프링이 아닌 다른 컨테이너에서도 사용할 수 있다.

#### 정리&#x20;

* 그러면 프로토타입 빈을 언제 사용할까? 매번 사용할 때 마다 의존관계 주입이 완료된 새로운 객체가 필요하면 사용하면 된다. 그런데 실무에서 웹 애플리케이션을 개발해보면, 싱글톤 빈으로 대부분의 문제를 해결할 수 있기 때문에 프로토타입 빈을 직접적으로 사용하는 일은 매우 드물다.
* `ObjectProvider`, `JSR330`  `Provider` 등은 프로토타입 뿐만 아니라 DL이 필요한 경우는 언제든지 사용할 수 있다.

> #### **참고: '추상화'와 '관심사의 분리'**
>
> 궁극적으로 `ApplicationContext`와 `ObjectProvider`의 차이는 **'추상화 수준'** 과 **'관심사의 분리(Separation of Concerns)'** 에 있다.
>
> * `ApplicationContext`는 스프링 컨테이너의 **모든 것을 담은 '일반 인터페이스'** 이므로, 이에 의존하는 것은 컨테이너의 광범위한 기능에 종속되는 것을 의미한다.&#x20;
> * `ObjectProvider`는 **'특정 타입의 빈을 지연 조회'하는 매우 제한적인 기능만을 위한 '특화된 인터페이스'** 이다. 이에 의존하는 것은 필요한 최소한의 기능에만 종속되는 것을 의미하며, 컨테이너의 다른 복잡한 내부 동작에 대한 지식 없이도 작업을 수행할 수 있게 해준다.&#x20;
>
> 따라서 `ObjectProvider`를 사용하면 코드가 스프링 컨테이너의 광범위한 기능에 덜 묶이게 되고, 특정 빈을 획득하는 '책임'을 더 명확히 분리하여, 코드의 유연성, 테스트 용이성, 그리고 POJO 스러움을 훨씬 더 높일 수 있게 되는 것이다.&#x20;

## 5. 웹 스코프&#x20;

지금까지 싱글톤과 프로토타입 스코프를 학습했다. 싱글톤은 스프링 컨테이너의 시작과 끝까지 함께하는 매우 긴 스코프이고, 프로토타입은 생성과 의존관계 주입, 그리고 초기화까지만 진행하는 특별한 스코프이다.&#x20;

이번에는 웹 스코프에 대해서 알아보자.

#### 웹 스코프&#x20;

* 웹 스코프는 웹 환경에서만 동작한다.&#x20;
* 웹 스코프는 프로토타입과 다르게 스프링이 해당 스코프의 종료시점까지 관리한다. 따라서 종료 메서드가 호출된다.&#x20;

#### 웹 스코프 종류&#x20;

* **request**: HTTP 요청 하나가 들어오고 나갈 때 까지 유지되는 스코프, 각각의 HTTP 요청마다 별도의 빈 인스턴스가 생성되고, 관리된다.
* **session**: HTTP Session과 동일한 생명주기를 가지는 스코프
* **application**: 서블릿 컨텍스트(`ServletContext` )와 동일한 생명주기를 가지는 스코프
* **websocket**: 웹 소켓과 동일한 생명주기를 가지는 스코프

여기서는 request 스코프를 예제로 설명하겠다. 나머지도 범위만 다르지 동작 방식은 비슷하다.

#### HTTP request 요청 당 각각 할당되는 request 스코프

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 22.02.52.png" alt=""><figcaption></figcaption></figure>

## request 스코프 예제 개발

동시에 여러 HTTP 요청이 오면 정확히 어떤 요청이 남긴 로그인지 구분하기 어렵다.\
이럴때 사용하기 딱 좋은것이 바로 request 스코프이다.

#### MyLogger

```java
package hello.core.common;

import jakarta.annotation.PostConstruct;
import jakarta.annotation.PreDestroy;
import org.springframework.context.annotation.Scope;
import org.springframework.context.annotation.ScopedProxyMode;
import org.springframework.stereotype.Component;

import java.util.UUID;

@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {

    private String uuid;
    private String requestURL;

    public void setRequestURL(String requestURL) {
        this.requestURL = requestURL;
    }

    public void log(String message) {
        System.out.println("[" + uuid + "]" + "[" + requestURL + "]" + "[" + message + "]");
    }

    @PostConstruct
    public void init() {
        this.uuid = UUID.randomUUID().toString();
        System.out.println("[" + uuid + "] request scope bean create : " + this);
    }

    @PreDestroy
    public void close() {
        System.out.println("[" + uuid + "] request scope bean close : " + this);
    }
}
```

* `proxyMode = ScopedProxyMode.TARGET_CLASS` 가 없다면, MyLogger 는 HTTP Request 가 들어오는 경우에만 빈이 생성되기 때문에 스프링 컨테이너 기동시 `LogDemoController`, `LogDemoService` 에 의존성을 넣어주는 과정에서 오류가 발생한다. (싱글톤 컨테이너와 request 컨테이너 간 라이프사이클이 달라 발생하는 문제)&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 22.12.26.png" alt=""><figcaption></figcaption></figure>

#### LogDemoController

```java
package hello.core.web;

import hello.core.common.MyLogger;
import jakarta.servlet.http.HttpServletRequest;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@RequiredArgsConstructor
public class LogDemoController {

    private final LogDemoService logDemoService;
    private final MyLogger myLogger;

    @RequestMapping("log-demo")
    @ResponseBody
    public String logDemo(HttpServletRequest request) throws InterruptedException {
        String requestURL = request.getRequestURL().toString();

        System.out.println("myLogger = " + myLogger.getClass());
        myLogger.setRequestURL(requestURL);

        myLogger.log("controller test");
        Thread.sleep(1000);
        logDemoService.logic("testId");
        return "OK";
    }
}
```

> **참고**: requestURL을 MyLogger에 저장하는 부분은 컨트롤러 보다는 **공통 처리가 가능한 스프링 인터셉터나 서블릿 필터 같은 곳을 활용하는 것이 좋다.** 여기서는 예제를 단순화하고, 아직 스프링 인터셉터를 학습하지 않은 분들을 위해서 컨트롤러를 사용했다. 스프링 웹에 익숙하다면 인터셉터를 사용해서 구현해보자.

#### LogDemoService 추가&#x20;

```java
package hello.core.web;

import hello.core.common.MyLogger;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

@Service
@RequiredArgsConstructor
public class LogDemoService {

    private final MyLogger myLogger;

    public void logic(String id) {
        myLogger.log("service id = " + id);
    }
}
```

### 스코프와 프록시&#x20;

```java
@Component
@Scope(value = "request", proxyMode = ScopedProxyMode.TARGET_CLASS)
public class MyLogger {
}
```

* 여기가 핵심이다. `proxyMode = ScopedProxyMode.TARGET_CLASS`를 추가해주자.
  * 적용 대상이 인터페이스가 아닌 클래스면 `TARGET_CLASS` 를 선택
  * 적용 대상이 인터페이스면 `INTERFACES` 를 선택
* 이렇게 하면 MyLogger의 가짜 프록시 클래스를 만들어두고 HTTP request와 상관 없이 가짜 프록시 클래스를\
  다른 빈에 미리 주입해 둘 수 있다.

### 웹 스코프와 프록시 동장 원리

먼저 주입된 myLogger를 확인해보자.

```java
System.out.println("myLogger = " + myLogger.getClass());
```

#### 출력결과

```atom
myLogger = class hello.core.common.MyLogger$$EnhancerBySpringCGLIB$$b68b726d
```

#### **CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.**

* `@Scope` 의 `proxyMode = ScopedProxyMode.TARGET_CLASS)` 를 설정하면 스프링 컨테이너는 CGLIB 라는 바이트코드를 조작하는 라이브러리를 사용해서, MyLogger를 상속받은 가짜 프록시 객체를 생성한다.
* 결과를 확인해보면 우리가 등록한 순수한 MyLogger 클래스가 아니라 `MyLogger$$EnhancerBySpringCGLIB` 이라는 클래스로 만들어진 객체가 대신 등록된 것을 확인할 수 있다.
* 그리고 스프링 컨테이너에 "myLogger"라는 이름으로 진짜 대신에 이 가짜 프록시 객체를 등록한다.
* `ac.getBean("myLogger", MyLogger.class)` 로 조회해도 프록시 객체가 조회되는 것을 확인할 수 있다.
* 그래서 의존관계 주입도 이 가짜 프록시 객체가 주입된다.

<figure><img src="../../.gitbook/assets/스크린샷 2025-05-28 22.15.34.png" alt=""><figcaption></figcaption></figure>

#### 가짜 프록시 객체는 요청이 오면 그때 내부에서 진짜 빈을 요청하는 위임 로직이 들어있다.

* 가짜 프록시 객체는 내부에 진짜 `myLogger` 를 찾는 방법을 알고 있다.
* 클라이언트가 `myLogger.log()` 을 호출하면 사실은 가짜 프록시 객체의 메서드를 호출한 것이다.
* 가짜 프록시 객체는 request 스코프의 진짜 `myLogger.log()` 를 호출한다.
* 가짜 프록시 객체는 원본 클래스를 상속 받아서 만들어졌기 때문에 이 객체를 사용하는 클라이언트 입장에서는 \
  사실 원본인지 아닌지도 모르게, 동일하게 사용할 수 있다(다형성)

#### 동작 정리&#x20;

* CGLIB라는 라이브러리로 내 클래스를 상속 받은 가짜 프록시 객체를 만들어서 주입한다.
* 이 가짜 프록시 객체는 실제 요청이 오면 그때 내부에서 실제 빈을 요청하는 위임 로직이 들어있다.
* 가짜 프록시 객체는 실제 request scope와는 관계가 없다. 그냥 가짜이고, 내부에 단순한 위임 로직만 있고, \
  싱글톤 처럼 동작한다.

#### 특징 정리&#x20;

* 프록시 객체 덕분에 클라이언트는 마치 싱글톤 빈을 사용하듯이 편리하게 request scope를 사용할 수 있다.
*   **사실 Provider를 사용하든, 프록시를 사용하든 핵심 아이디어는 진짜 객체 조회를 꼭 필요한 시점까지 지연처리**

    **한다는 점이다.**
*   단지 애노테이션 설정 변경만으로 원본 객체를 프록시 객체로 대체할 수 있다. 이것이 바로 다형성과 DI 컨테이너

    가 가진 큰 강점이다.
* 꼭 웹 스코프가 아니어도 프록시는 사용할 수 있다.
