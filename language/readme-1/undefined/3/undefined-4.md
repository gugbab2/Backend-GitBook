# 메서드 참조

## 메서드 참조가 필요한 이유&#x20;

이번에는 특정 상황에서 람다를 조금 더 편리하게 사용할 수 있는 메서드 참조(Method References) 에 대해 알아보자.&#x20;

### 예제1&#x20;

먼저 코드를 통해 메서드 참조가 필요한 이유를 알아보자.&#x20;

```java
package mtehodref.start;

import java.util.function.BinaryOperator;

public class MethodRefStartV1 {

    public static void main(String[] args) {
        BinaryOperator<Integer> add1 = (x, y) -> x + y;
        BinaryOperator<Integer> add2 = (x, y) -> x + y;

        Integer result1 = add1.apply(1, 2);
        System.out.println("result1 = " + result1);

        Integer result2 = add2.apply(1, 2);
        System.out.println("result2 = " + result2);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.09.19.png" alt=""><figcaption></figcaption></figure>

이 예제는 가장 기본적인 람다를 보여준다. 두 정수를 더하는 간단한 연산을 수행하는데, `add1`, `add2` 는 동일한 기능을 하는 람다를 각각 정의하고 있다.&#x20;

여기에는 다음과 같은 문제점이 있다.&#x20;

* 동일한 기능을 하는 람다를 여러번 작성해야 한다.&#x20;
* 코드가 중복되어 있어서, 유지보수의 어려움이 있을 수 있다.&#x20;
* 만약 덧셈 로직이 변경되어야 한다면, 모든 람다를 각각 수정해야 한다.&#x20;

### 예제2&#x20;

```java
package mtehodref.start;

import java.util.function.BinaryOperator;

public class MethodRefStartV2 {

    public static void main(String[] args) {
        BinaryOperator<Integer> add1 = (x, y) -> add(x, y);
        BinaryOperator<Integer> add2 = (x, y) -> add(x, y);

        Integer result1 = add1.apply(1, 2);
        System.out.println("result1 = " + result1);

        Integer result2 = add2.apply(1, 2);
        System.out.println("result2 = " + result2);
    }

