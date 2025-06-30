# 스프링 트랜잭션 이해

## 1. 스프링 트랜잭션 소개&#x20;

### 스프링 트랜잭션 추상화&#x20;

각각의 데이터 접근 기술들은 트랜잭션을 처리하는 방식에 차이가 있다. 예를 들어 JDBC 기술과 JPA 기술은 트랜잭션을 사용하는 코드 자체가 다르다.&#x20;

따라서 JDBC 기술을 사용하다가 JPA 기술로 변경하게 되면 트랜잭션을 사용하는 코드도 모두 함께 변경해야 한다.&#x20;

스프링은 이러한 문제를 해결하기 위해서 트랜잭션 추상화를 제공한다. 트랜잭션을 사용하는 입장에서는 스프링 트랜잭션 추상화를 통해 둘을 동일한 방식으로 사용할 수 있게 되는 것이다.&#x20;

스프링은 `PlatformTransactionManager` 라는 인터페이스를 통해서 트랜잭션을 추상화한다.&#x20;

#### PlatformTransactionManager 인터페이스

```java
package org.springframework.transaction;

import org.springframework.lang.Nullable;

public interface PlatformTransactionManager extends TransactionManager {
    TransactionStatus getTransaction(@Nullable TransactionDefinition definition) throws TransactionException;

    void commit(TransactionStatus status) throws TransactionException;

    void rollback(TransactionStatus status) throws TransactionException;
}
```

* 트랜잭션은 트랜잭션 시작(획득), 커밋, 롤백으로 단순하게 추상화 할 수 있다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.25.37.png" alt=""><figcaption></figcaption></figure>

*   스프링은 트랜잭션을 추상화해서 제공할 뿐만 아니라, 실무에서 주로 사용하는 데이터 접근 기술에 대한 트랜잭션

    매니저의 구현체도 제공한다. 우리는 필요한 구현체를 스프링 빈으로 등록하고 주입 받아서 사용하기만 하면 된다.
* 여기에 더해서 스프링 부트는 어떤 데이터 접근 기술을 사용하는지를 자동으로 인식해서 적절한 트랜잭션 매니저를 \
  선택해서 스프링 빈으로 등록해주기 때문에 트랜잭션 매니저를 선택하고 등록하는 과정도 생략할 수 있다.&#x20;
  *   예를 들어서 `JdbcTemplate`, `MyBatis` 를 사용하면

      `DataSourceTransactionManager(JdbcTransactionManager)` 를 스프링 빈으로 등록하고,&#x20;
  * JPA를 사용하면 `JpaTransactionManager` 를 스프링 빈으로 등록해준다.

### 스프링 트랜잭션 사용 방식&#x20;

`PlatformTransactionManager` 를 사용하는 방법은 크게 2가지가 있다.

#### 선언전 트랜잭션 관리 vs 프로그래밍 방식 트랜잭션 관리&#x20;

* 선언적 트랜잭션 관리(Declarative Transaction Management)
  * `@Transactional` 애노테이션 하나만 선언해서 매우 편리하게 트랜잭션을 적용하는 것을 선언적 트랜잭션 \
    관리라 한다.
  * 선언적 트랜잭션 관리는 과거 XML에 설정하기도 했다.
  * 이름 그대로 해당 로직에 트랜잭션을 적용하겠다 라고 어딘가에 선언하기만 하면 트랜잭션이 적용되는 방식이다.
* 프로그래밍 방식의 트랜잭션 관리(programmatic transaction management)
  * 트랜잭션 매니저 또는 트랜잭션 템플릿 등을 사용해서 트랜잭션 관련 코드를 직접 작성하는 것을 프로그래밍 \
    방식의 트랜잭션 관리라 한다.
* 프로그래밍 방식의 트랜잭션 관리를 사용하게 되면, 애플리케이션 코드가 트랜잭션이라는 기술 코드와 강하게 결합된다.
*   선언적 트랜잭션 관리가 프로그래밍 방식에 비해서 훨씬 간편하고 실용적이기 때문에 실무에서는 대부분 선언적

    트랜잭션 관리를 사용한다.

### 선언적 트랜잭션과 AOP

`@Transactional` 을 통한 선언적 트랜잭션 관리 방식을 사용하게 되면 기본적으로 프록시 방식의 AOP 가 적용된다.&#x20;

#### 프록시 도입 전&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.29.05.png" alt="" width="563"><figcaption></figcaption></figure>

트랜잭션을 처리하기 위한 프록시를 도입하기 전에는 서비스의 로직에서 트랜잭션을 직접 시작했다.

