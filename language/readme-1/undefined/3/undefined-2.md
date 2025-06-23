# 함수형 인터페이스

## 함수형 인터페이스와 제네릭1&#x20;

함수형 인터페이스도 인터페이스이기 때문에, 제네릭을 도입할 수 있다.

먼저 함수형 인터페이스에 제네릭이 필요한 이유를 알아보자.

### 각각 다른 타입 사용&#x20;

다음 코드는 문자 타입, 숫자 타입을 각각 처리하는 두 개의 함수형 인터페이스를 사용한다.&#x20;

```java
package lambda.lambda3;

public class GeericMain1 {

    public static void main(String[] args) {
        StringFunction upperCase = s -> s.toUpperCase();
        String result1 = upperCase.apply("hello");
        System.out.println("result1 = " + result1);

        NumberFunction square = n -> n * n;
        Integer result2 = square.apply(3);
        System.out.println("result2 = " + result2);
    }

    @FunctionalInterface
    interface StringFunction {
        String apply(String s);
    }

    @FunctionalInterface
    interface NumberFunction {
        Integer apply(Integer i);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 21.00.12.png" alt=""><figcaption></figcaption></figure>

`StringFunction` 이 제공하는 `apply` 메서드와 `NumberFunction` 이 제공하는 `apply` 메서드는 둘다 하나의 인자를 입력 받고, 결과를 반환한다. 다만 입력받는 타입과 반환 타입이 다를 뿐이다. 이렇게 매개변수나 반환 타입이 다를 때마다 계속 함수형 인터페이스를 만들어야 할까?&#x20;

### Object 타입으로 합치기

`Object` 는 모든 타입의 부모이다. 따라서 다형성(다형적 참조) 를 사용해서 이 문제를 간단히 해결할 수 있을 것 같다.&#x20;

```javascript
package lambda.lambda3;

public class GeericMain2 {

    public static void main(String[] args) {
        ObjectFunction upperCase = s -> ((String)s).toUpperCase();
        String result1 = (String) upperCase.apply("hello");
        System.out.println("result1 = " + result1);

        ObjectFunction square = n -> (Integer)n * (Integer)n;
        Integer result2 = (Integer) square.apply(3);
        System.out.println("result2 = " + result2);
    }

    @FunctionalInterface
    interface ObjectFunction {
        Object apply(Object obj);
    }
}
```

* 메서드가 `Object` 를 매개변수로 사용하고, `Object` 를 반환하면 모든 타입을 입력 받고, 또 모든 타입을 반환할 수 있다. 따라서 이전과 같은 타입에 따라 각각 다른 함수형 인터페이스를 만들지 않아도 된다. 따라서 앞서 각각 만든 함수형 인터페이스 2개를 1개로 합칠 수 있다.&#x20;
* 물론 `Object` 를 사용하기 때문에 복잡하고 안전하지 않은 캐스팅 과정이 필요하다.&#x20;
* 실행 결과는 기존과 같다.&#x20;

코드를 이해하기 쉽게 익명 클래스로 변경해보자.&#x20;

```java
package lambda.lambda3;

public class GeericMain3 {

    public static void main(String[] args) {
        ObjectFunction upperCase = new ObjectFunction() {
            @Override
            public Object apply(Object s) {
                return ((String) s).toUpperCase();
            }
        };
        String result1 = (String) upperCase.apply("hello");
        System.out.println("result1 = " + result1);

        ObjectFunction square = new ObjectFunction() {
            @Override
            public Object apply(Object n) {
                return (Integer) n * (Integer) n;
            }
        };
        Integer result2 = (Integer) square.apply(3);
        System.out.println("result2 = " + result2);
    }

    @FunctionalInterface
    interface ObjectFunction {
        Object apply(Object obj);
    }
}
```

* 실행 결과는 기존과 같다.&#x20;

#### 정리&#x20;

`Object` 와 다형성을 활용한 덕분에 코드의 중복을 제거하고, 재사용성을 늘리게 되었다. 하지만, `Object` 를 사용하므로 다운 캐스팅을 해야 하고, 결과적으로 타입 안정성 문제가 발생한다.&#x20;

지금까지 개발한 프로그램은 코드 재사용과 타입 안정성이라는 2마리 토끼를 한번에 잡을 수 없다. 코드 재사용을 늘리기 위해 `Object` 와 다형성을 사용하면 타입 안정성이 떨어지는 문제가 발생한다.&#x20;

* `StringFunction` , `NumberFunction` 각각의 타입별로 함수형 인터페이스를 모두 정의
  * 코드 재사용 X&#x20;
  * 타입 안정성 O&#x20;
* `ObjectFunction` 를 사용해서 `Object`의 다형성을 활용해서 하나의 함수형 인터페이스만 정의
  * 코드 재사용 O&#x20;
  * 타입 안정성 X&#x20;

## 함수형 인터페이스와 제네릭2&#x20;

### 제네릭 도입&#x20;

이제 함수형 인터페이스에 제네릭을 도입해서 코드 재사용도 늘리고, 타입 안정성까지 높여보자.&#x20;

```java
package lambda.lambda3;

