# Optional

## 옵셔널이 필요한 이유&#x20;

#### NullPointException 문제&#x20;

* 자바에서 `null` 은 "값이 없음" 을 표현하는 가장 기본적인 방법이다.&#x20;
* 하지만 `null` 을 잘못 사용하거나 `null` 참조에 대해 메서드르 호출하면 `NullPointException` 이 발생하여 프로그램이 예기치 않게 종료될 수 있다.&#x20;
* 특히 여러 메서드가 연쇄적으로 호출되어 내부에서 `null` 체크가 누락되면, 추적하기 어렵고 디버깅 비용이 증가한다.&#x20;

#### 가독성 저하&#x20;

* `null` 을 반환하거나 사용하게 되면, 코드를 작성할 때마다 조건문으로 `null` 체크를 해야만 한다.&#x20;
* 이러한 `null` 체크 로직이 누적되면 코드가 복잡해지고 가독성이 떨어진다.&#x20;

#### 의도가 드러나지 않음&#x20;

* 메서드 시그니처만 보고서는 이 메서드가 `null` 을 반환할 수 있다는 사실을 명백하게 알기 어렵다.&#x20;
* 호출하는 입장에서는 "반드시 값이 존재할 것" 이라고 가정했다가 런타임에 `null` 이 나와서 문제가 발생할 수 있다.&#x20;

#### Optional의 등장&#x20;

* 이러한 문제를 해결하고자 자바 8부터 `Optional` 클래스를 도입했다.&#x20;
* `Optional` 은 "값이 있을 수도 있고, 없을 수도 있음" 을 명시적으로 표현해주어, 메서드의 호출 의도를 좀 더 분명하게 드러낸다.&#x20;
* `Optional` 을 사용하면 "빈 값" 을 표현할 때, 더 이상 `null` 자체를 넘겨주지 않고 `Optional.empty()` 처럼 의도를 드러내는 객체를 사용할 수 있다.
* 그 결과, `Optional` 을 사용하면 `null` 체크 로직을 간결하게 만들고, 특정 경우에 `NullPointException` 이 발생할 수 있는 부분을 빌드 타임이나, IDE, 코드 리뷰에서 더 쉽게 파악할 수 있게 해준다.&#x20;

### null 을 직접 반환하는 경우

```java
package optional;

import java.util.HashMap;
import java.util.Map;

public class OptionalStartMain1 {

    private static final Map<Long, String> map = new HashMap<>();
    
    static {
        map.put(1L, "Kim");
        map.put(2L, "Seo");
    }
    
    public static void main(String[] args) {
        findAndPrint(1L);   // 값이 있는 경우
        findAndPrint(3L);   // 값이 없는 경우
    }
    
    // 이름이 있으면 이름을 대문자로 출력, 없으면 "UNKNOWN" 을 출력 
    static void findAndPrint(Long id) {
        String name = findNameById(id);
        // 1. NullPointException
        // System.out.println("name.toUpperCase() = " + name.toUpperCase());

        // 2. if 문을 활용한 null 체크 필요
        if (name != null) {
            System.out.println(id + ": " + name.toUpperCase());
        } else {
            System.out.println(id + ": " + "UNKNOWN");
        }
    }

    static String findNameById(Long id) {
        return map.get(id);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.21.28.png" alt=""><figcaption></figcaption></figure>

* 이 예제에서는 `map.get(id)` 결과가 존재하지 않을 경우 `null` 을 반환한다.&#x20;
* 따라서 `findAndPoint()` 메서드에서 `name` 이 `null` 이 아닌지 매번 확인해야 한다.&#x20;
* `null` 을 반환하는 방식을 채택하면, 메서드를 호출하는 쪽에서 `null` 체크 로직을 작성해주어야되고, 이를 빠뜨리면 `NullPointException` 이 발생한다.
* 반환 타입이 `String` 이므로 얼핏 보면 항상 문자열이 반환될 것처럼 보이지만, 실제로는 `null` 이 반환될 수 있어 주의가 필요하다.&#x20;

### Optional 을 반환하는 경우&#x20;

```java
package optional;

import java.util.HashMap;
import java.util.Map;
import java.util.Optional;

public class OptionalStartMain2 {

    private static final Map<Long, String> map = new HashMap<>();
    
    static {
        map.put(1L, "Kim");
        map.put(2L, "Seo");
    }
    
    public static void main(String[] args) {
        findAndPrint(1L);   // 값이 있는 경우
        findAndPrint(3L);   // 값이 없는 경우
    }
    
    // 이름이 있으면 이름을 대문자로 출력, 없으면 "UNKNOWN" 을 출력 
    static void findAndPrint(Long id) {
        Optional<String> optName = findNameById(id);
        String name = optName.orElse("UNKNOWN");
        System.out.println(id + ": " + name.toUpperCase());
    }