```java
// 트랜잭션 시작 
TransactionStatus status = transactionManager.getTransaction(new DefaultTransactionDefinition());

try {
    //비즈니스 로직
    bizLogic(fromId, toId, money);
    transactionManager.commit(status); //성공시 커밋
    
} catch (Exception e) {
    transactionManager.rollback(status); //실패시 롤백
    throw new IllegalStateException(e);
}
```

#### 프록시 도입 후&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.30.32.png" alt="" width="563"><figcaption></figcaption></figure>

트랜잭션을 처리하기 위한 프록시를 적용하면 트랜잭션을 처리하는 객체와 비즈니스 로직을 처리하는 서비스 객체를 명확하게 분리할 수 있다.

```java
public class TransactionProxy {
    private MemberService target;
    
    public void logic() {
        //트랜잭션 시작
        TransactionStatus status = transactionManager.getTransaction(..);
        
        try {
            //실제 대상 호출
            target.logic();
            transactionManager.commit(status); //성공시 커밋
        
        } catch (Exception e) {
            transactionManager.rollback(status); //실패시 롤백
            throw new IllegalStateException(e);
        }
    }
}
```

```java
public class Service {
    public void logic() {
        //트랜잭션 관련 코드 제거, 순수 비즈니스 로직만 남음
        bizLogic(fromId, toId, money);
    }
}
```

* 프록시 도입 전: 서비스에 비즈니스 로직과 트랜잭션 처리 로직이 함께 섞여있다.
* 프록시 도입 후: 트랜잭션 프록시가 트랜잭션 처리 로직을 모두 가져간다. 그리고 트랜잭션을 시작한 후에 실제 서비스를 대신 호출한다. 트랜잭션 프록시 덕분에 서비스 계층에는 순수한 비즈니즈 로직만 남길 수 있다.

### 프록시 도입 후 전체 과정&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.32.38.png" alt=""><figcaption></figcaption></figure>

* 트랜잭션은 커넥션에 `con.setAutocommit(false)` 를 지정하면서 시작한다.
* 같은 트랜잭션을 유지하려면 같은 데이터베이스 커넥션을 사용해야 한다.
* 이것을 위해 스프링 내부에서는 트랜잭션 동기화 매니저가 사용된다.
* `JdbcTemplate` 을 포함한 대부분의 데이터 접근 기술들은 트랜잭션을 유지하기 위해 내부에서 트랜잭션 동기화 매니저를 통해 리소스(커넥션)를 동기화 한다.

### 스프링이 제공하는 트랜잭션 AOP&#x20;

* 스프링의 트랜잭션은 매우 중요한 기능이고, 전세계 누구나 다 사용하는 기능이다. 스프링은 트랜잭션 AOP를 처리하기 위한 모든 기능을 제공한다. 스프링 부트를 사용하면 트랜잭션 AOP를 처리하기 위해 필요한 스프링 빈들도 자동으로 등록해준다.
* 개발자는 트랜잭션 처리가 필요한 곳에 `@Transactional` 애노테이션만 붙여주면 된다. 스프링의 트랜잭션 AOP는 이 애노테이션을 인식해서 트랜잭션을 처리하는 프록시를 적용해준다.

## 2. 트랜잭션 적용 확인&#x20;

### 트랜잭션 적용 확인&#x20;

`@Transactional` 을 통해 선언적 트랜잭션 방식을 사용하면 단순히 애노테이션 하나로 트랜잭션을 적용할 수 있다.\
그런데 이 기능은 트랜잭션 관련 코드가 눈에 보이지 않고, AOP를 기반으로 동작하기 때문에, 실제 트랜잭션이 적용되고 있는지 아닌지를 확인하기가 어렵다.

스프링 트랜잭션이 실제 적용되고 있는지 확인하는 방법을 알아보자.

#### TxApplyBasicTest

```java
package hello.springtx.apply;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.aop.support.AopUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionSynchronizationManager;

import static org.assertj.core.api.Assertions.*;

@Slf4j
@SpringBootTest
public class TxBasicTest {

    @Autowired
    BasicService basicService;

    @Test
    void proxyCheck() {
        log.info("aop class={}", basicService.getClass());
        assertThat(AopUtils.isAopProxy(basicService)).isTrue();
    }

    @Test
    void txTest() {
        basicService.tx();
        basicService.nonTx();
    }

    @TestConfiguration
    static class TxApplyBasicConfig {

        @Bean
        BasicService basicService() {
            return new BasicService();
        }
    }

    static class BasicService {

        @Transactional
        public void tx() {
            log.info("call tx");
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
        }

        public void nonTx() {
            log.info("call nonTx");
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
        }
    }
}
```

#### **proxyCheck() - 실행**