public class GeericMain4 {

    public static void main(String[] args) {
        GenericFunction<String, String> upperCase = new GenericFunction<>() {
            @Override
            public String apply(String s) {
                return s.toUpperCase();
            }
        };
        String result1 = upperCase.apply("hello");
        System.out.println("result1 = " + result1);

        GenericFunction<Integer, Integer> square = new GenericFunction<>() {
            @Override
            public Integer apply(Integer n) {
                return n * n;
            }
        };
        Integer result2 = square.apply(3);
        System.out.println("result2 = " + result2);
    }

    @FunctionalInterface
    interface GenericFunction<T, R> {
        R apply(T t);
    }
}
```

* 실행 결과는 다음과 같다.&#x20;

`ObjectFunction` -> `GenericFunction` 으로 변경했다.

```java
@FunctionalInterface
interface GenericFunction<T, R> {
    R apply(T t);
}
```

* `T` : 매개변수 타입&#x20;
* `R` : 반환 타입&#x20;

함수형 인터페이스에 제네릭을 도입한 덕분에 메서드 `apply()` 의 매개변수와 반환 타입을 유연하게 변경할 수 있다.&#x20;

### 제네릭과 람다&#x20;

앞서 만든 익명 클래스를 이제 람다로 변경해보자.&#x20;

```java
package lambda.lambda3;

public class GeericMain5 {

    public static void main(String[] args) {
        GenericFunction<String, String> upperCase = s -> s.toUpperCase();
        String result1 = upperCase.apply("hello");
        System.out.println("result1 = " + result1);

        GenericFunction<Integer, Integer> square = n -> n * n;
        Integer result2 = square.apply(3);
        System.out.println("result2 = " + result2);
    }

    @FunctionalInterface
    interface GenericFunction<T, R> {
        R apply(T t);
    }
}
```

* 실행 결과는 기존과 같다.&#x20;
* 익명 클래스를 람다로 변경했다.&#x20;

`GenericFunction` 은 매개변수가 1개이고, 반환값이 있는 모든 람다에 사용할 수 있다. \
매개변수의 타입과 반환값은 사용시점에 제네릭을 활용해서 얼마든지 변경할 수 있기 때문이다.&#x20;

제네릭이 도입된 함수형 인터페이스는 재사용성이 매우 높다.&#x20;

#### 제네릭이 도입된 함수형 인터페이스의 활용&#x20;

```java
package lambda.lambda3;

public class GeericMain6 {

    public static void main(String[] args) {
        GenericFunction<String, String> toUpperCase = str -> str.toUpperCase();
        GenericFunction<String, Integer> stringLength = str -> str.length();
        GenericFunction<Integer, Integer> square = x -> x * x;
        GenericFunction<Integer, Boolean> isEven = num -> num % 2 == 0;

        System.out.println(toUpperCase.apply("hello"));
        System.out.println(stringLength.apply("hello"));
        System.out.println(square.apply(3));
        System.out.println(isEven.apply(3));
    }

