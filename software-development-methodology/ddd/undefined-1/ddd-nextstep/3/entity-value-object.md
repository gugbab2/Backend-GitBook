# 전술적 설계 - VALUE OBJECT 와 ENTITY

* 전술적 설계 - 전략적 설계(모델링) 을 통해서 도출한 결과물을 코드로 변경하는데 필요한 전술적 도구들
* 90년대 OOP -> 2000년대 DDD, 때문에 DDD 는 OOP 의 영향을 많이 받았다.&#x20;
  * 객체는 상태와 행위를 가진다.&#x20;
  * 비지니스 로직을 수행하기 위해서는 객체의 행위를 통해서 이루어져야 한다.&#x20;
  * 풍푸한 도메인 로직을 가져야 한다.&#x20;
    * 풍부한 서비스로직은 지양해야 한다.&#x20;

## VALUE OBJECT&#x20;

* 의미를 명확하게 표현하거나 두 개 이상의 데이터가 개념적으로 하나인 경우 이용
  * 정말 `String` 타입으로 우편 번호를 표현할 수 있는가? -> YES!&#x20;
  * `String` 타입은 우편 번호인가? -> NO..
  * **처음에는 동작하지 않고, 시스템이 성숙해짐에 따라서 기본형 타입을 객체로 대체하게 된다.**
* 값 객체 타입은 불변
  * **모든 객체는 일단 불변이 좋다고 생각하자.**&#x20;
  * **시작부터 불변으로 객체를 만들고, 필요한 경우 가변으로 바꾸는 것으로 생각하자.**&#x20;
  * 서버 자원을 생각하면서 개발하는 시대는 지났다.

### 예시, Car 의 CarName

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-02-16 08.59.43.png" alt=""><figcaption></figcaption></figure>

* 위 코드에서 `Car` 객체를 생성할 때 이름에 대한 제약 조건, `Car` 객체의 이름을 변경할 때 제약조건이 겹친다. \
  (코드 중복이 발생한다)&#x20;
* 이를 해결하기 위한 좋은 방법은 공통된 로직에 대한 메서드를 분리하는 방법도 있지만, **해당 필드에 대한 `CarName` 값 객체를 만들어 해당 객체를 사용하면 무조건적으로 `CarName` 객체를 사용하면 5글자를 넘지 않겠다는 확신을 줄 수 있다. (심리적 안정감)**
* **또한, `CarName` 객체를 통해서 자동차 이름은 5글자를 넘어설 수 없다는 도메인 로직을 반영할 수 있다.**

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-02-16 09.03.11.png" alt=""><figcaption></figcaption></figure>

### VALUE OBJECT 는 식별자가 없다? (애매혀;;)

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-02-11 21.46.17.png" alt=""><figcaption></figcaption></figure>

* 대부분의 책에서는 ENTITY 는 식별자가 있고, VALUE OBJECT 는 식별자가 없다고 한다.&#x20;
* 하지만 제이슨은 VALUE OBJECT 도 식별자가 있다고 생각한다.&#x20;
  * 위 코드에서 **`new Name("Jason", "Park")` 자체로 식별자(값의 동등성, 복합키)로 생각한다.**&#x20;
* 항상 `equals()`, `hashCode()` 메서드를 오버라이드 할 것을 권고한다.
  * VALUE OBJECT 는 기본적으로 값의 모음 자체가 식별자의 역할을 하기 때문에 `equals()`, `hashCode()` 를 오버라이딩 해주어야 한다.&#x20;

### 불변 객체&#x20;

* 모든 클래스를 상태를 변경할 수 없는 불변 클래스(immutable class) 로 만들면 유지 보수성이 크게 향상된다.&#x20;

```java
public class Cash {
    private final int dollars;

    public Cash mul(final int factor) {
        return new Cash(this.dollars * factor);
    }
}
```

```java
Cash five = new Cash(5);
Cash fifty = five.mul(10);
System.out.println(fifty);
```

* 불변 객체에는 아래의 식별자 변경(identity mutability) 문제가 발생하지 않는다.

```java
Map<Cash, String> map = new HashMap<>();
Cash five = new Cash("$5");
Cash ten = new Cash("$10");
map.put(five, "five");
map.put(ten, "ten");    // 새로운 객체 반환(원본 five 는 변경되지 않음) 
five.mul(2);            // 원본 five 는 그대로 유지됨
```

```java
map.get(five);
```

#### 불변 객체 장점

1. 불변객체는 객체가 완전하고 상태가 아니면 아예 실패하는 실패 원자성(failure atomicity) 을 가진다.&#x20;