*   `AopUtils.isAopProxy()` : 선언적 트랜잭션 방식에서 스프링 트랜잭션은 AOP를 기반으로 동작한다.

    `@Transactional` 을 메서드나 클래스에 붙이면 해당 객체는 트랜잭션 AOP 적용의 대상이 되고, 결과적으로

    실제 객체 대신에 트랜잭션을 처리해주는 프록시 객체가 스프링 빈에 등록된다. 그리고 주입을 받을 때도 실제 객체 \
    대신에 프록시 객체가 주입된다.
*   클래스 이름을 출력해보면 `basicService$$EnhancerBySpringCGLIB...` 라고 프록시 클래스의 이름이

    출력되는 것을 확인할 수 있다.

#### proxyCheck() - 실행결과&#x20;

```
TxBasicTest : aop class=class ..$BasicService$$EnhancerBySpringCGLIB$$xxxxxx
```

#### 스프링 컨테이너에 트랜잭션 프록시 등록&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.37.50.png" alt=""><figcaption></figcaption></figure>

* `@Transactional` 애노테이션이 특정 클래스나 메서드에 하나라도 있으면 트랜잭션 AOP 는 프록시를 만들어서 스프링 컨테이너에 등록한다. 그리고 실제 `basicService` 객체 대신에 프록시인 `basicService$$CGLIB` 를 스프링 빈에 등록한다. 그리고 프록시는 내부에 실제 `basicService` 를 참조하게 된다. 여기서 핵심은 실제 객체 대신에 프로시가 스프링 컨테이너에 등록되었다는 점이다.&#x20;
* 클라이언트인 `txBasicTest` 는 스프링 컨테이너에 `@Autowired BasicService basicService`로 의존관계 주입을 요청한다. 스프링 컨테이너에는 실제 객체 대신에 프록시가 스프링 빈으로 등록되어 있기 때문에 프록시를 주입한다.
* 프록시는 `BasicService` 를 상속해서 만들어지기 때문에 다형성을 활용할 수 있다. 따라서 `BasicService` 대신에 프록시인 `BasicService$$CGLIB` 를 주입할 수 있다.

#### 트랜잭션 프록시 동작 방식&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.44.27.png" alt=""><figcaption></figcaption></figure>

* 클라이언트가 주입 받은 `basicService$$CGLIB` 는 트랜잭션을 적용하는 프록시이다.

#### 로그 추가&#x20;

`application.properties`

```properties
logging.level.org.springframework.transaction.interceptor=TRACE
```

이 로그를 추가하면 트랜잭션 프록시가 호출하는 트랜잭션의 시작과 종료를 명확하게 로그로 확인할 수 있다.

#### basicService.tx() 호출&#x20;

*   클라이언트가 `basicService.tx()` 를 호출하면, 프록시의 `tx()` 가 호출된다. 여기서 프록시는 `tx()` 메서

    드가 트랜잭션을 사용할 수 있는지 확인해본다. `tx()` 메서드에는 `@Transactional` 이 붙어있으므로 트랜잭션 적용 대상이다.
* 따라서 트랜잭션을 시작한 다음에 실제 `basicService.tx()` 를 호출한다.
* 그리고 실제 `basicService.tx()` 의 호출이 끝나서 프록시로 제어가(리턴) 돌아오면 프록시는 트랜잭션 로직을 커밋하거나 롤백해서 트랜잭션을 종료한다.

#### basicService.nonTx() 호출&#x20;

* 클라이언트가 `basicService.nonTx()` 를 호출하면, 트랜잭션 프록시의 `nonTx()` 가 호출된다. 여기서 `nonTx()` 메서드가 트랜잭션을 사용할 수 있는지 확인해본다. `nonTx()` 에는 `@Transactional` 이 없으므로 \
  적용 대상이 아니다.
* 따라서 트랜잭션을 시작하지 않고, `basicService.nonTx()` 를 호출하고 종료한다.

**TransactionSynchronizationManager.isActualTransactionActive()**

* 현재 쓰레드에 트랜잭션이 적용되어 있는지 확인할 수 있는 기능이다. 결과가 `true` 면 트랜잭션이 적용되어 있는 것이다. 트랜잭션의 적용 여부를 가장 확실하게 확인할 수 있다.

## 3. 트랜잭션 적용 위치&#x20;

스프링에서 우선순위는 항상 **더 구체적이고 자세한 것이 높은 우선순위를 가진다**. 이것만 기억하면 스프링에서 발생하는 \
대부분의 우선순위를 쉽게 기억할 수 있다. 그리고 더 구체적인 것이 더 높은 우선순위를 가지는 것은 상식적으로 자연스럽다.