    @FunctionalInterface
    interface GenericFunction<T, R> {
        R apply(T t);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 12.37.49.png" alt=""><figcaption></figcaption></figure>

#### 정리&#x20;

* **제네릭을 사용하면 동일한 구조의 함수형 인터페이스를 다양한 타입에 재사용할 수 있다.**
* 예제에서는 문자열을 대문자로 변환하기, 문자열의 길이 구하기, 숫자의 제곱 구하기, 짝수 여부 확인하기 등 서로 다른 기능들을 하나의 함수형 인터페이스로 구현했다.&#x20;
* `T` 는 입력 타입을, `R` 은 반환 타입을 나타내며, 실제 사용할 때 구체적인 타입을 지정하면 된다.
* 이렇게 제네릭을 활용하면 타입 안정성을 보장하면서도 유연한 코드를 작성할 수 있다.
* 컴파일 시점에 타입 체크가 이루어지므로 런타임 에러를 방지할 수 있다.
* 제네릭을 사용하지 않았다면 각각의 경우에 대해 별도의 함수형 인터페이스를 만들어야 했을 것이다.&#x20;
* 이는 코드 중복을 줄이고 유지보수성을 높이는데 큰 도움이 된다.&#x20;

## 람다와 타겟 타입&#x20;

### 남은 문제

우리가 만든 `GenericFunction` 은 코드 중복을 줄이고 유지보수성을 높여주지만 2가지 문제가 있다.&#x20;

#### 문제1. 모든 개발자가 비슷한 함수형 인터페이스를 개발해야 한다.&#x20;

우리가 만든 `GenericFunction` 은, 매개변수가 1개이고, 반환값이 있는 모든 람다에 사용할 수 있다. 그런데 람다를 사용하려면 함수형 인터페이스가 필수이기 때문에 전 세계 개발자들이 모두 비슷하게 `GenericFunction` 을 각각 만들어서 사용해야 한다. 그리고 비슷한 모양의 `GenericFunction` 이 많이 만들어질 것이다.&#x20;

#### 문제2. 개발자A 가 만든 함수형 인터페이스와 개발자B가 만든 함수형 인터페이스는 서로 호환되지 않는다.&#x20;

이 문제는 다음 코드를 통해서 확인해보자.&#x20;

```java
package lambda.lambda3;

public class TargetType1 {

    public static void main(String[] args) {
        // 람다 직접 대입 : 문제 없음
        FunctionA functionA = i -> "value = " + i;
        FunctionB functionB = i -> "value = " + i;

        // 이미 만들어진 FunctionA 인스턴스를 FunctionB 에 대입
//        FunctionB targetB = functionA;  // 컴파일 에러(시그니처는 같더라도 타입이 다르다..)
    }

    @FunctionalInterface
    interface FunctionA {
        String apply(Integer i);
    }

    @FunctionalInterface
    interface FunctionB {
        String apply(Integer i);
    }
}
```

람다를 함수형 인터페이스에 대입할 때는 `FunctionA`, `FunctionB` 모두 메서드 시그니처가 맞으므로 문제 없이 잘 대입된다.&#x20;

`FunctionB targetB = functionA` 부분은 컴파일 오류가 발생한다.

두 인터페이스 모두 `Integer` 를 받아 `String` 을 리턴하는 동일한 `apply()` 메서드를 가지고 있지만, **자바 타입 시스템상 전혀 다른 인터페이스**이므로 서로 호환되지 않는다.

이 부분을 자세히 알아보자

### 람다와 타겟 타입&#x20;

람다는 그 자체만으로 구체적인 타입이 정해져 있지 않고, 타겟 타입(target type) 이라고 불리는 맥락에 의해 타입이 결정된다.

```java
FunctionA functionA = i -> "value = " + i;
```

* 이 코드에서 `i -> "value = " + i` 라는 람다는 `FunctionA` 라는 타겟 타입을 만나서 비로소 `FunctionA` 타입으로 결정된다.

```java
FunctionB functionB = i -> "value = " + i;
```

* 동일한 람다라도 이런 코드가 있었다면, 똑같은 람다가 이번에는 `FunctionB` 타입으로 타겟팅되어 유효하게 컴파일된다.

정리하면 람다는 그 자체만으로 구체적인 타입이 정해져 있지 않고, 대입되는 함수형 인터페이스(타겟 타입)에 의해 비로소 타입이 결정된다. \
(`functionA`, `functionB` 의 메서드 시그니처는 같더라도 다른 타입이다!)

이렇게 타입이 결정되고 나면 이후에는 다른 타입에 대입하는 것이 불가능하다. 이후 함수형 인터페이스를 다른 함수형 인터페이스에 대입하는 것은 타입이 서로 다르기 때문에, 메서드에 시그니처가 같아도 대입이 되지 않는다.&#x20;

```java
FunctionB targetB = functionA; // 컴파일 에러!
```

위 코드를 보면, `functionA` 는 분명 `FunctionA` 타입의 변수가 이미 된 상태이다. 즉, `FunctionA` 라는 "명시적인인터페이스 타입"을 가진 객체가 되어 있다.&#x20;

그런데 이 객체를 `FunctionB` 타입에 대입하려고 할 때, 자바 컴파일러는`FunctionA` 와 `FunctionB` 가 서로 다른 타입임을 명확히 인식한다. 쉽게 이야기해서 `FunctionA` 와`FunctionB` 는 서로 타입이 다르다. 따라서 대입이 불가능하다. 이것은 마치 `Integer` 에 `String` 을 대입하는 것과 같다.

두 인터페이스가 시그니처가 같고 똑같은 모양의 함수형 인터페이스라도, 타입 자체는 별개이므로 상호 대입은 허용되지 않는다.

#### 정리&#x20;

* **람다**는 익명 함수로서 특정 타입을 가지지 않고, 대입되는 참조 변수가 어떤 함수형 인터페이스를 가리키느냐에 따라 타입이 결정 된다.&#x20;
* 한편 이미 대입된 변수(`functionA`) 는 엄연히 `FunctionA` 타입의 객체가 되었으므로, 이를 `FunctionB` 참조 변수에 그대로 대입할 수는 없다. 두 인터페이스 이름이 다르기 때문에 자바 컴파일러는 다른 타입으로 간주한다.
* 따라서 시그니처가 똑같은 함수형 인터페이스라도, 타입이 다르면 상호 대입이 되지 않는 것이 자바의 타입 시스템 규칙이다.&#x20;

### 자바가 기본으로 제공하는 함수형 인터페이스

자바는 이런 문제들을 해결하기 위해서 필요한 함수형 인터페이스 대부분을 기본으로 제공한다.&#x20;

자바가 제공하는 함수형 인터페이스를 사용하면, 비슷한 함수형 인터페이스를 불필요하게 만드는 문제는 물론이고, 함수형 인터페이스의 호환성 문제까지 해결할 수 있다.

#### Function - 자바 기본 제공&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    ...
}
```

* 자바는 `java.util.function` 패키지에 다양한 기본 함수형 인터페이스들을 제공한다.

```java
package lambda.lambda3;

import java.util.function.Function;

// 자바가 기본으로 제공하는 Function 사용
public class TargetType2 {

