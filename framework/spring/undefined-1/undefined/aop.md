# AOP(Aspect Oriented Programming), 관점 지향 프로그래밍

## &#x20;1. AOP 의 개념 및 사용 방법 예제 코드&#x20;

### AOP 란?&#x20;

* 프로그래밍을 하다보면 공통적인 기능이 많이 발생한다. 이러한 공통 기능을 모든 모듈에 적용하기 힘들어 상속을 이용한다.&#x20;
* 하지만 Java 에서는 다중 상속이 불가하며, 상속을 받아 공통 기능을 부여하기에 한계가 있다.&#x20;
  * 예를 들어, 우리가 개발한 API 의 호출 시간을 측정하고 싶다고 하자.&#x20;
  * 이를 AOP 없이 구현한다고 하면&#x20;
    * 중복 코드가 발생할 여지가 있고, 코드의 변경이 필요하면 여러 코드에 종속적으로 변경이 필요할 것이며,&#x20;
    * 핵심적인 비지니스 로직에 호출 시간 측정이라는 부수적인 로직이 추가되어 가독성과 효율성이 떨어지는 등의 문제가 발생할 수 있다.&#x20;
* 이러한 문제점을 해결하기 위해서 AOP 가 등장하게 되었다.
* AOP 는 새로운 프로그래밍 패러다임이 아니라, OOP 를 보조하는 기술로, **핵심적 관리 사항(Core Concern) 과 공통 관심 사항(Cross-Cutting Concer) 으로 분리시키고 각각을 모듈화하는 것을 의미한다.**&#x20;
  * 예를 들어, 우리가 개발한 API 중 회원가입 API 가 있다면, 아래와 같이 분류할 수 있다.&#x20;
    * 핵심적 관리 사항 : 회원가입 비지니스 로직&#x20;
    * 공통 관심 사항 : 호출 시간 측정&#x20;

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt="" width="554"><figcaption></figcaption></figure>

#### AOP 장점&#x20;

* 공통 관심 사항을 핵심 관심사항으로부터 분리시켜 핵심 로직을 깔끔하게 유지할 수 있다.&#x20;
* 그에 따라 코드의 가독성, 유지보수성 등을 높일 수 있다.&#x20;
* 각각의 모듈에 수정이 필요하다면, 다른 모듈의 수정 없이 해당 로직만 변경하면 된다.&#x20;
* 공통 로직을 적용할 대상(메서드 단위)을 선택할 수 있다.&#x20;

### 패키지 기반, 어노테이션 기반으로 AOP 사용해보기&#x20;

#### 의존성 추가 (Gradle 기준)&#x20;

```gradle
implementation 'org.springframework.boot:spring-boot-starter-aop'
```

#### AOP 활성화&#x20;

* AOP 를 사용하고자 하는 빈 클래스에 `@EnableAspectJAutoProxy` 추가

<pre class="language-java"><code class="lang-java"><strong>@EnableAspectJAutoProxy
</strong>@SpringBootApplication
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }

}
</code></pre>

#### 실행 시간 측정을 위한 Aspect 클래스 - 패키지 기반

* AOP 클래스로 설정하기 위해 `@Aspect` 어노테이션 추가&#x20;
* Spring 빈으로 등록하기 위해 `@Component` 어노테이션 추가&#x20;
* `@Around` 를 통해 AOP 가 적용될 범위 지정&#x20;
  * `@Around("execution(* *..controller.*.*(..))")` : 해당 패키지 내 메서드를 지정한 상태&#x20;
* `ProceedingJoinPoint` 는 AOP 에서 메서드 실행 전후의 제어를 가능하게 하는 객체로, `pjp.proceed()` 를 호출하면 원래 대상 메서드가 실행된다.&#x20;

```java
@Aspect
@Component
@Log4j2
public class ExecutionTimeAop {

   // 모든 패키지 내의 controller package에 존재하는 클래스
   @Around("execution(* *..controller.*.*(..))")
   public Object calculateExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
      // 해당 클래스 처리 전의 시간
      StopWatch sw = new StopWatch();
      sw.start();

      // 해당 클래스의 메소드 실행
      Object result = pjp.proceed();

      // 해당 클래스 처리 후의 시간
      sw.stop();
      long executionTime = sw.getTotalTimeMillis();

      String className = pjp.getTarget().getClass().getName();
      String methodName = pjp.getSignature().getName();
      String task = className + "." + methodName;

      log.debug("[ExecutionTime] " + task + "-->" + executionTime + "(ms)");

      return result;
   }
}
```

#### 실행 시간 측정을 위한 Aspect 클래스 - 어노테이션 기반

* AOP 처리를 위한 어노테이션 생성&#x20;
  * `@Target({ElementType.TYPE, ElementType.METHOD})` : 클래스와 메서드가 타겟이다.&#x20;

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
public @interface ExecutionTimeChecker {
}
```

```java
@Aspect
@Component
@Log4j2
public class ExecutionTimeAop {