예를 들어서 메서드와 클래스에 애노테이션을 붙일 수 있다면 더 구체적인 메서드가 더 높은 우선순위를 가진다.\
인터페이스와 해당 인터페이스를 구현한 클래스에 애노테이션을 붙일 수 있다면 더 구체적인 클래스가 더 높은 우선순위를 가진다.&#x20;

#### TxLevelTest

```java
package hello.springtx.apply;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionSynchronizationManager;

@SpringBootTest
public class TxLevelTest {

    @Autowired
    LevelService levelService;

    @Test
    void orderTest() {
        levelService.write();
        levelService.read();
    }

    @TestConfiguration
    static class TxLevelConfig {
        @Bean
        LevelService levelService() {
            return new LevelService();
        }
    }

    @Slf4j
    @Transactional(readOnly = true)
    static class LevelService {

        @Transactional(readOnly = false)
        public void write() {
            log.info("call write");
            printTxInfo();
        }

        public void read() {
            log.info("call read");
            printTxInfo();
        }

        private void printTxInfo() {
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
            boolean readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
            log.info("tx readOnly={}", readOnly);
        }
    }
}
```

스프링의 `@Transactional` 은 다음 두 가지 규칙이 있다.

1. 우선순위 규칙&#x20;
2. 클래스에 적용하면 메서드는 자동 적용

#### 우선 순위&#x20;

트랜잭션을 사용할 때는 다양한 옵션을 사용할 수 있다. 그런데 어떤 경우에는 옵션을 주고, 어떤 경우에는 옵션을 주지 않으면 어떤 것이 선택될까? 예를 들어서 읽기 전용 트랜잭션 옵션을 사용하는 경우와 아닌 경우를 비교해보자. (읽기 전용 옵션에 대한 자세한 내용은 뒤에서 다룬다. 여기서는 적용 순서에 집중하자.)

* `LevelService` 의 타입에 `@Transactional(readOnly = true)` 이 붙어있다.
* `write()` : 해당 메서드에 `@Transactional(readOnly = false)` 이 붙어있다.
  *   이렇게 되면 타입에 있는 `@Transactional(readOnly = true)` 와 해당 메서드에 있는

      `@Transactional(readOnly = false)` 둘 중 하나를 적용해야 한다.
  * 클래스 보다는 메서드가 더 구체적이므로 메서드에 있는 `@Transactional(readOnly = false)` 옵션을 \
    사용한 트랜잭션이 적용된다.

#### **클래스에 적용하면 메서드는 자동 적용**

* `read()` : 해당 메서드에 `@Transactional` 이 없다. 이 경우 더 상위인 클래스를 확인한다.
  * 클래스에 `@Transactional(readOnly = true)` 이 적용되어 있다. 따라서 트랜잭션이 적용되고 \
    `readOnly = true` 옵션을 사용하게 된다.\


### 인터페이스에 @Transactional 적용

인터페이스에도 `@Transactional` 을 적용할 수 있다. 이 경우 다음 순서로 적용된다. 구체적인 것이 더 높은 우선순위를 가진다고 생각하면 바로 이해가 될 것이다.

1. 클래스의 메서드 (우선순위가 가장 높다.)
2. 클래스의 타입
3. 인터페이스의 메서드
4. 인터페이스의 타입 (우선순위가 가장 낮다.)

그런데 인터페이스에 `@Transactional` 사용하는 것은 스프링 공식 메뉴얼에서 권장하지 않는 방법이다. AOP를 적용하는 방식에 따라서 인터페이스에 애노테이션을 두면 AOP가 적용이 되지 않는 경우도 있기 때문이다. 가급적 구체 클래스에 `@Transactional` 을 사용하자.

## 4. 트랜잭션 AOP 주의 사항 - 프록시 내부 호출1

`@Transactional` 을 사용하면 스프링의 트랜잭션 AOP가 적용된다.

트랜잭션 AOP는 기본적으로 프록시 방식의 AOP를 사용한다.\
앞서 배운 것 처럼 `@Transactional` 을 적용하면 프록시 객체가 요청을 먼저 받아서 트랜잭션을 처리하고, 실제 객체를 호출해준다.\
따라서 트랜잭션을 적용하려면 항상 프록시를 통해서 대상 객체(Target)을 호출해야 한다.\
이렇게 해야 프록시에서 먼저 트랜잭션을 적용하고, 이후에 대상 객체를 호출하게 된다.\
만약 프록시를 거치지 않고 대상 객체를 직접 호출하게 되면 AOP가 적용되지 않고, 트랜잭션도 적용되지 않는다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 13.55.16.png" alt=""><figcaption></figcaption></figure>