    public static void main(String[] args) {
        Function<String, String> upperCase = s -> s.toUpperCase();
        String reulst1 = upperCase.apply("hello");
        System.out.println("reulst1 = " + reulst1);

        Function<Integer, Integer> square = n -> n * n;
        Integer result2 = square.apply(3);
        System.out.println("result2 = " + result2);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.01.41.png" alt=""><figcaption></figcaption></figure>

**같은 타입을 사용하므로 대입도 문제 없다.**&#x20;

```java
package lambda.lambda3;

import java.util.function.Function;

// 자바가 기본으로 제공하는 Function 대입
public class TargetType3 {

    public static void main(String[] args) {
        Function<Integer, String> functionA = i -> "value = " + i;
        System.out.println(functionA.apply(10));

        Function<Integer, String> functionB = i -> "value = " + i;
        System.out.println(functionB.apply(10));

        functionB = functionA;
        System.out.println(functionB.apply(10));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.02.27.png" alt=""><figcaption></figcaption></figure>

**따라서 자바가 기본으로 제공하는 함수형 인터페이스를 사용하자.**&#x20;

## 기본 함수형 인터페이스&#x20;

#### 자바가 제공하는 대표적인 함수형 인터페이스&#x20;

* `Function` : 입력O, 반환O
* `Consumer` : 입력O, 반환X
* `Supplier` : 입력X, 반환O
* `Runnable` : 입력X, 반환X

함수형 인터페이스들은 대부분 제네릭을 활용하므로 종류가 많을 필요는 없다.&#x20;

함수형 인터페이스는 대부분 `java.util.function`  패키지에 위치한다. \
(`Runnable` 은 `java.lang` 패키지에 위치)

### Function

핵심 코드만 적어두었다. 앞으로 자바가 제공하는 함수형 인터페이스를 소개할 때는 간략하게 핵심 코드만 적어두겠다.&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
}
```

* 하나의 매개변수를 받고, 결과를 반환하는 함수형 인터페이스이다.\
  (둘 이상의 매개변수를 받는 함수형 인터페이스는 뒤에서 설명한다)&#x20;
* 입력값(T) 를 받아서 다른 타입의 출력값(R) 을 반환하는 연산을 표현할 때 사용한다. 물론 같은 타입의 출력 값도 가능하다.
* 일반적인 함수(Function) 의 개념에 가장 가깝다.&#x20;
* 예, 문자열을 받아서 정수로 변환, 객체를 받아서 특정 필드 추출 등&#x20;

#### 용어 설명&#x20;

* "Function" 은 수학적인 "함수" 개념을 그대로 반영한 이름이다.
* apply는 "적용하다"라는 의미로, 입력값에 함수를 적용해서 결과를 얻는다는 수학적 개념을 표현한다.
* 예: f(x)처럼 입력 x에 함수 f를 적용(apply)하여 결과를 얻는다.

```java
package lambda.lambda4;

import java.util.function.Function;

public class FunctionMain {

    public static void main(String[] args) {
        // 익명 클래스
        Function<String, Integer> function1 = new Function<>() {
            @Override
            public Integer apply(String s) {
                return s.length();
            }
        };
        System.out.println("function1 = " + function1.apply("hello"));

        // 람다
        Function<String, Integer> function2 = s -> s.length();
        System.out.println("function2 = " + function2.apply("hello"));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.08.41.png" alt=""><figcaption></figcaption></figure>

### Consumer&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
}
```

* 입력 값(T)만 받고, 결과를 반환하지 않는(`void` ) 연산을 수행하는 함수형 인터페이스이다.
* 입력값(T)을 받아서 처리하지만 결과를 반환하지 않는 연산을 표현할 때 사용한다.
* 입력 받은 데이터를 기반으로 내부적으로 처리만 하는 경우에 유용하다.
  * 예) 컬렉션에 값 추가, 콘솔 출력, 로그 작성, DB 저장 등

#### 용어 설명&#x20;

*   "Consumer" 는 "소비자"라는 의미로, 데이터를 받아서 소비(사용)만 하고 아무것도 돌려주지 않는다는 개념을 표