    static Optional<String> findNameById(Long id) {
        String findName = map.get(id);
        Optional<String> optName = Optional.ofNullable(findName);
        return optName;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.23.53.png" alt=""><figcaption></figcaption></figure>

* 이번에는 `Optional<String>` 을 반환하도록 변경했다.&#x20;
* `Optional.ofNullable(findName)` 을 통해 `null` 이 될 수도 있는 값을 `Optional` 로 감싼다.
*   메서드 시그니처(`Optional<String> findNameById(Long id)` )만 보고도 "반환 결과가 있을 수도, 없

    을 수도 있겠구나"라고 명시적으로 인지할 수 있다.
* `Optional.orElse("대체값")` : 옵셔널에 값이 있으면 해당 값을 반환하고, 값이 없다면 대체값을 반환한다.
* `findAndPrint()` 메서드에서는 `Optional<String>` 을 받아서, `orElse("UNKNOWN")` 로 "빈 값"인 경우 대체 문자열을 지정할 수 있다.
*   이 방식은 "값이 없을 수도 있다"는 점을 호출하는 측에 명확히 전달하므로, 놓치기 쉬운 `null` 체크를 강제하고

    코드의 안정성을 높인다.

### Optional 소개&#x20;

#### 자바 Optional 클래스 코드&#x20;

```java
package java.util;

public final class Optional<T> {
    private final T value;
    ...
}
```

#### 정의

*   `java.util.Optional<T>` 는 "존재할 수도 있고 존재하지 않을 수도 있는" 값을 감싸는 일종의 컨테이너 클래

    스이다.
* 내부적으로 `null` 을 직접 다루는 대신, `Optional` 객체에 감싸서 `Optional.empty()` 또는`Optional.of(value)` 형태로 다룬다.

#### 등장 배경&#x20;

* "값이 없을 수 있다" 는 상황을 프로그래머가 명시적으로 처리하도록 유도하고, 런타임 `NullPointException` 을 예방하기 위해서 도입되었다.&#x20;
*   코드를 보는 사람이나 협업하는 팀원 모두가, 해당 메서드의 반환값이 비어있을 수도 있음을 알 수 있게 되어 오류

    를 줄일 수 있다.

#### 참고&#x20;

* `Optional` 은 "값이 없을 수도 있다"는 상황을 반환할 때 주로 사용된다.
*   "항상 값이 있어야 하는 상황"에서는 `Optional` 을 사용할 필요 없이 그냥 해당 타입을 바로 사용하거나 예외를

    던지는 방식이 더 좋을 수 있다.

## Optional 의 생성과 값 획득

### Optional 생성&#x20;

#### Optional 을 생성하는 메서드&#x20;

* `Optional.of(T value)`&#x20;
  * 내부 값이 확실히 `null` 이 아닐 때 사용. `null` 을 전달하면 `NullPointExcpetion` 발생&#x20;
* `Optional.ofNullable(T value)`
  * 값이 `null` 일 수도 있고 아닐 수도 있을 때 사용. `null` 이면 `Optional.empty()` 를 반환한다.
* `Optional.empty()`
  * 명시적으로 "값이 없음" 을 표현할 때 사용&#x20;

```java
package optional;

import java.util.Optional;

public class OptionalCreationMain {

    public static void main(String[] args) {
        // 1. of() : 값이 null 이 아님이 확실할 때 사용, null 이면 NullPointException 이 발생
        String nonNullValue = "Hello, Optional";
        Optional<String> opt1 = Optional.of(nonNullValue);
        System.out.println("opt1 = " + opt1);

        // 2. ofNullable() : 값이 null 일 수도 아닐 수도 있을 때
        Optional<String> opt2 = Optional.ofNullable("Hello, Optional");
        Optional<String> opt3 = Optional.ofNullable(null);

        System.out.println("opt2 = " + opt2);
        System.out.println("opt3 = " + opt3);
        
        // 3. empty() : 비어있는 Optional을 명시적으로 생성
        Optional<Object> opt4 = Optional.empty();
        System.out.println("empty = " + opt4);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.32.58.png" alt=""><figcaption></figcaption></figure>

### Optional 값 획득&#x20;

#### Optional 의 값을 확인하거나, 획득하는 메서드&#x20;

1. &#x20;`isPresent()`, `isEmpty()`&#x20;
   1. 값이 있으면 `true`&#x20;
   2. 값이 없으면 `false` 를 반환. 간단 확인용.
   3. `isEmpty()` : 자바 11 이상에서 사용 가능, 값이 비어있으면 `true`, 값이 있으면 `false` 를 반환
2. &#x20;`get()`&#x20;
   1. 값이 있는 경우 그 값을 반환
   2. 값이 없으면 `NoSuchElementException` 발생.
   3. 직접 사용 시 주의해야 하며, 가급적이면 `orElse`, `orElseXxx` 계열 메서드를 사용하는 것이 안전.
3. &#x20;`orElse(T other)`
   1. 값이 있으면 그 값을 반환
   2. 값이 없으면 `other` 를 반환.
4. &#x20;`orElseGet(Supplier<? extends T> supplier)`&#x20;
   1. 값이 있으면 그 값을 반환
   2. 값이 없으면 `supplier` 호출하여 생성된 값을 반환.
5. &#x20;`orElseThrow(...)`
   1. 값이 있으면 그 값을 반환
   2. 값이 없으면 지정한 예외를 던짐.
6. &#x20;`or(Supplier<? extends Optional<? extends T>> supplier)`&#x20;
   1. 값이 있으면 해당 값의 `Optional` 을 그대로 반환
   2. 값이 없으면 `supplier` 가 제공하는 다른 `Optional` 반환
   3. **값 대신 `Optional` 을 반환한다는 특징**

```java
package optional;

import javax.swing.text.html.Option;
import java.util.Optional;

public class OptionalRetrievalMain {

    public static void main(String[] args) {
        // 예제 : 문자열 "Java" 가 있는 Optional 과 비어있는 Optional 준비
        Optional<String> optVale = Optional.of("Hello");
        Optional<String> optEmpty = Optional.empty();

        // isPresent() : 값이 있으면 ture
        System.out.println("===  1. isPresent() / isEmpty() ===");
        System.out.println("optVale.isPresent() = " + optVale.isPresent());
        System.out.println("optEmpty.isPresent() = " + optEmpty.isPresent());
        System.out.println("optEmpty.isEmpty() = " + optEmpty.isEmpty());
        
        // get() : 직접 내부 값을 꺼냄, 값이 없으면 예외(NoSuchElementException) 
        System.out.println("===  2. get() ===");
        String getValue = optVale.get();
        System.out.println("getValue = " + getValue);
//        String getValue2 = optEmpty.get();  // 런타임 에러

        // 값이 있느면 그 값, 없으면 지정된 기본값 사용
        System.out.println("===  3. orElse() ===");
        String value1 = optVale.orElse("기본값");
        String empty1 = optEmpty.orElse("기본값");
        System.out.println("value1 = " + value1);
        System.out.println("empty1 = " + empty1);

        // 값이 없을 때만 람다(Supplier) 가 실행되어 기본값 생성
        System.out.println("=== 4. orElseGet() ===");
        String value2 = optVale.orElseGet(() -> {
            System.out.println("람다 호출 - optValue");
            return "New Value";
        });
        String empty2 = optEmpty.orElseGet(() -> {
            System.out.println("람다 호출 - optEmpty");
            return "New Value";
        });
        System.out.println("value2 = " + value2);
        System.out.println("empty2 = " + empty2);

        // 값이 있으면 반환, 없으면 예외 발생
        System.out.println("===  5. orElseThrow() ===");
        String value3 = optVale.orElseThrow(() -> new RuntimeException("값이 없습니다."));
        System.out.println("value3 = " + value3);

        try {
            String empty3 = optEmpty.orElseThrow(() -> new RuntimeException("값이 없습니다."));
            System.out.println("empty3 = " + empty3);   // 실행되지 않는다.
        } catch (Exception e) {
            System.out.println("예외 발생 : " + e.getMessage());
        }
        
        // Optional 을 반환 
        System.out.println("=== 6. or() ===");
        Optional<String> result1 = optVale.or(() -> Optional.of("Fallback"));
        System.out.println(result1);

        Optional<String> result2 = optEmpty.or(() -> Optional.of("Fallback"));
        System.out.println(result2);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.36.40.png" alt="" width="375"><figcaption></figcaption></figure>

* `isEmpty()` 는 자바 11 부터 사용할 수 있는 메서드로, 값이 없으면 `true` 를 반환한다.&#x20;
  * `isPresent()` 와 반대
* `get()` 메서드는 `Optional` 사용 시 가능하면 사용을 피해야 한다.&#x20;
  * 값이 없는 상태(`Optional.empty()` 에서 `get()` 을 호출하면 바로 예외가 터지므로, 안전하게 사용하려면 `isPresent()` 같은 사전 체크가 필요하다.&#x20;
  * `get()` 보다는 `orElse()`, `orElseGet()`, `orElseThrow()` 등의 메서드를 활용하면 좀 더 세련되고 안전하게 값을 처리할 수 있다.&#x20;

## Optional 값 처리&#x20;

#### Optional 값 처리 메서드&#x20;

* `ifPresent(Consumer<? super T> action)`&#x20;
  * 값이 존재하면 action 실행
  * 값이 없으면 아무것도 안 함
* `ifPresentOrElse(Consumer<? super T> action, Runnable emptyAction)`&#x20;
  * 값이 존재하면 action 실행
  * 값이 없으면 emptyAction 실행
* `map(Function<? super T, ? extends U> mapper)`&#x20;
  * 값이 있으면 `mapper` 를 적용한 결과`(Optional<U>)` 반환
  * 값이 없으면 `Optional.empty()` 반환
* `flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)`&#x20;
  * map과 유사하지만, `Optional` 을 반환할 때 중첩되지 않고 평탄화(flat)해서 반환
* `filter(Predicate<? super T> predicate)`&#x20;
  * 값이 있고 조건을 만족하면 그대로 반환,
  * 조건 불만족이거나 비어있으면 `Optional.empty()` 반환
* `stream()`
* 값이 있으면 단일 요소를 담은 `Stream<T>` 반환
* 값이 없으면 빈 스트림 반환

```java
package optional;

import java.util.Optional;

public class OptionalProcessingMain {

    public static void main(String[] args) {
        Optional<String> optValue = Optional.of("Hello");
        Optional<String> optEmpty = Optional.empty();

        // 값이 존재하면 Consumer 실행, 없으면 아무 일도 하지 않음
        System.out.println("=== 1. ifPresent() ===");
        optValue.ifPresent(v -> System.out.println("optValue 값 : " + v));
        optEmpty.ifPresent(v -> System.out.println("optEmpty 값 : " + v));   // 실행 안됨!

        // 값이 있으면 Consumer 실행, 없으면 Runnable 실행
        System.out.println("=== 2. ifPresentOrElse() ===");
        optValue.ifPresentOrElse(
                v -> System.out.println("optValue 값 : " + v),
                () -> System.out.println("optValue 는 비어있음")
        );
        optEmpty.ifPresentOrElse(
                v -> System.out.println("optEmpty 값 : " + v),
                () -> System.out.println("optEmpty 는 비어있음")
        );

        // 값이 있으면 Function 적용 후 Optional 로 반환, 없으면 Optional.empty()
        System.out.println("=== 3. map() ===");
        Optional<Integer> lengthOpt1 = optValue.map(String::length);
        System.out.println("optValue.map(String::length = " + lengthOpt1);
        
        Optional<Integer> lengthOpt2 = optEmpty.map(String::length);
        System.out.println("optEmpty.map(String::length = " + lengthOpt2);

        // map() 과 유사하나, 이미 Optional 을 반환하는 경우 중첩을 제거
        System.out.println("=== 4. flatMap() ===");
        System.out.println("[map]");
        Optional<Optional<String>> nestedOpt = optValue.map(s -> Optional.of(s));
        System.out.println("nestedOpt = " + nestedOpt);

        // flatMap 을 사용하면 한 번에 평탄화
        System.out.println("[flatMap]");
        Optional<String> flattenedOpt = optValue.flatMap(s -> Optional.of(s));
        System.out.println("flattenedOpt = " + flattenedOpt);

        // 값이 있고 조건을 만족하면 그 값을 그대로, 불만족시 Optional.Empty()
        System.out.println("=== 5. filter() ===");
        Optional<String> filtered1 = optValue.filter(s -> s.startsWith("H"));
        Optional<String> filtered2 = optValue.filter(s -> s.startsWith("X"));

        System.out.println("filter(H) = " + filtered1); // Optional[Hello]
        System.out.println("filter(X) = " + filtered2); // Optional.empty

        System.out.println("=== 6. stream() ===");
        optValue.stream()
                .forEach(s -> System.out.println("optValue.stream() -> " + s));

        // 값이 없으므로 실행이 안됨
        optEmpty.stream()
                .forEach(s -> System.out.println("optValue.stream() -> " + s));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.52.45.png" alt="" width="375"><figcaption></figcaption></figure>

## 즉시 평가와 지연 평가1&#x20;

앞서 설명한 orElse(), orElseGet() 의 차이가 잘 느껴지지 않을 수 있다. 둘의 차이를 이해하려먼 즉시 평가와 지연 평가를 이해해야한다.&#x20;

* 즉시 평가(eager evaluation)&#x20;
  * 값(혹은 객체)을 바로 생성하거나 계산해 버리는 것&#x20;
* 지연 평가(lazy evaluation)&#x20;
  * 값(혹은 객체)을 실제로 필요한 때 생성하거나, 계산해 버리는 것&#x20;

여기서 평가라고 하는 것은 쉽게 이야기해 계산이라고 생각하면 된다.&#x20;

#### 로거&#x20;

```java
package optional.logger;

import java.util.function.Supplier;

public class Logger {

    private  boolean isDebug = false;

    public boolean isDebug() {
        return isDebug;
    }

    public void setDebug(boolean debug) {
        isDebug = debug;
    }

    // DEBUG 로 설정한 경우만 출력 - 데이터를 받음
    public void debug(Object message) {
        if (isDebug) {
            System.out.println("[DEBUG] " + message);
        }
    }
}
```

* 이 로거의 사용 목적은 일반적인 상황에서는 로그를 남기지 않다가, 디버깅이 필요한 경우에만 디버깅용 로그를 추가로 출력하는 것이다.&#x20;
* debug() 에 전달한 메시지는 isDebug 값을 true 로 설정한 경우에만 메시지를 출력한다.&#x20;

```java
package optional.logger;

public class LogMain1 {

    public static void main(String[] args) {
        Logger logger = new Logger();
        logger.setDebug(true);
        logger.debug(10 + 20);

        System.out.println("=== 디버그 모드 끄기 ===");
        logger.setDebug(false);
        logger.debug(100 + 200);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 09.59.48.png" alt=""><figcaption></figcaption></figure>

### 자바 언어의 연산 순서와 즉시 평가&#x20;

자바는 연산식을 보면 기본적으로 즉시 평가한다. 이 말을 이해하기 위해

**debug(10 + 20)** 연산부터 알아보자.

```java
// 자바 언어의 연산자 우선순위상 메서드를 호출하기 전에 괄호 안의 내용이 먼저 계산된다.
logger.debug(10 + 20); // 1. 여기서는 10 + 20이 즉시 평가된다.
logger.debug(30);      // 2. 10 + 20 연산의 평가 결과는 30이 된다.
debug(30)              // 3. 메서드를 호출한다. 이때 계산된 30의 값이 인자로 전달된다.
```

자바는 `10 + 20` 이라는 연산을 처리할 순서가 되면 그때 바로 즉시 평가(계산) 한다.

우리에게는 너무 자연스러운 방식이기 때문에 아무런 문제가 될 것이 없어 보인다.

그런데 이런 방식이 때로는 문제가 되는 경우가 있다.



**debug(100 + 200)** 연산을 통해 어떤 문제가 있는지 알아보자.

```java
System.out.println("=== 디버그 모드 끄기 ===");
logger.setDebug(false);
logger.debug(100 + 200);
```

이 연산은 debug 모드가 꺼져있기 때문에 출력되지 않는다. 따라서 \`100 + 200\` 연산은 어디에도 사용되지 않는다.

하지만 이 연산은 계산된 후에 버려진다. 다음 코드를 보자.

```java
// 자바 언어의 연산자 우선순위상 메서드를 호출하기 전에 괄호 안의 내용이 먼저 계산된다.
logger.debug(100 + 200);   // 1. 여기서는 100 + 200이 즉시 평가된다.
logger.debug(300);         // 2. 100 + 200 연산의 평가 결과는 300이 된다.
debug(300)                 // 3. 메서드를 호출한다. 이때 계산된 300의 값이 인자로 전달된다.
```

```java
public void debug(Object message = 300) { // 4. message에 계산된 300이 할당된다.
    if (isDebug) { // 5. debug 모드가 꺼져있으므로 false이다.
        System.out.println("[DEBUG] " + message); // 6. 실행되지 않는다.
    }
}
```

이 연산의 결과 `300` 은 debug 모드가 꺼져있기 때문에 출력되지 않는다. 따라서 앞서 계산한 `100 + 200` 연산은 어디에도 사용되지 않는다. 결과적으로 연산은 계산된 후에 버려진다.

결과적으로 `100 + 200` 연산은 미래에 전혀 사용하지 않을 값을 계산해서 아까운 CPU 전기만 낭비한 것이다.

## 즉시 평가와 지연 평가3

자바 언어에서 연산을 정의하는 시점과 해당 연산을 실행하는 시점을 분리하는 방법은 여러 가지가 있다.&#x20;

* 익명 클래스를 만들고, 메서드를 나중에 호출&#x20;
* 람다를 만들고, 람다를 나중에 호출&#x20;

람다를 사용해서 연산을 정의하는 시점과 실행하는 시점을 분리해서 문제를 해결해보자.&#x20;

**Logger에 람다(`Supplier` )를 받는 debug 메서드를 하나 추가하자**

```java
package optional.logger;

import java.util.function.Supplier;

public class Logger {

    ...

    // 추가
    // DEBUG 로 설정한 경우만 출력 - 람다를 받아서 실행
    public void debug(Supplier<?> supplier) {
        if (isDebug) {
            System.out.println("[DEBUG] " + supplier.get());
        }
    }
}
```

```java
package optional.logger;

public class LogMain3 {

    public static void main(String[] args) {
        Logger logger = new Logger();
        logger.setDebug(true);
        logger.debug(() -> value100() + value200());

        System.out.println("=== 디버그 모드 끄기 ===");
        logger.setDebug(false);
        logger.debug(() -> value100() + value200());
    }

    static int value100() {
        System.out.println("value100 호출");
        return 100;
    }

    static int value200() {
        System.out.println("value200 호출");
        return 200;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 10.05.11.png" alt=""><figcaption></figcaption></figure>

#### 정리&#x20;

람다를 사용해서 연산을 정의하는 시점과 실행(평가) 하는 시점을 분리했다. 따라서 값이 실제로 필요할 때 까지 계산을 미룰 수 있었다. 람다를 활용한 지연 평가 덕분에 꼭 필요한 계산만 처리할 수 있었다.&#x20;

## orElse() vs orElseGet()&#x20;

우리는 앞서 즉시 평가와 지연 평가에 대해서 알아보았다.&#x20;

람다를 사용하면 연산을 정의하는 시점과 실행하는 시점을 분리해서, 값이 실제로 필요할 때 까지 계산을 미룰 수 있다.&#x20;

이제 `orElse()`, `orElseGet()` 의 차이를 이해할 수 있을 것이다.&#x20;

`orElse()` 는 보통 데이터를 받아서 인자가 즉시 평가되고, `orElseGet()` 은 람다를 받아서 인자가 지연 평가된다.

```javascript
package optional;

import java.util.Optional;
import java.util.Random;

public class OrElseGetMain {

    public static void main(String[] args) {
        Optional<Integer> optValue = Optional.of(100);
        Optional<Integer> optEmpty = Optional.empty();

        System.out.println("단순 계산");
        Integer i1 = optValue.orElse(10 + 20);  // 10 + 20 계산 후 버림
        Integer i2 = optEmpty.orElse(10 + 20);  // 10 + 20 계산 후 사용
        System.out.println("i1 = " + i1);
        System.out.println("i2 = " + i2);

        // 값이 있으면 그 값, 없으면 지정된 기본 값 사용
        System.out.println("=== orElse ===");
        System.out.println("값이 있는 경우");
        Integer value1 = optValue.orElse(createData());
        System.out.println("value1 = " + value1);

        System.out.println("값이 없는 경우");
        Integer empty1 = optEmpty.orElse(createData());
        System.out.println("empty1 = " + empty1);

        // 값이 있으면 그 값, 없으면 지정된 람다 사용
        System.out.println("=== orElseGet ===");
        System.out.println("값이 있는 경우");
        Integer value2 = optValue.orElseGet(() -> createData());
        System.out.println("value2 = " + value2);

        System.out.println("값이 없는 경우");
        Integer empty2 = optEmpty.orElseGet(() -> createData());
        System.out.println("empty2 = " + empty2);
    }

    static int createData () {
        System.out.println("데이터를 생성합니다..");
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        int createValue = new Random().nextInt(100);
        System.out.println("데이터 생성이 완료되었습니다. 생성 값 : " + createValue);
        return createValue;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 10.33.15.png" alt="" width="375"><figcaption></figcaption></figure>

#### 두 메서드의 차이

* `orElse(T other)` 는 "빈 값이면 `other` 를 반환"하는데, `other` 를 "항상" 미리 계산한다.
  * 따라서 `other` 를 생성하는 비용이 큰 경우, 실제로 값이 있을 때도 쓸데없이 생성 로직이 실행될 수 있다.
  * `orElse()` 에 넘기는 표현식은 **호출 즉시 평가**하므로 **즉시 평가(eager evaluation)**&#xAC00; 적용된다.
* `orElseGet(Supplier supplier)` 은 빈 값이면 `supplier` 를 통해 값을 생성하기 때문에, **값이 있을 때는**`supplier` **가 호출되지 않는다.**
  * 생성 비용이 높은 객체를 다룰 때는 `orElseGet()` 이 더 효율적이다.
  * `orElseGet()` 에 넘기는 표현식은 **필요할 때만 평가**하므로 **지연 평가(lazy evaluation)**&#xAC00; 적용된다.

#### 사용 용도&#x20;

`orElse(T other)`

* **값이 이미 존재하지 않을 가능성이 높거나, 혹은 `orElse()` 에 넘기는 객체(또는 메서드) 가 생성 비용이 크지 않은 경우** 사용해도 괜찮다.&#x20;
* 연산이 없는 상수나 변수의 경우 사용해도 괜찮다.&#x20;

`orElseGet(Supplier supplier)`&#x20;

*   주로 **orElse()에 넘길 값의 생성 비용이 큰 경우**, 혹은 **값이 들어있을 확률이 높아 굳이 매번 대체 값을 계산할 필**

    **요가 없는 경우**에 사용한다.

정리하면, **단순한 대체 값**을 전달하거나 코드가 매우 간단하다면 **orElse()** 를 사용하고, **객체 생성 비용이 큰 로직**이 들어있고, **Optional에 값이 이미 존재할 가능성이 높다면** `orElseGet()` 을 고려해볼 수 있다.

## Optional - 베스트 프렉티스&#x20;

* `Optional` 이 좋아보여도 무분별하게 사용하면 오히려 코드 가독성과 유지보수에 도움이 되지 않을 수 있다.
* `Optional` 은 주로 **메서드의 반환값**에 대해 값이 없을 수도 있음을 표현하기 위해 도입되었다.
  * 여기서 핵심은 메서드의 반환값에 `Optional` 을 사용하라는 것이다.

`Optional` 을 사용할 때 자주 이야기되는 **베스트 프랙티스(모범 사례)**&#xB97C; 정리했다.

실무 환경에서 `Optional` 을 사용할 때 다음 내용들을 알고 사용하면, `Optional` 을 더욱 잘 활용할 수 있을 것이다.

### 1. 반환 타입으로만 사용하고, 필드에는 가급적 사용하지 말기&#x20;

#### 원칙&#x20;

* `Optional` 은 주로 **메서드의 반환값**에 대해 "값이 없을 수도 있음"을 표현하기 위해 도입되었다.
* 클래스의 필드(멤버 변수)에 `Optional` 을 직접 두는 것은 권장하지 않는다.

#### 잘못된 예시&#x20;

```java
public class Product {
    // 안티 패턴: 필드를 Optional로 선언
    private Optional<String> name;
    
    // ... constructor, getter, etc.
}
```

*   이렇게 되면 다음과 같은 3가지 상황이 발생한다.

    1\. `name = null`

    2\. `name = Optional.empty()`

    3\. `name = Optional.of(value)`
*   `Optional` 자체도 참조 타입이기 때문에, 혹시라도 개발자가 부주의로 `Optional` 필드에 `null` 을 할당하면,

    그 자체가 `NullPointerException` 을 발생시킬 여지를 남긴다.
* 값이 없음을 명시하기 위해 사용하는 것이 `Optional` 인데, 정작 필드 자체가 `null` 이면 혼란이 가중된다.

#### 권장 예시&#x20;

```java
public class Product {
    // 필드는 원시 타입(혹은 일반 참조 타입) 그대로 둔다.
    private String name;
    
    // ... constructor, getter, etc.
    
    // name 값을 가져올 때, "필드가 null일 수도 있음"을 고려해야 한다면
    // 다음 메서드에서 Optional로 변환해서 반환할 수 있다.
    public Optional<String> getNameAsOptional() {
        return Optional.ofNullable(name);
    }
}
```

### 2. 메서드 매개변수로 Optional 을 사용하지 말기&#x20;

#### 원칙&#x20;

* 자바 공식 문서에 `Optional` 은 메서드의 **반환값**으로 사용하기를 권장하며, **매개변수**로 사용하지 말라고 명시되어 있다.
* 호출하는 측에서는 단순히 `null` 전달 대신 `Optional.empty()` 를 전달해야 하는 부담이 생기며, 결국 `null` 을 사용하든 `Optional.empty()` 를 사용하든 큰 차이가 없어 가독성만 떨어진다.

#### 잘못된 예시&#x20;

```java
public void processOrder(Optional<Long> orderId) {
    if (orderId.isPresent()) {
        System.out.println("Order ID: " + orderId.get());
    } else {
        System.out.println("Order ID is empty!");
    }
}
```

*   호출하는 입장에서는 `processOrder(Optional.empty())` 처럼 호출해야 하는데, 사실

    `processOrder(null)` 과 큰 차이가 없고, 오히려 `Optional.empty()` 를 만드는 비용이 추가된다.

#### 권장 예시&#x20;

* **오버로드**된 메서드를 만들거나,
* **명시적으로 `null` 허용 여부**를 문서화하는 방식을 택합니다.

```java
// 오버로드 예시
public void processOrder(long orderId) {
    // 이 메서드는 orderId가 항상 있어야 하는 경우
    System.out.println("Order ID: " + orderId);
}
public void processOrder() {
    // 이 메서드는 orderId가 없을 때 호출할 경우
    System.out.println("Order ID is empty!");
}
```

```java
// 방어적 코드(여기서는 null 허용, 내부에서 처리)
public void processOrder(Long orderId) {
    if (orderId == null) {
        System.out.println("Order ID is empty!");
        return;
    }
    System.out.println("Order ID: " + orderId);
}
```

* 어떤 방식이든 `Optional` 을 매개변수로 받는 것은 지양하고, **오히려 반환 타입**을 `Optional` 로 두는 것이 더 자연스러운 활용 방법이다.

### 3. 컬렉션(Collection) 이나 배열 타입을 Optional 로 감싸지 말기&#x20;

#### 원칙&#x20;

* `List<T>`, `Set<T>`, `Map<K,V>` 등 **컬렉션(Collection)** 자체는 **비어있는 상태(empty)를 표현**할 수 있다.
* 따라서 `Optional<List<T>>` 처럼 다시 감싸면 `Optional.empty()` 와 "빈 리스트"(`Collections.emptyList()` )가 이중 표현이 되고, 혼란을 야기한다.

#### 잘못된 예시&#x20;

```java
public Optional<List<String>> getUserRoles(String userId) {
    List<String> userRolesList ...;
    if (foundUser) {
        return Optional.of(userRolesList);
    } else {
        return Optional.empty();
    }
}
```

반환 받는 쪽에서는 다음 코드와 같이 사용해야 한다.

```java
Optional<List<String>> optList = getUserRoles("someUser");
if (optList.isPresent()) {
    // ...
}
```

하지만 정작 내부의 리스트가 `empty` 일 수도 있으므로, 한 번 더 체크해야 하는 모호함이 생긴다.

* `Optional` 이 비어있는지 체크해야 하고, `userRolesList` 가 비어있는지 추가로 체크해야 한다.

#### 권장 예시&#x20;

```java
public List<String> getUserRoles(String userId) {
    // ...
    if (!foundUser) {
    // 권장: 빈 리스트 반환
        return Collections.emptyList();
    }
    return userRolesList;
}
```

### 4. isPresent(), get() 조합을 직접 사용하지 않기&#x20;

#### 원칙&#x20;

* `Optional` 의 `get()` 메서드는 가급적 사용하지 않아야 한다.
* `if (opt.isPresent()) { ... opt.get() ... } else { ... }` 는 사실상 `null` 체크와 다를 바가 \
  없으며, 깜빡하면 `NoSuchElementException` 같은 예외가 발생할 위험이 있다.
* 대신 `orElse`, `orElseGet`, `orElseThrow`, `ifPresentOrElse`, `map`, `filter` 등의 메서드를 활용하면 간결하고 안전하게 처리할 수 있다

#### 잘못된 예시&#x20;

```java
public static void main(String[] args) {
    Optional<String> optStr = Optional.ofNullable("Hello");
    if (optStr.isPresent()) {
        System.out.println(optStr.get());
    } else {
        System.out.println("Nothing");
    }
}
```

#### 권장 예시&#x20;

```java
public static void main(String[] args) {
    Optional<String> optStr = Optional.ofNullable("Hello");
    // 1) orElse
    System.out.println(optStr.orElse("Nothing"));
    
    // 2) ifPresentOrElse
    optStr.ifPresentOrElse(
        System.out::println,
        () -> System.out.println("Nothing")
    );
    
    // 3) map
    int length = optStr.map(String::length).orElse(0);
    System.out.println("Length: " + length);
}
```

### 5. orElseGet() vs orElse() 차이를 분명히 이해하기&#x20;

### 6. 무조건 Optional 이 좋은 것은 아니다.&#x20;

#### 원칙&#x20;

*   `Optional` 은 분명히 편의성과 안전성을 높여주지만, 모든 곳에서 "무조건" 사용하는 것은 오히려 코드 복잡성을

    증가시킬 수 있다.
* 다음과 같은 경우  **`Optional` 사용이 오히려 불필요**할 수 있다.
  * "항상 값이 있는" 상황
    *   비즈니스 로직상 `null` 이 될 수 없는 경우, 그냥 일반 타입을 사용하거나, 방어적 코드로 예외를 던지

        는 편이 낫다.
  * "값이 없으면 예외를 던지는 것"이 더 자연스러운 상황
    *   예를 들어, ID 기반으로 무조건 존재하는 DB 엔티티를 찾아야 하는 경우, `Optional` 대신 예외를

        던지는 게 API 설계상 명확할 수 있다. 물론 이런 부분은 비즈니스 상황에 따라 다를 수 있다.
  * "흔히 비는 경우"가 아니라 "흔히 채워져 있는" 경우
    *   `Optional` 을 쓰면 매번 `.get()`, `orElse()`, `orElseThrow()` 등 처리가 강제되므로 오히려

        코드가 장황해질 수 있다.
  * "성능이 극도로 중요한" 로우레벨 코드
    *   `Optional` 은 래퍼 객체를 생성하므로, 수많은 객체가 단기간에 생겨나는 영역(예: 루프 내부)에서는

        성능 영향을 줄 수 있다. (일반적인 비즈니스 로직에서는 문제가 되지 않는다. 극한 최적화가 필요한

        코드라면 고려 대상)

#### 예제 코드&#x20;

<pre class="language-java"><code class="lang-java">// 1. 항상 값이 있는 경우: 차라리 Optional 사용 X
public String findConfigValue() {
    // 이 로직은 무조건 "NotNull" 반환
    // null이 나오면 프로그래밍적 오류
    return "ConfigValue";
}

// 2. 값이 없으면 예외가 맞는 경우
public String findRequiredEntity(Long id) {
    // DB나 Repository에서 무조건 존재해야 하는 엔티티
    Entity entity = repository.find(id);
    if (entity == null) {
        throw new IllegalStateException("Required Entity not found!");
    }
    return entity.getName();
}

// 3. null이 날 가능성이 희박하고, 주요 흐름에서 필수로 존재해야 하는 경우
public String getValue(Data data) {
    // 비즈니스상 data.getValue()가 null이면 안 되는 상황이라면?
    // Optional보다 null 체크 후 예외가 더 직관적일 수 있음
    if (data.getValue() == null) {
        throw new IllegalArgumentException("Value is missing, cannot proceed!");
<strong>    }
</strong>    return data.getValue();
}
</code></pre>

#### 클라이언트 메서드 vs 서버 메서드&#x20;

사실 `Optional` 을 고려할 때 가장 중요한 핵심은 `Optional` 을 생성하고 반환하는 서버쪽 메서드가 아니라, `Optional` 을 반환하는 코드를 호출하는 클라이언트 메서드에 있다. **결과적으로 `Optional` 을 반환받는 클라이언트의 입장을 고려해서 하는 선택이, `Optional` 을 가장 잘 사용하는 방법이다.**

* **"이 로직은 `null` 을 반환할 수 있는가?"**
* **"`null` 이 가능하다면, 호출하는 사람 입장에서 '값이 없을 수도 있다'는 사실을 명시적으로 인지할 필요가 있는가?"**
* **`null` 이 적절하지 않고, 예외를 던지는 게 더 맞진 않은가?"**