AOP를 적용하면 스프링은 대상 객체 대신에 프록시를 스프링 빈으로 등록한다. 따라서 스프링은 의존관계 주입시에 항상 실제 객체 대신에 프록시 객체를 주입한다. 프록시 객체가 주입되기 때문에 대상 객체를 직접 호출하는 문제는 일반적으로 발생하지 않는다. \
하지만 **대상 객체의 내부에서 메서드 호출이 발생하면 프록시를 거치지 않고 대상 객체를 직접 호출하는 문제가 발생**한다. 이렇게 되면 `@Transactional` 이 있어도 트랜잭션이 적용되지 않는다. 실무에서 반드시한번은 만나서 고생하는 문제이기 때문에 꼭 이해하고 넘어가자.

#### **InternalCallV1Test**

```java
package hello.springtx.apply;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionSynchronizationManager;

@Slf4j
@SpringBootTest
public class InternalCallV1Test {

    @Autowired
    CallService callService;

    @Test
    void printProxy() {
        log.info("callService class={}", callService.getClass());
    }

    @Test
    void internalCall() {
        callService.internal();
    }

    @Test
    void externalCall() {
        callService.external();
    }

    @TestConfiguration
    static class InternalCallV1Config {

        @Bean
        CallService callService() {
            return new CallService();
        }
    }

    static class CallService {

        public void external() {
            log.info("call external");
            printTxInfo();
            internal();
        }

        @Transactional
        public void internal() {
            log.info("call internal");
            printTxInfo();
        }

        private void printTxInfo() {
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
            boolean readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
            log.info("tx readOnly={}", readOnly);
        }
    }
}
```

#### CallService

* `external()` 은 트랜잭션이 없다.
* `internal()` 은 `@Transactional` 을 통해 트랜잭션을 적용한다.

`@Transactional` 이 하나라도 있으면 트랜잭션 프록시 객체가 만들어진다. 그리고 `callService` 빈을 주입 받으면 트랜잭션 프록시 객체가 대신 주입된다.

#### interCall() 실행&#x20;

**internal()**

```java
@Transactional
public void internal() {
    log.info("call internal");
    printTxInfo();
}
```

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 14.01.29.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트인 테스트 코드는 `callService.internal()` 을 호출한다. 여기서 `callService` 는 트랜잭션 프록시이다.
2. `callService` 의 트랜잭션 프록시가 호출된다.
3. `internal()` 메서드에 `@Transactional` 이 붙어 있으므로 트랜잭션 프록시는 트랜잭션을 적용한다.
4. 트랜잭션 적용 후 실제 `callService` 객체 인스턴스의 `internal()` 을 호출한다.

**실제 `callService` 가 처리를 완료하면 응답이 트랜잭션 프록시로 돌아오고, 트랜잭션 프록시는 트랜잭션을 완료한다.**

#### **externalCall() 실행**

**external()**

```java
public void external() {
    log.info("call external");
    printTxInfo();
    internal();
}
```

`external()` 은 `@Transactional` 애노테이션이 없다. 따라서 트랜잭션 없이 시작한다. 그런데 내부에서`@Transactional` 이 있는 `internal()` 을 호출하는 것을 확인할 수 있다.\
이 경우 `external()` 은 트랜잭션이 없지만, `internal()`에서는 트랜잭션이 적용되는 것 처럼 보인다.

하지만, 실행 로그를 보면 트랜잭션 관련 코드가 전혀 보이지 않는다. 프록시가 아닌 실제 `callService` 에서 남긴 로그만 확인된다. 추가로 `internal()` 내부에서 호출한 `tx active=false` 로그를 통해 확실히 트랜잭션이 수행되지 않은 것을 확인할 수 있다.

우리의 기대와 다르게 `internal()` 에서 트랜잭션이 전혀 적용되지 않았다. 왜 이런 문제가 발생하는 것일까?

#### 프록시 내부 호출&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 14.05.05.png" alt=""><figcaption></figcaption></figure>

1. 클라이언트인 테스트 코드는 `callService.external()` 을 호출한다. 여기서 `callService` 는 트랜잭션 프록시이다.
2. `callService` 의 트랜잭션 프록시가 호출된다.
3. `external()` 메서드에는 `@Transactional` 이 없다. 따라서 트랜잭션 프록시는 트랜잭션을 적용하지 않는다.
4. 트랜잭션 적용하지 않고, 실제 `callService` 객체 인스턴스의 `external()` 을 호출한다.
5. `external()` 은 내부에서 `internal()` 메서드를 호출한다. 그런데 여기서 문제가 발생한다.

#### 문제 원인&#x20;