    현한다.
* accept는 "받아들이다"라는 의미로, 입력값을 받아들여서 처리한다는 동작을 설명한다.
* 예: 로그를 출력하는 consumer는 데이터를 받아서 출력만 하고 끝난다.
* 쉽게 이야기해서 입력 값을 받아서(accept) 소비(consume)해 버린다고 생각하면 된다.

```java
package lambda.lambda4;

import java.util.function.Consumer;

public class ConsumerMain {

    public static void main(String[] args) {
        // 익명 클래스
        Consumer<String> function1 = new Consumer<>() {
            @Override
            public void accept(String s) {
                System.out.println("input : " + s);
            }
        };
        function1.accept("hello consumer");

        // 람다
        Consumer<String> function2 = s -> System.out.println("input : " + s);
        function2.accept("hello consumer");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.10.25.png" alt=""><figcaption></figcaption></figure>

### Supplier&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

* 입력을 받지 않고(`()` ) 어떤 데이터를 공급(supply)해주는 함수형 인터페이스이다.
* 객체나 값 생성, 지연 초기화 등에 주로 사용된다. (지연 초기화는 뒤에서 설명)

#### 용어 설명&#x20;

* "Supplier"는 "공급자"라는 의미로, 요청할 때마다 값을 공급해주는 역할을 한다.
* get은 "얻다"라는 의미로, supplier로부터 값을 얻어온다는 개념을 표현한다.
* 예: 랜덤 값을 제공하는 supplier는 호출할 때마다 새로운 랜덤 값을 공급한다.

```java
package lambda.lambda4;

import java.util.Random;
import java.util.function.Supplier;

public class SupplierMain {

    public static void main(String[] args) {
        // 익명 클래스
        Supplier<Integer> function1 = new Supplier<>() {
            @Override
            public Integer get() {
                return new Random().nextInt(10);
            }
        };
        System.out.println("function1 = " + function1.get());

        // 람다
        Supplier<Integer> function2 = () -> new Random().nextInt(10);
        System.out.println("function2 = " + function2.get());
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.12.00.png" alt=""><figcaption></figcaption></figure>

### Runnable&#x20;

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
    void run();
}
```

* 입력값도 없고 반환값도 없는 함수형 인터페이스이다.&#x20;
* 자바에서는 원래부터 스레드 실행을 위한 인터페이스로 쓰였지만, 자바 8 이후에는 람다식으로도 많이 표현된다. \
  자바8로 업데이트 되면서 `@FunctionalInterface`애노테이션도 붙었다.
* `java.lang` 패키지에 있다. 자바의 경우 원래부터 있던 인터페이스는 하위 호환을 위해 그대로 유지한다.
* 주로 멀티스레딩에서 스레드의 작업을 정의할 때 사용한다.
* 입력값도 없고, 반환값도 없는 함수형 인터페이스가 필요할 때 사용한다.

```java
package lambda.lambda4;

public class RunnableMain {

