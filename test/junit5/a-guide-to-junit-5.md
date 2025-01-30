# A Guide to JUnit 5

> 참고 링크 \
> [https://www.baeldung.com/junit-5](https://www.baeldung.com/junit-5)

## 1. 개요&#x20;

* JUnit 은 Java 생태계에서 가장 인기있는 단위 테스트 프레임워크이다.&#x20;
* Java8 부터 사용이 가능하다.&#x20;
* JUnit5 는 람다 사용을 지원하는 방향으로 개발되었다.&#x20;

## **2. Architecture** <a href="#bd-dependencies-1" id="bd-dependencies-1"></a>

### JUnit Platform

* 이 플랫폼은 JVM에서 테스트 프레임워크를 출시하는 역할을 합니다. JUnit과 클라이언트 간의 안정적이고 강력한 인터페이스(예: 빌드 도구)를 정의합니다.
* 이 플랫폼은 클라이언트를 JUnit과 쉽게 통합하여 테스트를 검색하고 실행합니다.
* [또한 JUnit 플랫폼에서 실행되는 테스트 프레임워크를 개발하기 위한 TestEngine](https://junit.org/junit5/docs/5.0.1/api/org/junit/platform/engine/TestEngine.html) API 도 정의합니다 . 사용자 정의 TestEngine을 구현함으로써 타사 테스트 라이브러리를 JUnit에 직접 연결할 수 있습니다.

### **JUnit Jupiter** <a href="#bd-2-junit-jupiter" id="bd-2-junit-jupiter"></a>

* 이 모듈에는 JUnit 5에서 테스트를 작성하기 위한 새로운 프로그래밍 및 확장 모델이 포함되어 있습니다. JUnit 4와 비교했을 때 새로운 어노테이션은 다음과 같습니다.
  * _@TestFactory_ – 동적 테스트를 위한 테스트 팩토리인 메서드를 나타냅니다.
  * _@DisplayName_ – 테스트 클래스 또는 테스트 메서드에 대한 사용자 정의 표시 이름을 정의합니다.
  * _@Nested_ – 주석이 달린 클래스가 중첩된 비정적 테스트 클래스임을 나타냅니다.
  * _@Tag_ – 필터링 테스트를 위한 태그를 선언합니다.
  * _@ExtendWith_ – 사용자 정의 확장을 등록합니다.
  * _@BeforeEach –_ 주석이 달린 메서드가 각 테스트 메서드 전에 실행됨을 나타냅니다(이전에는 _@Before_ )
  * _@AfterEach – 주석이 달린 메서드가 각 테스트 메서드(이전에는 @After_ ) 이후에 실행됨을 나타냅니다 .
  * _@BeforeAll_ – 주석이 달린 메서드가 현재 클래스의 모든 테스트 메서드보다 먼저 실행됨을 나타냅니다(이전에는 _@BeforeClass_ ).
  * _@AfterAll_ – 주석이 달린 메서드가 현재 클래스의 모든 테스트 메서드 이후에 실행됨을 나타냅니다(이전에는 _@AfterClass_ ).
  * _@Disabled_ – 테스트 클래스 또는 메서드를 비활성화합니다(이전에는 _@Ignore_ )

### **JUnit Vantage**

* JUnit Vintage는 JUnit 5 플랫폼에서 JUnit 3 및 JUnit 4 기반 테스트 실행을 지원합니다.

## 3. 기본 어노테이션

### **@BeforeAll 및 @BeforeEach** <a href="#bd-1-beforeall-and-beforeeach" id="bd-1-beforeall-and-beforeeach"></a>

#### **@BeforeAll**&#x20;

* 클래스 내 전체 테스트 전 한번만 실행&#x20;
* **static 메서드**에서만 사용할 수 있음 (`@BeforeAll`은 인스턴스 생성 없이 실행되기 때문).
* 데이터베이스 연결, 리소스 로딩 등 **비용이 큰 초기화 작업**에 적합함.

> ### @BeforeAll 실행 시 static 메서드만 사용할 수 있는 이유&#x20;
>
> #### JUnit 실행 방식&#x20;
>
> * JUnit 5에서 테스트 클래스는 기본적으로 **각 테스트 메서드 실행 시 새로운 인스턴스를 생성**해.
> * 즉, 테스트 클래스의 객체가 **테스트마다 새롭게 생성**되기 때문에 **객체의 필드는 테스트 간 공유되지 않음**.
>
> #### 그렇다면 문제는?&#x20;
>
> * `@BeforeAll`은 **모든 테스트 전에 한 번만 실행**되어야 하는데, 만약 `@BeforeAll`이 인스턴스 메서드라면, JUnit은 이를 실행할 객체가 필요함.
> * 하지만 기본적으로 **JUnit은 각 테스트 실행마다 새로운 인스턴스를 생성하기 때문에**, 테스트 실행 전에 한 번만 실행될 메서드를 **어떤 인스턴스에서 실행할지 정할 수 없음**.
>
> #### 해결방법 : static 메서드 사용&#x20;
>
> * **`static` 메서드는 클래스 레벨에서 실행**되기 때문에, JUnit이 테스트 클래스의 인스턴스를 만들지 않아도 `@BeforeAll`을 실행할 수 있음.

#### @BeforeEach&#x20;

* **각각의 테스트 메서드가 실행되기 전에 실행됨.**
* static일 필요 없음 (일반 인스턴스 메서드에서 사용 가능).
* **테스트별 독립적인 초기화가 필요할 때 유용**함.

```java
@BeforeAll
static void setup() {
    log.info("@BeforeAll - executes once before all test methods in this class");
}

@BeforeEach
void init() {
    log.info("@BeforeEach - executes before each test method in this class");
}
```

### **@DisplayName 및 @Disabled** <a href="#bd-2-displayname-and-disabled" id="bd-2-displayname-and-disabled"></a>

#### @DisplayName&#x20;

* 테스트 이름 지정&#x20;

#### @Disabled&#x20;

* 테스트 비활성화&#x20;
* 이유 설명 가능&#x20;

```java
@DisplayName("Single test successful")
@Test
void testSingleSuccessTest() {
    log.info("Success");
}

@Test
@Disabled("Not implemented yet")
void testShowSomething() {
}
```

### @AfterAll @AfterEach&#x20;

#### @AfterAll&#x20;

* **테스트 클래스의 모든 테스트가 실행된 후에 한 번만 실행됨.**
* `@BeforeAll`과 마찬가지로 **static 메서드**에서만 사용할 수 있음.
* 데이터베이스 연결 해제, 파일 닫기, 리소스 정리 같은 **전체적인 정리 작업**을 수행할 때 유용.

#### @AfterEach&#x20;

* **각 테스트가 실행된 후마다 실행됨.**
* `static`일 필요 없음 (일반 인스턴스 메서드에서 사용 가능).
* 각 테스트 후 **데이터 초기화**나 **임시 리소스 정리**가 필요할 때 유용.

```java
@AfterEach
void tearDown() {
    log.info("@AfterEach - executed after each test method.");
}

@AfterAll
static void done() {
    log.info("@AfterAll - executed after all test methods.");
}
```

## 4.&#x20;

## 5. 예외 테스트

* JUnit 5에는 예외 테스트 방법이 두 가지 있으며, 둘 다 _assertThrows()_ 메서드를 사용하여 구현할 수 있습니다.
  * 첫 번째 예제에서는 발생한 예외의 세부 정보를 확인하고,&#x20;
  * 두 번째 예제에서는 예외 유형을 검증합니다.

```java
@Test
void shouldThrowException() {
    Throwable exception = assertThrows(UnsupportedOperationException.class, () -> {
      throw new UnsupportedOperationException("Not supported");
    });
    assertEquals("Not supported", exception.getMessage());
}

@Test
void assertThrowsException() {
    String str = null;
    assertThrows(IllegalArgumentException.class, () -> {
      Integer.valueOf(str);
    });
}
```