자바 언어에서 메서드 앞에 별도의 참조가 없으면 `this` 라는 뜻으로 자기 자신의 인스턴스를 가리킨다.\
결과적으로 자기 자신의 내부 메서드를 호출하는 `this.internal()` 이 되는데, 여기서 `this` 는 자기 자신을 가리키므로, 실제 대상 객체(`target`)의 인스턴스를 뜻한다. 결과적으로 이러한 내부 호출은 프록시를 거치지 않는다. 따라서 트랜잭션을 적용할 수 없다. 결과적으로 `target` 에 있는 `internal()` 을 직접 호출하게 된 것이다.

#### **프록시 방식의 AOP 한계**

`@Transactional` 를 사용하는 트랜잭션 AOP는 프록시를 사용한다. 프록시를 사용하면 메서드 내부 호출에 프록시를 적용할 수 없다.

그렇다면 이 문제를 어떻게 해결할 수 있을까?\
가장 단순한 방법은 내부 호출을 피하기 위해 `internal()` 메서드를 별도의 클래스로 분리하는 것이다.

## 5. 트랜잭션 AOP 주의 사항 - 프록시 내부 호출2&#x20;

#### InternalCallV2Test

```java
package hello.springtx.apply;

import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionSynchronizationManager;

@Slf4j
@SpringBootTest
public class InternalCallV2Test {

    @Autowired
    CallService callService;

    @Autowired
    InternalService internalService;

    @Test
    void printProxy() {
        log.info("callService class={}", callService.getClass());
    }

    @Test
    void internalCall() {
        internalService.internal();
    }

    @Test
    void externalCall() {
        callService.external();
    }

    @TestConfiguration
    static class InternalCallV1Config {

        @Bean
        CallService callService() {
            return new CallService(internalService());
        }

        @Bean
        InternalService internalService() {
            return new InternalService();
        }
    }

    @RequiredArgsConstructor
    static class CallService {

        private final InternalService internalService;

        public void external() {
            log.info("call external");
            printTxInfo();
            internalService.internal();
        }

        private void printTxInfo() {
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
            boolean readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
            log.info("tx readOnly={}", readOnly);
        }
    }

    static class InternalService {

        @Transactional
        public void internal() {
            log.info("call internal");
            printTxInfo();
        }

        private void printTxInfo() {
            boolean txActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("tx active={}", txActive);
            boolean readOnly = TransactionSynchronizationManager.isCurrentTransactionReadOnly();
            log.info("tx readOnly={}", readOnly);
        }
    }
}
```

* `InternalService` 클래스를 만들고 `internal()` 메서드를 여기로 옮겼다.
* 이렇게 메서드 내부 호출을 외부 호출로 변경했다.
* `CallService` 에는 트랜잭션 관련 코드가 전혀 없으므로 트랜잭션 프록시가 적용되지 않는다.
* `InternalService` 에는 트랜잭션 관련 코드가 있으므로 트랜잭션 프록시가 적용된다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 14.08.49.png" alt=""><figcaption></figcaption></figure>

**실제 호출되는 흐름을 분석해보자.**

1. 클라이언트인 테스트 코드는 `callService.external()` 을 호출한다.
2. `callService` 는 실제 `callService` 객체 인스턴스이다.
3. `callService` 는 주입 받은 `internalService.internal()` 을 호출한다.
4. `internalService` 는 트랜잭션 프록시이다. `internal()` 메서드에 `@Transactional` 이 붙어 있으므로 \
   트랜잭션 프록시는 트랜잭션을 적용한다.
5. 트랜잭션 적용 후 실제 `internalService` 객체 인스턴스의 `internal()` 을 호출한다.

### public 메서드만 트랜잭션 적용&#x20;

스프링의 트랜잭션 AOP 기능은 `public` 메서드에만 트랜잭션을 적용하도록 기본 설정이 되어있다. 그래서 `protected`, `private`, `package-visible` 에는 트랜잭션이 적용되지 않는다. 생각해보면 `protected`, `package-visible` 도 외부에서 호출이 가능하다. 따라서 이 부분은 앞서 설명한 프록시의 내부 호출과는 무관하고, \
스프링이 막아둔 것이다.

#### 스프링이 `public` 에만 트랜잭션을 적용하는 이유는 다음과 같다.

*   이렇게 클래스 레벨에 트랜잭션을 적용하면 모든 메서드에 트랜잭션이 걸릴 수 있다. 그러면 트랜잭션을 의도하지

    않는 곳 까지 트랜잭션이 과도하게 적용된다.&#x20;
* **트랜잭션은 주로 비즈니스 로직의 시작점에 걸기 때문에 대부분 외부에 열어준 곳을 시작점으로 사용한다.**&#x20;
* 이런 이유로 `public` 메서드에만 트랜잭션을 적용하도록 설정되어 있다.&#x20;