    public static void main(String[] args) {
        // 익명 클래스
        Runnable function1 = new Runnable() {
            @Override
            public void run() {
                System.out.println("Hello Runnable");
            }
        };
        function1.run();

        // 람다
        Runnable function2 = () -> System.out.println("Hello Runnable");
        function2.run();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.13.14.png" alt=""><figcaption></figcaption></figure>

## 특화 함수형 인터페이스&#x20;

특화 함수형 인터페이스는 의도를 명확하게 만든 조금 특별한 함수형 인터페이스이다.&#x20;

* `Pridicate` : 입력O, 반환 `boolean`
  * 조건 검사, 필터링 용도&#x20;
* `Operator(UnaryOperator, BinaryOperator)` : 입력O, 반환O&#x20;
  * 동일한 타입의 연산 수행, 입력과 같은 타입을 반환하는 용도

### Predicate&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
}
```

* 입력 값(T)을 받아서 `true` 또는 `false` 로 구분(판단)하는 함수형 인터페이스이다.
* 조건 검사, 필터링 등의 용도로 많이 사용된다.\
  (뒤에서 설명할 스트림 API에서 필터 조건을 지정할 때 자주 등장한다)

#### 용어 설명&#x20;

* "Predicate"는 수학/논리학에서 "술어"를 의미하며, 참/거짓을 판별하는 명제를 표현한다.
  * 술어: 어떤 대상의 성질이나 관계를 설명하면서, 그 설명이 참인지 거짓인지를 판단할 수 있게 해주는 표현
* test는 "시험하다"라는 의미로, 주어진 입력값이 조건을 만족하는지 테스트한다는 의미이다. 그래서 반환값이`boolean` 이다.
* 예: 숫자가 짝수인지 테스트하는 `predicate`는 조건 충족 여부를 판단한다.

```java
package lambda.lambda4;

import java.util.function.Function;
import java.util.function.Predicate;

public class PredicateMain {

    public static void main(String[] args) {
        Predicate<Integer> predicate1 = new Predicate<>() {
            @Override
            public boolean test(Integer value) {
                return value % 2 == 0;
            }
        };
        System.out.println("predicate1.test(10) = " + predicate1.test(10));

        Predicate<Integer> predicate2 = value -> value % 2 == 0;
        System.out.println("predicate2.test(10) = " + predicate2.test(10));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.22.33.png" alt=""><figcaption></figcaption></figure>

### Predicate 가 꼭 필요할까?&#x20;

`Predicate`는 입력이 `T`, 반환이 `boolean`이기 때문에, 결과적으로 `Function<T, Boolean>`으로 대체할 수 있다. 그럼에도 불고하고 `Predicate` 를 별도로 만든 이유는 다음과 같다.&#x20;

```java
Function<Integer, Boolean> f1 = value -> value % 2 == 0;
Predicate<Integer> f1 = value -> value % 2 == 0;
```

`Predicate<T>` 는 "입력 값을 받아 `true/false` 로 결과를 판단한다"라는 **의도를 명시적으로 드러내기 위해 정의**된 함수형 인터페이스이다.

물론 "`boolean` 을 반환하는 함수"라는 측면에서 보면 `Function<T, Boolean>` 지만 `Predicate<T>` 를 별도로 둠으로써 다음과 같은 이점들을 얻을 수 있다.

1. **의미의 명확성**
   1.  `Predicate<T>` 를 사용하면 "이 함수는 조건을 검사하거나 필터링 용도로 쓰인다"라는 **의도가 더 분명**해

       진다.
   2.  `Function<T, Boolean>` 을 쓰면 "이 함수는 무언가를 계산해 `Boolean`을 반환한다"라고 볼 수도 있

       지만, "조건 검사"라는 목적이 분명히 드러나지 않을 수 있다.
2. **가독성 및 유지보수성**
   1.  여러 사람과 협업하는 프로젝트에서, "조건을 판단하는 함수"는 `Predicate<T>` 라는 패턴을 사용함으로

       써 의미 전달이 명확해진다.
   2.  `boolean` 판단 로직이 들어가는 부분에서 `Predicate<T>` 를 사용하면 코드 가독성과 유지보수성이 향

       상된다.

       1. 이름도 명시적이고, 제네릭에 `<Boolean>` 을 적지 않아도 된다.

정리하면 `Function<T, Boolean>` 로도 같은 기능을 구현할 수는 있지만, **목적(조건 검사)과 용도(필터링 등)에 대해 더 분명히 표현하고, 가독성과 유지보수를 위해** `Predicate<T>` 라는 별도의 함수형 인터페이스가 마련되었다.

#### 의도가 가장 중요한 핵심&#x20;

자바가 제공하는 다양한 함수형 인터페이스들을 선택할 때는 단순히 입력값, 반환값만 보고서 선택하는게 아니라 해당 함수형 인터페이스가 제공하는 **의도**가 중요하다. 예를 들어서 조건 검사, 필터링 등을 사용한다면 `Function` 이 아니라`Predicate` 를 선택해야 한다. 그래야 다른 개발자가 "아\~ 이 코드는 조건 검사 등에 사용할 의도가 있구나" 하고 코드를 더욱 쉽게 이해할 수 있다.

### Operator

`Operator` 는 `UnaryOperator`, `BinaryOperator` 2가지 종류를 제공된다.&#x20;

#### 용어 설명&#x20;

* "Operator"라는 이름은 수학적인 연산자(Operator)의 개념에서 왔다.
* 수학에서 연산자는 보통 **같은 타입의 값들을 받아서 동일한 타입의 결과를 반환**한다.
  * 덧셈 연산자(+): `숫자 + 숫자` → `숫자`
  * 곱셈 연산자(\*): `숫자 * 숫자` → `숫자`
  * 논리 연산자(AND): `boolean AND boolean` →`boolean`
*   자바에서는 수학처럼 숫자의 연산에만 사용된다기 보다는 입력과 반환이 동일한 타입의 연산에 사용할 수 있다.

    예를 들어, 문자를 입력해서 대문자로 바꾸어 반환하는 작업도 될 수 있다.

### UnaryOperator (단항 연산)&#x20;

```java
package java.util.function;

@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    T apply(T t); // 실제 코드가 있지는 않음
}
```

* 단항 연산은 **하나의 피연산자(operand)**&#xC5D0; 대해 연산을 수행하는 것을 말한다.
  * 예) 숫자의 부호 연산(`-x` ), 논리 부정 연산(`!x` ) 등
* 입력(피연산자)과 결과(연산 결과)가 **동일한 타입**인 연산을 수행할 때 사용한다.
  * 예) 숫자 5를 입력하고 그 수를 제곱한 결과를 반환한다.
  * 예) `String` 을 입력받아 다시 `String` 을 반환하면서, 내부적으로 문자열을 대문자로 바꾼다든지, 앞뒤에 추가 문자열을 붙이는 작업을 할 수 있다.
*   `Function<T, T>` 를 상속받는데, 입력과 반환을 모두 같은 `T` 로 고정한다. 따라서 `UnaryOperator` 는 입력