```java
// 일반 객체 : 실패 원자성이 깨지는 예제 
public class Cash {
    private int dollars;
    private int cents;

    public mul(final int factor) {
        this.dollars *= factor;
    
        // 예외가 발생하면 dallars 의 값는 변경되었는데, cents 의 가격은 그대로이다. 
        if (/* 뭔가 잘못 됐다면 */) {
            throw new RuntimeException("oops...");
        }
        
        this.cents *= factor;
    }
);
```

```java
// 불변 객체 : 실패 원자성을 가지는 예제
public class Cash {
    private final int dollars;
    private final int cents;

    public Cash(int dollars, int cents) {
        this.dollars = dollars;
        this.cents = cents;
    }

    public Cash mul(final int factor) {
        int newDollars = this.dollars * factor;
        
        // 예외가 발생하더라도 속성이 final 로 선언되어 있기 때문에, 실패 원자성을 가진다. 
        if (/* 뭔가 잘못 됐다면 */) {
            throw new RuntimeException("oops...");
        }

        int newCents = this.cents * factor;

        return new Cash(newDollars, newCents);  // 새로운 객체 반환
    }
}
```

2. 불변 객체는 시간적 결합(temporal coupling) 을 없앨 수 있다.&#x20;

```java
// 일반 객체 : 시간적 결합을 가지게 된다. 
Cash price = new Cash();
price.setDollars(29);
System.out.println(price);  // "$29.00"!
price.setCents(95);
```

<pre class="language-java"><code class="lang-java"><strong>// 불변 객체 : 객체 생성 시 속성값을 모두 가져야하기 때문에, 시간적 결합을 없앤다. 
</strong><strong>Cash price = new Cash(29, 95);
</strong>System.out.println(price);  // "$29.95"
</code></pre>

3. 스레드 안정성&#x20;
   1. 객체가 여러 스레드에서 동시에(comcurrently) 사용될 수 있고 예측 가능한(predictable) 결과를 보장하는 객체의 품질
4. 단순성(simplicity), 객체가 더 단순해질 수록 응집도는 높아지고, 유지보수는 쉬워진다.&#x20;
5. 불변 객체의 크기가 작은 이유는 불변 객체의 경우 생성자 안에서만 상태를 초기화할 수 있기 때문이다.&#x20;

### 그렇다면 언제 가변 객체(ENTITY)를 사용하는가?&#x20;

* **어떤 객체의 값의 변경을 추적해야 하는 경우가 있을 때 사용한다.**
  * ex1, 내가 주소지를 변경해도 내가 변경되지는 않는 것이기 때문에 식별자가 필요하다. (주민등록번호)&#x20;
  * ex2, 지폐는 지폐를 추적(위조지폐 여부 확인을 위해?)하기 위해서 식별자가 필요하다. (발생번호, 일련번호)&#x20;
* 이런 경우 VALUE OBJECT 를 ENTITY 로 변경한다.&#x20;
  * **관점(컨텍스트) 에 따라서 ENTITY 변경이 필요 없을수도 있다.**

## ENTITY&#x20;

* 식별자를 갖는다.&#x20;
  * 객체의 상태 중 해당 객체의 고유한 성질을 표현할 수 있는 상태들을 식별자라고 부른다 .
* 식별자는 엔티티 객체마다 고유해서 각 엔티티는 서로 다른 식별자를 갖는다.&#x20;
  * 특정 규칙에 따라 생성 (ex, 날짜 + 일련번호) &#x20;
  * UUID 사용&#x20;
  * 값을 직접 입력&#x20;
  * 일련번호 사용

### ENTITY 에 대한 오해&#x20;

* 관점에 따라서 ENTITY 에 의미가 달라진다.
  * 예시1, JPA - Persistence Entity
  * 예시2, DDD - 식별자를 갖는(가변) 객체를 Entity 라고 부른다.

### 도메인 모델에 set 메서드 넣지 않기&#x20;

* 도메인 모델에 getter 메서드와 setter 메서드를 무조건 추가하는 것은 좋지 않은 버릇
* 특히 setter 메서드는 도메인의 핵심 개념이나 의도를 코드에서 사라지게 한다.
* setter 메서드의 또 다른 문제는 도메인 객체를 생성할 때 완전한 상태가 아닐 수도 있다는 것이다.
* 도메인 객체가 불완전한 상태로 사용되는 것을 막으려면 생성 시점에 필요한 것을 전달해 주어야 한다.

```java
changeShippingInfo() vs setShippingInfo()
completePayment() vs setOrderState()
```