## 6. 트랜잭션 AOP 주의 사항 - 초기화 시점&#x20;

스프링 초기화 시점에는 트랜잭션 AOP가 적용되지 않을 수 있다.

```java
package hello.springtx.apply;

import jakarta.annotation.PostConstruct;
import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.event.EventListener;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.TransactionSynchronizationManager;

@SpringBootTest
public class InitTxTest {

    @Autowired
    Hello hello;

    @Test
    void go() {
        // 스프링은 @PostConstruct 메서드에 트랜잭션을 적용하지 않는다.
        // 초기화 코드는 스프링 초기화 시점에 호출하기 때문에, 직접 호출하지 않아도 된다.
//        hello.initV1();

        // @EventListener(ApplicationReadyEvent.class) 메서드는 스프링이 애플리케이션을 시작할 때 호출된다.
        // 이 메서드는 트랜잭션이 적용되어 있다.
//        hello.initV2();
    }

    @TestConfiguration
    static class InitTxTestConfig {

        @Bean
        Hello hello() {
            return new Hello();
        }
    }

    @Slf4j
    static class Hello {

        @PostConstruct
        @Transactional
        public void initV1() {
            boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("Hello init @PostConstruct tx active={}", isActive);
        }

        @EventListener(ApplicationReadyEvent.class)
        @Transactional
        public void initV2() {
            boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();
            log.info("Hello init @EventListener(ApplicationReadyEvent.class) tx active={}", isActive);
        }
    }
}
```

초기화 코드(예: `@PostConstruct` )와 `@Transactional` 을 함께 사용하면 트랜잭션이 적용되지 않는다.

왜냐하면 초기화 코드가 먼저 호출되고, 그 다음에 트랜잭션 AOP가 적용되기 때문이다. 따라서 초기화 시점에는 해당 메서드에서 트랜잭션을 획득할 수 없다.

가장 확실한 대안은 `ApplicationReadyEvent` 이벤트를 사용하는 것이다.

```java
@EventListener(ApplicationReadyEvent.class)
@Transactional
public void initV2() {
    boolean isActive = TransactionSynchronizationManager.isActualTransactionActive();
    log.info("Hello init @EventListener(ApplicationReadyEvent.class) tx active={}", isActive);
}
```

이 이벤트는 트랜잭션 AOP를 포함한 스프링이 컨테이너가 완전히 생성되고 난 다음에 이벤트가 붙은 메서드를 호출해 준다. 따라서 `init2()` 는 트랜잭션이 적용된 것을 확인할 수 있다.

## 7. 트랜잭션 옵션 소개

...

## 8. 예외와 트랜잭션 커밋, 롤백 - 기본&#x20;

예외가 발생했는데, 내부에서 예외를 처리하지 못하고, 트랜잭션 번위 밖으로 예외가 던져지면 어떻게 될까?&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-06-27 14.17.47.png" alt=""><figcaption></figcaption></figure>

예외 발생시 스프링 트랜잭션 AOP 는 예외의 종류에 따라 트랜잭션을 커밋하거나 롤백한다.&#x20;

* 언체크 예외인 `RuntimeException`, `Error` 와 그 하위 예외가 발생하면 트랜잭션을 롤백한다.
* 체크 예외인 `Exception` 과 그 하위 예외가 발생하면 트랜잭션을 커밋한다.
* 물론 정상 응답(리턴)하면 트랜잭션을 커밋한다.

#### RollbackTest

```java
package hello.springtx.exception;

import lombok.extern.slf4j.Slf4j;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.TestConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.transaction.annotation.Transactional;

import static org.assertj.core.api.Assertions.*;

@Slf4j
@SpringBootTest
public class RollbackTest {

    @Autowired
    RollbackService service;

    @Test
    void runtimeException() {
        assertThatThrownBy(() -> service.runtimeException())
                .isInstanceOf(RuntimeException.class);
    }

    @Test
    void checkedException() {
        assertThatThrownBy(() -> service.checkedException())
                .isInstanceOf(MyException.class);
    }

    @Test
    void checkedExceptionRollbackFor() {
        assertThatThrownBy(() -> service.checkedExceptionRollbackFor())
                .isInstanceOf(MyException.class);
    }

    @TestConfiguration
    static class RollbackTestConfig {

        @Bean
        RollbackService rollbackService() {
            return new RollbackService();
        }
    }

    static class RollbackService {

        // 런타임 예외 발생 : 롤백
        @Transactional
        public void runtimeException() {
            log.info("런타임 예외 발생");
            throw new RuntimeException();
        }

        // 체크 예외 발생 : 커밋
        @Transactional
        public void checkedException() throws MyException {
            log.info("체크 예외 발생");
            throw new MyException();
        }

        // 체크 예외 rollbackFor 지정 : 롤백
        @Transactional(rollbackFor = MyException.class)
        public void checkedExceptionRollbackFor() throws MyException {
            log.info("체크 예외 발생, rollbackFor 지정");
            throw new MyException();
        }

    }

    static class MyException extends Exception {
    }
}
```