    과 반환 타입이 반드시 같아야 한다.

### BinaryOperator (이항 연산)&#x20;

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T,T,T> {
    T apply(T t1, T t2); // 실제 코드가 있지는 않음
}
```

* 이항 연산은 **두 개의 피연산자(operand)**&#xC5D0; 대해 연산을 수행하는 것을 말한다.
  * 예: 두 수의 덧셈(`x + y`), 곱셈(`x * y`) 등
* **같은 타입**의 두 입력을 받아, **같은 타입**의 결과를 반환할 때 사용된다.
  * 예) `Integer` 두 개를 받아서 더한 값을 반환
  * 예) `Integer` 두 개를 받아서 둘 중에 더 큰 값을 반환
*   `BiFunction<T,T,T>` 를 상속받는 방식으로 구현되어 있는데, 입력값 2개와 반환을 모두 같은 T로 고정한다.

    따라서 `BinaryOperator` 는 모든 입력값과 반환 타입이 반드시 같아야 한다.
* `BiFunction` 은 입력 매개변수가 2개인 `Function` 이다. 뒤에서 설명한다.

```java
package lambda.lambda4;

import java.util.function.BiFunction;
import java.util.function.BinaryOperator;
import java.util.function.Function;
import java.util.function.UnaryOperator;

public class OperatorMain {

    public static void main(String[] args) {
        // Unary
        Function<Integer, Integer> square1 = x -> x * x;
        UnaryOperator<Integer> square2 = x -> x * x;
        System.out.println("square1.apply(5) = " + square1.apply(5));
        System.out.println("square2.apply(5) = " + square2.apply(5));

        // Binary
        BiFunction<Integer, Integer, Integer> addition1 = (a, b) -> a + b;
        BinaryOperator<Integer> addition2 = (a, b) -> a + b;
        System.out.println("addition1.apply(10, 10) = " + addition1.apply(10, 10));
        System.out.println("addition2.apply(10, 10) = " + addition2.apply(10, 10));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.38.51.png" alt=""><figcaption></figcaption></figure>

### Operator 를 제공하는 이유&#x20;

`Predicated` 와 마찬가지로 `Operator` 도 `Function`, `BiFunction` 으로 구현이 가능하다.&#x20;

하지만, Predicate 로 구현하는데는 2가지 이유가 있다.&#x20;

1. **의도(목적)의 명시성**
2. **가독성과 유지보수성**

정리하면

* "단항 연산(입력 하나)"이고 **타입이 동일**하다면 `UnaryOperator<T>` 를,
* "이항 연산(입력 두 개)"이고 **타입이 동일**하다면 `BinaryOperator<T>` 를 쓰는 것이 개발자의 **의도**와 **로직**을 더 명확히 표현하고, **가독성**을 높일 수 있는 장점이 있다.

## 기타 함수형 인터페이스&#x20;

### 입력 값이 2개 이상&#x20;

매개변수가 2개 이상 필요한 경우에는 `BiXxx` 시리즈를 사용하면 된다. Bi는 Binary(이항, 둘)의 줄임말이다.

* 예) `BiFunction` , `BiConsumer` , `BiPredicate`

```java
package lambda.lambda4;

import java.util.function.BiConsumer;
import java.util.function.BiFunction;
import java.util.function.BiPredicate;

public class BiMain {

    public static void main(String[] args) {
        BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
        System.out.println("add.apply(5, 10) = " + add.apply(5, 10));

        BiConsumer<String, Integer> repeat = (c, n) -> {
            for (int i = 0; i < n; i++) {
                System.out.print(c);
            }
            System.out.println();
        };
        repeat.accept("*", 10);

        BiPredicate<Integer, Integer> isGreater = (a, b) -> a > b;
        System.out.println("isGreater.test(10, 5) = " + isGreater.test(10, 5));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.43.19.png" alt=""><figcaption></figcaption></figure>

### 입력값이 3개라면?&#x20;

입력값이 3개라면 `TriXxx` 가 있으면 좋겠지만, 이런 함수형 인터페이스는 기본으로 제공하지 않는다. 보통 함수형 인터페이스를 사용할 때 3개 이상의 매개변수는 잘 사용하지 않기 때문이다.

만약 입력값이 3개일 경우라면 다음과 같이 직접 만들어서 사용하면 된다.

```java
package lambda.lambda4;

public class TriMain {

