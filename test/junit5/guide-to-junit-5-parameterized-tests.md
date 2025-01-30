# Guide to JUnit 5 Parameterized Tests

> 참고 링크&#x20;
>
> [https://www.baeldung.com/parameterized-tests-junit-5](https://www.baeldung.com/parameterized-tests-junit-5)

## 1. 첫인상&#x20;

* 기존에 유틸리티 함수가 있다고 생각하고, 해당 함수 동작에 대한 확신을 가지고 싶다.&#x20;

```java
public class Numbers {
    public static boolean isOdd(int number) {
        return number % 2 != 0;
    }
}
```

* 매개변수화 된 테스트는 다른 테스트와 유사하지만, `@ParameterizedTest` 어노테이션을 추가한다.&#x20;
  * 아래 테스트는 결과적으로 `@ValueSource` 배열에서 각각의 값을 `isOdd_ShouldReturnTrueForOddNumbers` 매개변수(`number`) 값으로 사용해 해당 테스트를 6번 실행한다.&#x20;

```java
@ParameterizedTest
@ValueSource(ints = {1, 3, 5, -3, 15, Integer.MAX_VALUE}) // six numbers
void isOdd_ShouldReturnTrueForOddNumbers(int number) {
    assertTrue(Numbers.isOdd(number));
}
```

## 2. 테스트 케이스&#x20;

* 지금까지 알고 있었겠지만 매개변수화된 테스트는 서로 다른 인수를 사용하여 동일한 테스트를 여러 번 실행한다.&#x20;

### Primitive type&#x20;

* **`@ValueSource` 를 사용하면 기본형 타입 값 배열을 테스트 메서드에 전달할 수 있습니다.**
* 우리가 간단한 `isBlank` 메서드를 테스트할 것이라고 가정해 보자.&#x20;

```java
public class Strings {
    public static boolean isBlank(String input) {
        return input == null || input.trim().isEmpty();
    }
}
```

```java
@ParameterizedTest
@ValueSource(strings = {"", "  "})
void isBlank_ShouldReturnTrueForNullOrBlankStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

#### 제약사항&#x20;

* 제약사항 중 하나는 `@ValueSource` 는 기본형 타입의 값만을 제공한다.&#x20;
  * _`short`_&#x20;
  * _`byte`_&#x20;
  * _`int`_&#x20;
  * _`long`_&#x20;
  * _`float`_&#x20;
  * _`double`_
  * _`char`_
  * _`java.lang.String`_
  * _`java.lang.Class`_
* 또한 테스트 메서드 하나에 하나의 타입 종류만 사용할 수 있다.&#x20;
* 마지막으로 `@ValueSource` 를 통해서 `null` 을 전달할 수 없다.&#x20;

### Null, Empty Value

* **JUnit 5.4부터  `@NullSource`를 사용하여 매개변수화된 테스트 메서드에  단일 `null`**  값을 전달할 수 있다.&#x20;

```java
@ParameterizedTest
@NullSource
void isBlank_ShouldReturnTrueForNullInputs(String input) {
    assertTrue(Strings.isBlank(input));
}
```

* 기본형 타입을 매개변수로 받는 케이스는 `@NullSource` 를 사용할 수 없다.&#x20;
* 때문에, 매우 유사한 방식으로 동작하는 `@EmptySource` 를 사용하여 빈 값을 전달할 수 있다.&#x20;

```java
@ParameterizedTest
@EmptySource
void isBlank_ShouldReturnTrueForEmptyStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

* `null` 값과 빈 값 모두를 전달하기 위해서는 `@NullAndEmptySource` 를 사용할 수 있다.&#x20;

```java
@ParameterizedTest
@NullAndEmptySource
void isBlank_ShouldReturnTrueForNullAndEmptyStrings(String input) {
    assertTrue(Strings.isBlank(input));
}
```

### Enum

* 예를 들어서, 모든 월 숫자가 1\~12 임을 확인할 수 있다.&#x20;

```java
@ParameterizedTest
@EnumSource(Month.class) // passing all 12 months
void getValueForAMonth_IsAlwaysBetweenOneAndTwelve(Month month) {
    int monthNumber = month.getValue();
    assertTrue(monthNumber >= 1 && monthNumber <= 12);
}
```

* 또한 `names` 속성을 사용해 필터링을 할 수 있다.&#x20;

```java
@ParameterizedTest
@EnumSource(value = Month.class, names = {"APRIL", "JUNE", "SEPTEMBER", "NOVEMBER"})
void someMonths_Are30DaysLong(Month month) {
    final boolean isALeapYear = false;
    assertEquals(30, month.length(isALeapYear));
}
```

...