실행하기 전에 다음을 추가하자. 이렇게 하면 트랜잭션이 커밋되었는지 롤백 되었는지 로그로 확인할 수 있다.

`application.properties`

```properties
logging.level.org.springframework.transaction.interceptor=TRACE
logging.level.org.springframework.jdbc.datasource.DataSourceTransactionManager=DEBUG
#JPA log
logging.level.org.springframework.orm.jpa.JpaTransactionManager=DEBUG
logging.level.org.hibernate.resource.transaction=DEBUG
```

테스트를 실행시켜보면 다음 기본 전략대로 동작하는 것을 확인할 수 있다.&#x20;

* 언체크 예외인 `RuntimeException`, `Error` 와 그 하위 예외가 발생하면 트랜잭션을 롤백한다.
* 체크 예외인 `Exception` 과 그 하위 예외가 발생하면 트랜잭션을 커밋한다.
* 물론 정상 응답(리턴)하면 트랜잭션을 커밋한다.

## 9. 예외와 트랜잭션 커밋, 롤백 - 활용&#x20;

스프링은 왜 체크 예외는 커밋하고, 언체크(런타임) 예외는 롤백할까?

스프링 기본적으로 **체크 예외는 비즈니스 의미가 있을 때 사용하고, 런타임(언체크) 예외는 복구 불가능한 예외로 가정**한다.

* 체크 예외 : 비지니스 의미가 있을 때 사용&#x20;
* 언체크 예외 : 복구 불가능한 예외&#x20;

참고로 꼭 이런 정책을 따를 필요는 없다. 그때는 앞서 배운 `rollbackFor` 라는 옵션을 사용해서 체크 예외도 롤백하면 된다.

그런데 비즈니스 의미가 있는 **비즈니스 예외**라는 것이 무슨 뜻일까? 간단한 예제로 알아보자.

#### 비즈니스 요구사항&#x20;

주문을 하는데 상황에 따라 다음과 같이 조치한다.&#x20;

1. **정상**: 주문시 결제를 성공하면 주문 데이터를 저장하고 결제 상태를 `완료`로 처리한다.
2. **시스템 예외**: 주문시 내부에 복구 불가능한 예외가 발생하면 전체 데이터를 롤백한다.
3. **비즈니스 예외**: 주문시 결제 잔고가 부족하면 주문 데이터를 저장하고, 결제 상태를 `대기` 이 경우 **고객에게 잔고 부족을 알리고 별도의 계좌로 입금하도록 안내한다.**

이때 결제 잔고가 부족하면 `NotEnoughMoneyException` 이라는 체크 예외가 발생한다고 가정하겠다. 이 예외는 시스템에 문제가 있어서 발생하는 시스템 예외가 아니다. 시스템은 정상 동작했지만, 비즈니스 상황에서 문제가 되기 때문에 발생한 예외이다. 더 자세히 설명하자면, 고객의 잔고가 부족한 것은 시스템에 문제가 있는 것이 아니다. 오히려 시스템은 문제 없이 동작한 것이고, 비즈니스 상황이 예외인 것이다. 이런 예외를 비즈니스 예외라 한다. 그리고 비즈니스 예외는 매우 중요하고, 반드시 처리해야 하는 경우가 많으므로 체크 예외를 고려할 수 있다.

~~코드는 패스 ..~~&#x20;

#### 정리&#x20;

* `NotEnoughMoneyException` 은 시스템에 문제가 발생한 것이 아니라, 비즈니스 문제 상황을 예외를 통해 알려준다. 마치 예외가 리턴 값 처럼 사용된다. 따라서 이 경우에는 트랜잭션을 커밋하는 것이 맞다. 이 경우 롤백하면 생성한 `Order` 자체가 사라진다. 그러면 고객에게 잔고 부족을 알리고 별도의 계좌로 입금하도록 안내해도 주문(`Order`) 자체가 사라지기 때문에 문제가 된다.
* 그런데 비즈니스 상황에 따라 체크 예외의 경우에도 트랜잭션을 커밋하지 않고, 롤백하고 싶을 수 있다. 이때는 `rollbackFor` 옵션을 사용하면 된다.
* 런타임 예외는 항상 롤백된다. 체크 예외의 경우 `rollbackFor` 옵션을 사용해서 비즈니스 상황에 따라서 커밋과 \
  롤백을 선택하면 된다.