   @Around("@within(com.mang.atdd.membership.aop.ExecutionTimeChecker)")
   public Object calculateExecutionTime(ProceedingJoinPoint pjp) throws Throwable {
      // 해당 클래스 처리 전의 시간
      StopWatch sw = new StopWatch();
      sw.start();

      // 해당 클래스의 메소드 실행
      Object result = pjp.proceed();

      // 해당 클래스 처리 후의 시간
      sw.stop();
      long executionTime = sw.getTotalTimeMillis();

      String className = pjp.getTarget().getClass().getName();
      String methodName = pjp.getSignature().getName();
      String task = className + "." + methodName;

      log.debug("[ExecutionTime] " + task + "-->" + executionTime + "(ms)");

      return result;
   }
}
```

### 만약 AOP 를 사용하지 않았더라면?&#x20;

* 만약 AOP 를 적용하지 않은 상태에서 실행 시간 측정 단위를 밀리세컨드가 아닌, 세컨드로 변경한다고 하면 어떻게 해야 할까?&#x20;
* AOP 를 적용하지 않았다면, 관련 로직의 모든 코드를 수정해야 했을 것이다.&#x20;
* 하지만 AOP 를 적용함으로, AOP 로직만 수정하면 모든 내용이 적용된다.&#x20;

## 2. 궁금한점? - AOP 와 필터/인터셉터의 차이&#x20;

### &#x20;AOP 와 필터/인터셉터의 차이&#x20;

#### **AOP를 꼭 사용해야 할 경우**

* 로직이 **메서드 호출 자체**에 초점을 맞추고 있을 때.
* HTTP 계층에 의존하지 않거나, 특정 계층(서비스 계층 등)에 강하게 의존할 때.
* 메서드 실행의 제어 및 흐름 변경이 필요할 때.

#### **필터/인터셉터로 충분한 경우**

* 로직이 HTTP 요청과 응답의 전처리/후처리에만 초점이 맞춰져 있을 때.
* URL 기반으로 공통 로직이 적용될 때.

### AOP 를 사용해야만 하는 케이스&#x20;

#### AOP 는 메서드 실행 단위로 동작하기 떄문에, 다음과 같은 상황에서 강력한 도구가 된다.&#x20;

1. 비즈니스 로직과 강하게 결합되지 않으면서, 메서드 호출에 영향을 주고자 할때&#x20;
   1. 서비스 계층의 메서드 실행 전후에 로직을 삽입하는 작업은 AOP 가 적합하다 .
   2. ex) 메서드 실행 시간 측정, 성능 로깅, 트랜잭션 관리&#x20;
2. 다양한 클래스와 메서드에 일관된 처리가 필요할 때&#x20;
   1. 특정 패키지나 클래스 이름 패턴에 따라 로깅 처리&#x20;
   2.  필터와 인터셉터는 컨트롤러 요청에만 적용되므로, 모든 레이어에서 동작이 필요한 경우 AOP 가 필요하다.

       <figure><img src="../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
3. 메서드 실행 흐름 제어가 필요할 때&#x20;
   1. AOP 는 메서드 실행을 중단하거나, 대체하는 것이 가능하다 .
   2.  ex) 특정 조건에서 메서드 실행을 막거나, 메서드 반환 값을 조작 (프록시 패턴?)&#x20;

       <figure><img src="../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>
4. HTTP 요청에 의존하지 않은 로직을  처리할 때&#x20;
   1. AOP 는 HTTP 계층에 의존하지 않으므로, 배치 작업이나 비동기 메서드에서도 적용 가능&#x20;
   2. ex) 스케줄러 기반 메서드 로깅&#x20;

### 필터/인터셉터로 해결 가능한 케이스&#x20;

1. HTTP 요청/응답 전후에만 관심이 있을 때
   1. HTTP 요청 로깅, 인증/인가, 요청 헤더 수정 등은 필터나 인터셉터로 충분히 처리가 가능하다.&#x20;
   2. AOP 는 이러한 작업을 처리하기에 과잉 처리이다.&#x20;
2. 특정 URL 패턴에만 적용해야 할 때&#x20;
   1. 특정 URL 의 요청/응답에만 동작하는 로직이 필요하다면 필터/인터센터가 더 적합&#x20;
   2. AOP 는 URL 개념이 없고 메서드 호출 단위로 동작하기 때문&#x20;
3. 컨트롤러 수준의 전후 처리&#x20;
   1. 컨트롤러 진입 전후에 세션 검사, 사용자 권한 체크 등은 인터셉터가 적합&#x20;
   2. AOP 를 사용하면 컨트롤러 외의 서비스 계층까지 불필요하게 로직이 적용될 수 있다.&#x20;