    static int add(int x, int y) {
        return x + y;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.11.42.png" alt=""><figcaption></figcaption></figure>

이번 예제에서는 코드 중복 문제를 해결했다.&#x20;

* 덧셈 로직을 별도의 `add()` 메서드로 분리했다.&#x20;
* 람다는 `add()` 메서드를 호출한다.&#x20;
* 로직이 한 곳으로 모여 유지보수가 쉬워졌다.&#x20;

#### 남은 문제&#x20;

* 람다를 수정할 때마다 `(x, y) -> add(x, y)` 형태의 토크를 반복해서 작성해야 한다.&#x20;
* 매개변수를 전달하는 부분이 장황하다.&#x20;

### 예제3&#x20;

```java
package mtehodref.start;

import java.util.function.BinaryOperator;

public class MethodRefStartV3 {

    public static void main(String[] args) {
        BinaryOperator<Integer> add1 = MethodRefStartV3::add;   // (x, y) -> add(x, y);
        BinaryOperator<Integer> add2 = MethodRefStartV3::add;   // (x, y) -> add(x, y);

        Integer result1 = add1.apply(1, 2);
        System.out.println("result1 = " + result1);

        Integer result2 = add2.apply(1, 2);
        System.out.println("result2 = " + result2);
    }

    static int add(int x, int y) {
        return x + y;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.13.26.png" alt=""><figcaption></figcaption></figure>

여기서는 메서드 참조(Method Reference) 문법인 `클래스명::메서드명` 을 사용하여 `(x, y) -> add(x, y)` 라는 람다를 더욱 간단하게 표현했다. 이는 내부적으로 `(x, y) -> add(x, y)` 와 동일하게 작동한다.&#x20;

```java
BinaryOperator<Integer> add1 = (x, y) -> add(x, y);
BinaryOperator<Integer> add1 = MethodRefStartV3::add;
```

#### 메서드 참조의 장점&#x20;

* 메서드 참조를 사용하면 코드가 더욱 간결해지고, 가독성이 향상된다.&#x20;
* 더 이상 매개변수를 명시적으로 작성할 필요가 없다.&#x20;
  * 컴파일러가 자동으로 매개변수를 매칭한다.&#x20;
* 별도의 로직 분리와 함께 재사용성 역시 높아진다.&#x20;

#### 메서드 참조란?&#x20;

메서드 참조를 쉽게 말해서, **"이미 정의된 메서드를 그대로 참조하여 람다 표현식을 더 간결하게 작성하는 방법"** 이라고 할 수 있다. 예를 들어 `(x, y) -> add(x, y)` 라는 람다는 사실상 매개변수 `x`, `y` 를 그대로 `add` 메서드에 전달하기만 하는 코드이므로, `클래스명::메서드명` 형태의 메서드 참조로 간단히 표현할 수 있다. 이렇게 하면 불필요한 매개변수 선언 없이 코드가 깔끔해지고, 가독성도 높아진다.

#### 정리&#x20;

**메서드 참조는 이미 정의된 메소드를 람다로 변환하여 더욱 간결하게 사용할 수 있도록 해주는 문법적 편의 기능이다.** \
메서드 참조를 사용하면 이미 정의된 메서드를 장황한 람다 대신 간단하고 직관적으로 사용할 수 있다.\
이처럼 **람다를 작성할 때, 이미 정의된 메서드를 그대로 호출하는 경우**라면 메서드 참조를 사용해 더욱 직관적이고 간결한 \
코드를 작성할 수 있다.

## 메서드 참조1 - 시작

**메서드 참조는 이미 정의된 메소드를 람다로 변환하여 더욱 간결하게 사용할 수 있도록 해주는 문법적 편의 기능이다.** \
즉, 람다 내부에서 단순히 어떤 메서드(정적/인스턴스/생성자 등) 를 호출만하는 경우, 다음과 같은 형태로 메서드 참조를 사용할 수 있다.

```javascript
(x, y) -> 클래스명.메서드명(x, y) // 기존 람다
클래스명::메서드명 // 메서드 참조
```

이때 람다와 메서드 참조는 동일하게 동작한다. \
쉽게 이야기해서 메서드 참조는 람다가 단순히 어떤 메서드만 호출하는 경우, 이를 축약해주는 문법이라고 이해하면 된다.&#x20;

### 메서드 참조의 4가지 유형&#x20;

1. 정적 메서드 참조&#x20;
2. 특정 객체의 인스턴스 메서드 참조&#x20;
3. 생성자 참조&#x20;
4. 임의 객체의 인스턴스 메서드 참조&#x20;

#### 1. 정적 메서드 참조&#x20;

* 설명 : 이름 그대로(정적) 메서드를 참조한다.&#x20;
* 문법 : `클래스명::메서드명`
* &#x20;예 : `Math:max`, `Integer::parseInt`

#### 2. 특정 객체의 인스턴스 메서드 참조&#x20;

* 설명 : 이름 그대로 특정 객체의 인스턴스 메서드를 참조한다.&#x20;
* 문법 : `객체명::인스턴스메서드명`
* 예 : `person::introduce`, `person::getName 등`

#### 3. 생성자 참조&#x20;

* 설명 : 이름 그대로 생성자를 참조한다.&#x20;
* 문법 : `클래스명::new`
* 예 : `Person::new`

#### 4. 임의 객체의 인스턴스 메서드 참조&#x20;

* 설명 : 첫 번째 매개변수(또는 해당 람다가 받을 대상) 그 메서드를 호출하는 객체가 된다.&#x20;
* 문법 : `클래스명::인스턴스메서드명`
* 예 : `Person::introduce`, 같은 람다: `(Person p) -> p.introduce()`

### 예제1&#x20;

먼저 예제를 위한 간단한 `Person` 클래스를 만들어두자.&#x20;

```java
package mtehodref;

public class Person {

    private String name;

    public Person() {
        this("Unknown");
    }

    public Person(String name) {
        this.name = name;
    }

    // 정적 메서드
    public static String greeting() {
        return "Hello";
    }

    // 정적 메서드, 매개변수
    public static String greetingWithName(String name) {
        return "Hello" + name;
    }

    public String getName() {
        return name;
    }

    // 인스턴스 메서드
    public String introduce() {
        return "I am " + name;
    }

    // 인스턴스 메서드, 매개변수
    public String introduceWithNumber(int number) {
        return "I am " + name + ", my number is " + number;
    }
}
```

```java
package mtehodref;

import java.util.function.Supplier;

public class MethodRefEx1 {

    public static void main(String[] args) {
        // 1. 정적 메서드 참조
        Supplier<String> staticMethod1 = () -> Person.greeting();
        Supplier<String> staticMethod2 = Person::greeting;  // 클래스::정적메서드

        System.out.println("staticMethod1 = " + staticMethod1.get());
        System.out.println("staticMethod2 = " + staticMethod2.get());

        // 2. 특정 객체의 인스턴스 참조
        Person person = new Person("Kim");
        Supplier<String> instanceMethod1 = () -> person.introduce();
        Supplier<String> instanceMethod2 = person::introduce;

        System.out.println("instanceMethod1 = " + instanceMethod1.get());
        System.out.println("instanceMethod2 = " + instanceMethod2.get());

        // 3. 생성자 참조
        Supplier<Person> newPerson1 = () -> new Person();
        Supplier<Person> newPerson2 = Person::new;

        System.out.println("newPerson1 = " + newPerson1.get());
        System.out.println("newPerson2 = " + newPerson2.get());
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.27.12.png" alt=""><figcaption></figcaption></figure>

#### 1. Person::greeting

* 정적 메서드를 참조한 예시이다.&#x20;
* `() -> Person.greeting()` 을 `Person::greeting` 으로 간단히 표현했다.&#x20;

#### 2. person::introduce

* 특정 객체(`person`) 의 인스턴스 메서드를 참조한 예시이다.&#x20;
* `() -> person.introduce()` 를 `person::introduce` 로 간략화했다.&#x20;

#### 3. Person::new&#x20;

* 생성자를 참조한 예시이다.&#x20;
* `() -> new Person()` 을 `Person::new` 로 대체했다.&#x20;

#### 메서드 참조에서 () 를 사용하지 않는 이유&#x20;

* 참고로 메서드 참조의 문법을 잘 보면 뒤에 메서드 명 뒤에 `()` 가 없다.
* `()` 는 메서드를 즉시 호출한다는 의미를 가진다. 여기서 `()` 가 없다는 것은 메서드 참조를 하는 시점에는 메서드를 호출하는게 아니라 **단순히 메서드의 이름으로 해당 메서드를 참조만 한다는 뜻이다.**&#x20;

## 메서드 참조2 - 매개변수1&#x20;

이번에는 매개변수가 있을 때 메서드를 어떻게 참조하는지 알아보자.&#x20;

```java
package mtehodref;

import java.util.function.Function;

public class MethodRefEx2 {

    public static void main(String[] args) {
        // 1. 정적 메서 참조
        Function<String, String> staticMethod1 = name -> Person.greetingWithName(name);
        Function<String, String> staticMethod2 = Person::greetingWithName;

        System.out.println("staticMethod1 = " + staticMethod1.apply("Kim"));
        System.out.println("staticMethod2 = " + staticMethod2.apply("Kim"));

        // 2. 특정 객체의 인스턴스 참조
        Person person = new Person("Kim");
        Function<Integer, String> instanceMethod1 = n -> person.introduceWithNumber(n);
        Function<Integer, String> instanceMethod2 = person::introduceWithNumber;

        System.out.println("instanceMethod1 = " + instanceMethod1.apply(1));
        System.out.println("instanceMethod2 = " + instanceMethod2.apply(1));

        // 3. 생성자 참조
        Function<String, Person> newPerson1 = name -> new Person(name);
        Function<String, Person> newPerson2 = Person::new;

        System.out.println("newPerson1 = " + newPerson1.apply("Kim"));
        System.out.println("newPerson2 = " + newPerson2.apply("Kim"));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.33.43.png" alt=""><figcaption></figcaption></figure>

이 예제에서는 매개변수가 있는 메서드 참조를 다룬다.&#x20;

메서드 참조의 경우 매개변수를 생략한다. 매개변수가 여러개라면 순서대로 전달된다.&#x20;

#### 정적 메서드 참조 (매개변수가 있는 경우)&#x20;

* `Function<String, String> staticMethod1 = name -> Person.greetingWithName(name)`&#x20;
* `Function<String, String> staticMethod2 = Person::greetingWithName`

#### 특정 인스턴스의 인스턴스 메서드 참조 (매개변수가 있는 경우)&#x20;

* `Function<Integer, String> instanceMethod1 = n -> instance.introduceWithNumber(n)`
* `Function<Integer, String> instanceMethod2 = instance::introduceWithNumber`

#### 생성자 참조 (매개변수가 있는 생성자)&#x20;

* `Function<String, Person> supplier1 = name -> new Person(name)`
* `Function<String, Person> supplier2 = Person::new`

#### 메서드 참조에서 매개변수를 생략하는 이유&#x20;

함수형 인터페이스의 시그니처(매개변수와 반환 타입) 가 이미 정해져 있고, 컴파일러가 그 시그니처를 바탕으로 메서드 참조와 연결해주기 때문에, 명시적으로 매개변수를 작성하지 않아도 자동으로 추론되어 호출된다.&#x20;

## 메서드 참조3 - 임의 객체의 인스턴스 메서드 참조

### 예제3&#x20;

```java
package mtehodref;

import java.util.function.Function;

public class MethodRefEx3 {

    public static void main(String[] args) {
        // 4. 임의 객체의 인스턴스 메서드 참조(특정 타입의)
        Person person1 = new Person("Kim");
        Person person2 = new Person("Park");
        Person person3 = new Person("Lee");

        // 람다
        Function<Person, String> fun1 = (Person person) -> person.introduce();
        System.out.println("person1.introduce = " + fun1.apply(person1));
        System.out.println("person2.introduce = " + fun1.apply(person2));
        System.out.println("person3.introduce = " + fun1.apply(person3));

        // 메서드 참조, 타입이 첫 번째 매개변수가 됨,
        // 그리고 첫 번째 매개변수의 메서드 호출, 나머지는 순서대로 매개변수에 전달
        Function<Person, String> fun2 = Person::introduce;  // 타입::인스턴스메서드
        System.out.println("person1.introduce = " + fun2.apply(person1));
        System.out.println("person2.introduce = " + fun2.apply(person2));
        System.out.println("person3.introduce = " + fun2.apply(person3));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 15.56.17.png" alt=""><figcaption></figcaption></figure>

#### 람다 정의&#x20;

```java
Function<Person, String> fun1 = (Person person) -> person.introduce();
```

* 이 람다는 `Person` 타입을 매개변수로 받는다. 그리고 매개변수로 넘겨받은 `person` 인스턴스의 `introduce()` 인스턴스 메서드를 호출한다.

#### 람다 실행&#x20;

```java
System.out.println("person1.introduce = " + fun1.apply(person1));
System.out.println("person2.introduce = " + fun1.apply(person2));
System.out.println("person3.introduce = " + fun1.apply(person3));
```

*   앞서 정의한 람다는 `Function<Person, String>` 함수형 인터페이스를 사용한다. 따라서 `Person` 타입의

    인스턴스를 인자로 받고, `String` 을 반환한다.
* 코드에서 보면 `apply()` 에 `person1`, `person2`, `person3` 을 각각 전달한 것을 확인할 수 있다.
  * `person1` 을 람다에 전달하면 `person1.introduce()` 가 호출된다.
  * `person2` 를 람다에 전달하면 `person2.introduce()` 가 호출된다.
  * `person3` 을 람다에 전달하면 `person3.introduce()` 가 호출된다.

이 람다는 **매개변수로 지정한 특정 타입의 객체에 대해 동일한 메서드를 호출하는 패턴**을 보인다.&#x20;

이렇게 특정 타입의 임의 객체에 대해 동일한 인스턴스 메서드를 호출하는 패턴을 메서드 참조로 손쉽게 표현할 수 있다.&#x20;

```java
Function<Person, String> fun1 = (Person person) -> person.introduce() // 람다
Function<Person, String> fun2 = Person::introduce // 메서드 참조 (타입::인스턴스메서드)
```

### 임의 객체의 인스턴스 메서드 참조&#x20;

"`(Reference to an instance method of an arbitrary object of a particular type)`"

이런 메서드 참조를 **특정 타입의 임의 객체의 인스턴스 참조**라 한다.\
&#xNAN;**(실제로 메서드 참조 기능 중 가장 많이 사용된다)**

여기서 줄여서 **임의 객체의 인스턴스 참조**라 하겠다.&#x20;

임의 객체의 인스턴스 참조는 `클래스명::인스턴스메서드` 의 형태로 사용한다.&#x20;

* 주의! 왼쪽이 **클래스명**이고, 오른쪽이 **인스턴스 메서드**이다!&#x20;

`Person::introduce` 와 같이 선언하면 다음과 같은 람다가 된다.

```java
Person::introduce
1. 왼쪽에 지정한 클래스를 람다의 첫 번째 매개변수로 사용한다.
(Person person)

2. 오른쪽에 지정한 '인스턴스 메서드'를 첫 번째 매개변수를 통해 호출한다.
(Person person) -> person.introduce()
```

## 메서드 참조4 - 활용1&#x20;

임의 객체의 인스턴스 참조가 실제 어떻게 사용되는지 알아보자.

```java
package mtehodref;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class MethodRefEx4 {

    public static void main(String[] args) {
        List<Person> personList = List.of(
                new Person("Kim"),
                new Person("Park"),
                new Person("Lee")
        );

        List<String> result1 = mapPersonToString(personList, (Person p) -> p.introduce());
        List<String> result2 = mapPersonToString(personList, Person::introduce);
        System.out.println("result1 = " + result1);
        System.out.println("result2 = " + result2);

        List<String> upperResult1 = mapStringToString(result1, (String s) -> s.toUpperCase());
        List<String> upperResult2 = mapStringToString(result1, String::toUpperCase);
        System.out.println("upperResult1 = " + upperResult1);
        System.out.println("upperResult2 = " + upperResult2);
    }

    static List<String> mapPersonToString(List<Person> personList, Function<Person, String> fun) {
        List<String> result = new ArrayList<>();
        for (Person p : personList) {
            String applied = fun.apply(p);
            result.add(applied);
        }
        return result;
    }

    static List<String> mapStringToString(List<String> strings, Function<String, String> fun) {
        List<String> result = new ArrayList<>();
        for (String s : strings) {
            String applied = fun.apply(s);
            result.add(applied);
        }
        return result;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 16.07.59.png" alt=""><figcaption></figcaption></figure>

람다 대신 메서드 참조를 사용한 덕분에 코드가 간결해지고, 의도가 더 명확하게 드러나는 것을 확인할 수 있다.&#x20;

* `mapPersonToString(personList, Person::introduce)`&#x20;
  * `Person` 리스트에 있는 각각의 `Person` 인스턴스에 `introduce` 를 호출하고 그 결과를 리스트로 반환
* `mapStringToString(result2, String::toUpperCase)`
  * `String` 리스트에 있는 각각의 `String` 인스턴스에 `toUpperCase` 를 호출하고 그 결과를 리스트로 반환

우리가 앞서 만든 스트림을 사용하면 리스트에 들어있는 다양한 데이터를 더 쉽게 변환할 수 있을 것 같다.&#x20;

## 메서드 참조5 - 활용2&#x20;

이번에는 스트림에 메서드 참조를 활용해보자.&#x20;

### 예제5

```java
package mtehodref;

import lambda.lambda5.mystream.MyStreamV3;

import java.util.List;

public class MethodRefEx5 {

    public static void main(String[] args) {
        List<Person> personList = List.of(
                new Person("Kim"),
                new Person("Park"),
                new Person("Lee")
        );

        List<String> result1 = MyStreamV3.of(personList)
                .map(person -> person.introduce())
                .map(str -> str.toUpperCase())
                .toList();
        System.out.println("result1 = " + result1);

        List<String> result2 = MyStreamV3.of(personList)
                .map(Person::introduce)
                .map(String::toUpperCase)
                .toList();
        System.out.println("result2 = " + result2);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 16.11.03.png" alt=""><figcaption></figcaption></figure>

#### 메서드 참조의 장점&#x20;

**메서드 참조를 사용하면 람다 표현식을 더욱 직관적으로 표현할 수 있으며, 각 처리 단계에서 호출되는 메서드가 무엇인지 파악할 수 있다.**&#x20;

이처럼 람다로도 충분히 표현할 수 있지만, 내부적으로 호출만 하는 간단한 람다라면 메서드 참조가 더 짧고 명확하게 표현할 수 있다. 이런 방식은 코드 가독성을 옾이는 장점이 있다. 물론 메서드 참조 방식에 익숙해지는데는 시간이 걸린다..&#x20;

## 메서드 참조6 - 매개변수2&#x20;

이번에는 임의 객체의 인스턴스 메서드 참조에서 매개변수가 늘어나면 어떻게 되는지 알아보자.&#x20;

### 예제6&#x20;

```java
package mtehodref;

import java.util.function.BiFunction;

// 매개변수 추가
public class MethodRefEx6 {

    public static void main(String[] args) {
        // 4. 임의 객체의 인스턴스 메서드 참조(특정 타입의)
        Person person = new Person("Kim");

        // 람다
        BiFunction<Person, Integer, String> fun1 =
                (Person p, Integer number) -> p.introduceWithNumber(number);

        System.out.println("person.introduceWithNumber = " + fun1.apply(person, 1));

        // 메서드 참조, 타입이 첫 번째 매개변수가 됨, 그리고 첫 번째 매개변수의 메서드를 호출
        // 그리고 나머지는 순서대로 매개변수에 전달
        BiFunction<Person, Integer, String> fun2 = Person::introduceWithNumber; // 타입::메서드명
        System.out.println("person.introduceWithNumber = " + fun2.apply(person, 1));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 16.17.20.png" alt=""><figcaption></figcaption></figure>

#### 코드 분석&#x20;

```java
// 람다 사용
BiFunction<Person, Integer, String> fun1 =
(Person p, Integer number) -> p.introduceWithNumber(number);

// 메서드 참조 사용
BiFunction<Person, Integer, String> fun2 = Person::introduceWithNumber;
```

* `BiFunction<Person, Integer, String>` 인터페이스를 사용하여 `(Person, Integer) -> String` \
  형태의 람다/메서드 참조를 구현한다.
* `fun1` 에서는 람다를 사용하여 `p.introduceWithNumber(number)` 를 호출한다.
* `fun2` 에서는 `Person::introduceWithNumber` 라는 메서드 참조를 사용한다. 첫 번째 매개변수(`Person`) 가 메서드를 호출하는 객체가 되고, 두 번째 매개변수(`Integer`)가 `introduceWithNumber()` 의 실제 인자로 전달된다. 첫 번째 이후의 매개변수는 모두 순서대로 실제 인자로 전달된다.

이처럼 **임의 객체의 인스턴스 메서드 참조**는 함수형 인터페이스의 시그니처에 따라

* 첫 번째 인자를 **호출 대상 객체**로
* 나머지 인자들은 순서대로 **해당 메서드의 매개변수**로 전달한다.