    public static void main(String[] args) {
        TruFunction<Integer, Integer, Integer, Integer> triFunction =
                (a, b, c) -> a + b + c;
        System.out.println("triFunction.apply(1,2,3) = " + triFunction.apply(1, 2, 3));
    }

    @FunctionalInterface
    interface TruFunction<A, B, C, R> {
        R apply(A a, B b, C c);
    }
}
```

### 기본형 지원 함수형 인터페이스&#x20;

다음과 같이 기본형(primitive type) 을 지원하는 함수형 인터페이스도 있다.&#x20;

```javascript
package java.util.function;

@FunctionalInterface
public interface IntFunction<R> {
    R apply(int value);
}
```

#### 기본형 지원 함수형 인터페이스가 존재하는 이유&#x20;

* 오토박싱/언박싱(auto-boxing/unboxing)으로 인한 성능 비용을 줄이기 위해
* 자바 제네릭의 한계(제네릭은 primitive 타입을 직접 다룰 수 없음)를 극복하기 위해
  *   자바의 제네릭은 기본형(primitive) 타입을 직접 다룰 수 없어서, `Function<int, R>` 같은 식으로는 선

      언할 수 없다.

```java
package lambda.lambda4;

import java.util.function.IntFunction;
import java.util.function.IntToLongFunction;
import java.util.function.IntUnaryOperator;
import java.util.function.ToIntFunction;

public class PrimitiveFunction {

    public static void main(String[] args) {
        // 기본형 매개변수
        IntFunction<String> function = x -> "숫자 : " + x;
        System.out.println("function.apply(100) = " + function.apply(100));

        // 기본형 반환
        ToIntFunction<String> toIntFunction = s -> s.length();
        System.out.println("toIntFunction = " + toIntFunction.applyAsInt("Hello"));

        // 기본형 매개변수, 기본형 반환
        IntToLongFunction intToLongFunction = x -> x * 100L;
        System.out.println("toIntFunction.applyAsInt(100) = " + intToLongFunction.applyAsLong(10));

        IntUnaryOperator intUnaryOperator = x -> x * 100;
        System.out.println("intUnaryOperator.applyAsInt(10) = " + intUnaryOperator.applyAsInt(10));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-16 13.46.00.png" alt=""><figcaption></figcaption></figure>

## 정리&#x20;

정리하자면, **람다와 함수형 인터페이스**를 제대로 활용하면 코드가 간결해지고 가독성이 높아지며, **제네릭**을 도입하면 재사용성과 타입 안정성까지 모두 확보할 수 있다. 또한 자바가 기본적으로 제공하는 다양한 함수형 인터페이스를 적극 활용하면 불필요하게 유사항 인터페이스를 여러 개 만들 필요가 없고 호환성 문제도 해결된다.&#x20;

무엇보다 "**의도를 명확하게 드러내는**" 함수형 인터페이스를 적절히 선택하는 것이 중요하다.&#x20;

조건 검사는 `Predicate`, 입력과 반환 타입이 같은 단항 연산은 `UnaryOperator`, 매개변수가 2개 이상이면서 반환 타입이 같은 연산은 `BinaryOperator` 처럼 상황에 맞는 인터페이스를 사용하면 코드의 목적이 분명해지고 유지보수성이 향상된다.&#x20;
