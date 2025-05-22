# 람다

## 람다 정의&#x20;

* 자바 8부터 도입된 람다는 자바에서 함수형 프로그래밍을 지원하기 위한 핵심 기능이다.&#x20;
  * 함수형 프로그래밍에 대해서는 뒤에서 설명한다.&#x20;
* **람다는 익명 함수**이다. 따라서 이름 없이 함수를 표현한다.&#x20;

#### 메서드나 함수는 다음과 같이 표현한다.&#x20;

```java
반환타입 메서드명(매개변수) {
    본문
}
```

#### 람다는 다음과 같이 간결하게 표현한다.

```
(매개변수) -> (본문) 
```

* 람다는 익명 함수이다. 따라서 이름이 없다.&#x20;
* 자바는 독립적인 함수를 지원하지 않으며, 메서드는 반드시 클래스나 인터페이스에 속한다.&#x20;

> 용어 - 람다 vs 람다식(Lambda Expression)&#x20;
>
> * 람다 : 익명 함수를 지칭하는 일반적인 용어이다. 쉽게 이야기해서 개념이다.&#x20;
> * 람다식 : (매개변수) -> {본문} 형태로 람다를 구현하는 구체적인 문법 표현을 지칭한다.&#x20;
>
> 쉽게 이야기해서 람다는 개념을, 람다식은 자바에서 그 개념을 구현하는 구체적인 문법을 의미한다. 람다가 넓은 의미이고, 또 실무에서 두 용어를 구분해서 사용하지는 않기 때문에 여기서는 대부분 간결하게 람다라고 하겠다.&#x20;

#### 람다는 표현이 간결하다.&#x20;

```java
Procedure procedure = new Procedure() {
    @Override
    public void run() {
        System.out.println("hello! lambda");
    }
};
```

* 익명 클래스를 사용하면 `new` 키워드, 생성할 클래스명, 메서드명, 반환 타입 등을 모두 나열해야 한다.

```java
Procedure procedure = () -> {
    System.out.println("hello! lambda");
};
```

* 람다를 사용하면 이런 부분을 모두 생략하고, 매개변수와 본문만 적으면 된다.

#### 람다는 변수처럼 다룰 수 있다.&#x20;

```java
Procedure procedure = () -> { // 람다를 변수에 담음
    System.out.println("hello! lambda");
};
procedure.run(); // 변수를 통해 람다를 실행
```

* 람다를 `procedure` 라는 변수에 담았다.
* `procedure` 변수를 통해 이곳에 담은 람다를 실행할 수 있다.

#### 람다도 클래스가 만들어지고, 인스턴스가 생성된다.&#x20;

* 람다도 익명 클래스처럼 클래스가 만들어지고, 인스턴스가 생성된다.&#x20;

다음 코드로 확인해보자.&#x20;

```java
package lambda.lambda1;

import lambda.Procedure;

public class InstanceMain {

    public static void main(String[] args) {
        Procedure procedure1 = new Procedure() {

            @Override
            public void run() {
                System.out.println("hello! lambda");
            }
        };
        System.out.println("class.class =  " + procedure1.getClass());
        System.out.println("class.instance = " + procedure1);

        Procedure procedure2 = () -> {
                System.out.println("hello! lambda");
        };
        System.out.println("class.class = " + procedure2.getClass());
        System.out.println("class.instance = " + procedure2);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 12.33.10.png" alt=""><figcaption></figcaption></figure>

* 익명 클래스의 경우 `$` 로 구분하고 뒤에 숫자가 붙는다.&#x20;
* 람다의 경우 `$$` 로 구분하고 뒤에 복잡한 문자가 붙는다.&#x20;
* 실행 환경에 따라서 결과는 다를 수 있다.

#### 정리&#x20;

* 람다를 사용하면 익명 클래스 사용의 보일러플레이트(불필요한 코드) 코드를 코게 줄이고, 간결한 코드로 생산성과 가독성을 높일 수 있다.&#x20;
* 대부분의 익명 클래스는 람다로 대체할 수 있다.&#x20;
  * 참고로 람다가 익명 클래스를 완전히 대체할 수 있는 것은 아니다. 람다와 익명 클래스의 차이는 뒤에서 따로 정리하겠다.&#x20;
* 람다를 사용할 때 `new` 키워드를 사용하지는 않지만, 람다도 익명 클래스처럼 인스턴스가 생성된다.&#x20;
* 지금의 람다를 익명 클래스의 구현을 간단히 표현할 수 있는 **문법 설탕**(Synatactic sugar, 코드를 간결하게 만드는 문법적 편의) 역할 정도로 생각하자. 람다와 익명 클래스의 차이는 뒤에서 설명한다.&#x20;

## 함수형 인터페이스

* **함수형 인터페이스**는 정확히 하나의 추상 메서드를 가지는 인터페이스를 말한다.&#x20;
* 람다는 추상 메서드가 하나인 **함수형 인터페이스에만 할당할 수 있다.**&#x20;
* 단일 추상 메서드를 줄여서 **SAM**(Single Abstract Method) 라고 한다.&#x20;
* 참고로 람다는 클래스, 추상 클래스에는 할당할 수 없다. 오직 단일 추상 메서드를 가지는 인터페이스에만 할당할 수 있다.&#x20;

#### 여러 추상 메서드&#x20;

<pre class="language-java"><code class="lang-java">package lambda.lambda1;

public interface NotSamInterface {
<strong>    void run();
</strong>    void go();
}
</code></pre>

* 인터페이스의 메서드 앞에서는 `abstract`(추상) 이 생략되어 있다.&#x20;
* 여기에는 `run()`, `go()` 두 개의 추상 메서드가 선언되어 있다.&#x20;
* 단일 추상 메서드(SAM) 가 아니다. 이 인터페이스에는 람다를 할당할 수 없다.&#x20;

#### 단일 추상 메서드&#x20;

```java
package lambda.lambda1;

public interface SamInterface {
    void run();
}
```

* 여기에는 `run()` 한 개의 추상 메서드만 선언되어 있다.
* 단일 추상 메서드(SAM)이다. 이 인터페이스에는 람다를 할당할 수 있다.

```java
package lambda.lambda1;

public class SamMain {

    public static void main(String[] args) {
        SamInterface samInterface = () -> {
            System.out.println("sam");
        };
        samInterface.run();

        // 컴파일 오류 : 추상 메서드가 2개 이상이기 때문에, 어떤 메서드를 구현한 것인지 알 수 없다.
//        NotSamInterface notSamInterface = () -> {
//            System.out.println("not sam");
//        };
//        notSamInterface.run();
//        notSamInterface.go();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 12.40.07.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 12.40.18.png" alt=""><figcaption></figcaption></figure>

* `NotSamInterface` 이 함수형 인터페이스가 아니라는 컴파일 오류 메시지가 나온다.
* 오류를 확인했으면 컴파일 오류 부분을 다시 주석 처리하자.

#### 자바는 왜 다음 코드를 허용하지 않을까?&#x20;

<pre class="language-java"><code class="lang-java">NotSamInterface notSamInterface = () -> {
<strong>    System.out.println("not sam");
</strong>}
notSamInterface.run(); // ?
notSamInterface.go(); // ?
</code></pre>

* 람다는 하나의 함수이다. 따라서 람다를 인터페이스에 담으려면 하나의 메서드(함수) 선언만 존재해야 한다.
* 인터페이스는 여러 메서드(함수)를 선언할 수 있다. 여기서는 `run()`, `go()` 두 메서드가 존재한다.
* 이 함수를 `NotSamInterface` 에 있는 `run()` 또는 `go()` 둘 중에 하나에 할당해야 하는 문제가 발생한다.

자바는 이러한 문제를 해결하기 위해, 단 하나의 추상 메서드(SAM: Single Abstract Method)만을 포함하는 **함수형 인터페이스에만 람다를 할당할 수 있도록 제한**했다.

`SamInterface`은 `run()`이라는 하나의 추상 메서드만을 포함한다. 따라서 문제 없이 람다를 할당하고 실행 가능하다.

### @FunctionalInterface&#x20;

함수형 인터페이스는 단 하나의 추상 메서드(SAM: Single Abstract Method)만을 포함하는 인터페이스이다.\
그리고 람다는 함수형 인터페이스에만 할당할 수 있다.

그런데 단 하나의 추상 메서드만을 포함한다는 것을 어떻게 보장할 수 있을까?

`@FunctionalInterface`애노테이션을 붙여주면 된다. 이 애노테이션이 있으면 단 하나의 추상 메서드가 아니면 컴파일 단계에서 오류가 발생한다. 따라서 함수형 인터페이스임을 보장할 수 있다.

```java
package lambda.lambda1;

@FunctionalInterface // 애노테이션 추가
public interface SamInterface {
    void run();
}
```

*   `@FunctionalInterface` 을 통해 함수형 인터페이스임을 선언해두면, 이후에 누군가 실수로 추상 메서드를

    추가할 때 컴파일 오류가 발생한다.

따라서 람다를 사용할 함수형 인터페이스라면 `@FunctionalInterface` 를 필수로 추가하는 것을 권장한다.

## 람다와 시그니처&#x20;

람다를 함수형 인터페이스에 할당할 때는 메서드의 형태를 정의하는 요소인 메서드 시그니처가 일치해야 한다.&#x20;

메서드 시그니처의 주요 구성 요소는 다음과 같다.&#x20;

1. 메서드 이름&#x20;
2. 매개변수의 수와 타입(순서 포함)&#x20;
3. 반환 타입

#### MyFunction 예시&#x20;

```java
@FunctionalInterface
public interface MyFunction {
    int apply(int a, int b);
}
```

```java
MyFunction myFunction = (int a, int b) -> {
    return a + b;
};
```

람다는 익명함수이므로 시그니처에서 이름은 제외하고, **매개변수, 반환 타입이 함수형 인터페이스에 선언한 메서드와 맞아야 한다.**&#x20;

이 람다는 매개변수로 `int a`, `int b`, 그리고 반환 값으로 `a+b` 인 `int` 타입을 반환하므로 시그니처가 맞다. 따라서 람다를 함수형 인터페이스에 할당할 수 있다.&#x20;

## 람다와 생략&#x20;

### 단일 표현식1&#x20;

```java
package lambda.lambda1;

import lambda.MyFunction;

public class LambdaSimple1 {

    public static void main(String[] args) {
        MyFunction function1 = (int a, int b) -> {
            return a + b;
        };
        System.out.println("function1 = " + function1.apply(1,2));

        // 단일 표현식인 경우 중괄호와 리턴 생략 가능
        MyFunction function2 = (int a, int b) -> a + b;
        System.out.println("function2 = " + function2.apply(1, 2));

        // 단일 표현식이 아닌 경우 중괄호와 리턴 생략 불가
        MyFunction function3 = (int a, int b) -> {
            System.out.println("람다 실행");
            return a + b;
        };
        System.out.println("function3 = " + function3.apply(1,2));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 12.51.43.png" alt=""><figcaption></figcaption></figure>

* 단일 표현식의 경우 중괄호와 리턴을 함께 생략할 수 있다.&#x20;

#### 표현식(expression) 이란?&#x20;

* 하나의 값으로 평가되는 코드 조각을 의미한다.&#x20;
* 표현식은 산술 논리 표현식, 메서드 호출, 객체 생성등이 있다.&#x20;
  * 예) `x + y`, `price * quantity`, `calculateTotal()`, `age >= 18`&#x20;
* 표현식이 아닌것은 제어문, 메서드 선언 같은 것이 있다.
  * 예) `if (condition) { }`

#### 람다 - 단일 표현식(single expression) 인 경우&#x20;

* 중괄호 `{}` 와 `return` 키워드를 함께 생략할 수 있음
  * 표현식의 결과가 자동으로 반환값이 됨
* 중괄호를 사용하는 경우에는 반드시 `return` 문을 포함해야 한다.
  * `return` 문을 명시적으로 포함하는 경우 중괄호를 사용해야 한다.
  * 반환 타입이 `void` 인 경우 `return` 생략 가능

#### 단일 표현식이 아닌 경우&#x20;

```java
(int a, int b) -> {
    System.out.println("람다 실행");
    return a + b;
};
```

* 단일 표현식이 아닌 경우 중괄호(`{}` )를 생략할 수 없다. 이 경우 반환 값이 있으면 `return` 문도 포함해야 한다.

### 단일 표현식2&#x20;

```java
package lambda.lambda1;

import lambda.Procedure;

public class LambdaSimple2 {

    public static void main(String[] args) {
        Procedure procedure1 = () -> {
            System.out.println("hello! lambda");
        };
        procedure1.run();

        // 단일 표현식의 경우 중괄호 생락 가능
        Procedure procedure2 = () -> System.out.println("hello! lambda");
        procedure2.run();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 12.56.20.png" alt=""><figcaption></figcaption></figure>

매개변수와 반환 값이 없는 경우도 동일하다.\
`Procedure.run()` 의 경우 반환 타입이 `void` 이기 때문에 중괄호를 사용해도 `return` 은 생략할 수 있다.

### 타입 추론&#x20;

다음과 같은 람다 코드를 작성한다고 생각해보자.&#x20;

```java
MyFunction function1 = (int a, int b) -> a + b;
```

* 여기서 매개변수에 해당하는 `(int a, int b)` 부분을 집중해보자.&#x20;
* 함수형 인터페이스인 `MyFunction`의 `apply()` 메서드를 보면 이미 `int a, int b` 로 매개변수의 타입이 정의되어 있다.
* 따라서 이 정보를 사용하면 람다의 `(int a, int b)` 에서 타입 정보를 생략할 수 있다.

```java
@FunctionalInterface
public interface MyFunction {
    int apply(int a, int b);
}
```

```java
package lambda.lambda1;

import lambda.MyFunction;

public class LambdaSimple3 {

    public static void main(String[] args) {
        // 타입 생략 전
        MyFunction myFunction1 = (int a, int b) -> a + b;

        // MyFunction 타입을 통해 타입 추론 가능, 람다는 타입 생략 가능
        MyFunction myFunction2 = (a, b) -> a + b;

        int result = myFunction2.apply(1, 2);
        System.out.println("result = " + result);
    }
}
```

* 자바 컴파일러는 람다가 사용되는 함수형 인터페이스의 메서드 타입을 기반으로 람다의 매개변수와 반환값의 타입을 추론한다. 따라서 람다는 타입을 생략할 수 있다.&#x20;
* 반환 타입은 문법적으로 명시할 수 없다. 대신에 컴파일러가 자동으로 추론한다.&#x20;

### 매개변수 괄호 생략&#x20;

```java
package lambda.lambda1;

public class LambdaSimple4 {

    public static void main(String[] args) {
        MyCall Call1 = (int value) -> value * 2;  // 기본
        MyCall Call2 = (value) -> value * 2;  // 타입 추론    
        MyCall Call3 = value -> value * 2;    // 매개변수 1개, () 생략 가능

        System.out.println("Call3 = " + Call3.call(10));
    }

    interface MyCall {
        int call(int value);
    }
}
```

* 매개변수가 정확히 하나이면서, 타입을 생략하고, 이름만 있는 경우 소괄호`()` 를 생략할 수 있다.
* 매개변수가 없는 경우에는 `()` 가 필수이다.
* 매개변수가 둘 이상이면 `()` 가 필수이다.

#### 정리&#x20;

* **매개변수 타입** : 생략 가능하지만 필요하다면 명시적으로 작성할 수 있다.&#x20;
* **반환 타입** : 문법적으로 명시할 수 없고, 식의 결과를 보고 컴파일러가 항상 추론한다.&#x20;
* 람다는 보통 간략하게 사용하는 것을 권장한다.&#x20;
  * 단일 표현식이면 중괄호와 리턴을 생략하자.&#x20;
  * 타입 추론을 통해 매개변수의 타입을 생략하자. (컴파일러가 추론할 수 있다면, 생략하자)&#x20;

## 람다의 전달&#x20;

### 람다를 변수에 대입하기&#x20;

```java
package lambda.lambda2;

import lambda.MyFunction;

public class LambdaPassMain1 {

    // 1. 람다를 변수에 대입하기.
    public static void main(String[] args) {
        MyFunction add = (a, b) -> a + b;
        MyFunction sub = (a, b) -> a - b;

        System.out.println("add.apply(1,2) : " + add.apply(1, 2));
        System.out.println("sub.apply(1,2) : " + sub.apply(1, 2));

        MyFunction cal = add;
        System.out.println("cal(add).apply(1,2) : " + cal.apply(1, 2));

        cal = sub;
        System.out.println("cal(add).apply(1,2) : " + cal.apply(1, 2));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 13.14.59.png" alt=""><figcaption></figcaption></figure>

```java
MyFunction add = (a, b) -> a + b;
```

* 이 대입식에서 변수 `add` 의 타입은 `MyFuncion` 함수형 인터페이스이다. 따라서 `MyFunction` 형식에 맞는 람다를 대입할 수 잇다. (메서드 시그니처가 일치한다)

#### 람다의 대입&#x20;

```java
MyFunction add = (a, b) -> a + b;
MyFunction cal = add;
```

```java
// 람다의 대입 분석
MyFunction add = (a, b) -> a + b; // 1. 람다 인스턴스 생성
MyFunction add = x001; // 2. 참조값 반환, add에 x001 대입

MyFunction cal = add;
MyFunction cal = x001; // 3. cal에 참조값 대입
```

함수형 인터페이스로 선언한 변수에 람다를 대입하는 것은 **람다 인스턴스의 참조값을 대입하는 것이다.**\
**(**&#xC774;해가 잘 안된다면 익명 클래스의 인스턴스를 생성하고 대입한다고 생각해보자)

참고로 함수형 인터페이스도 인터페이스이다.

### 람다를 메서드(함수) 에 전달하기&#x20;

```java
package lambda.lambda2;

import lambda.MyFunction;

// 2. 람다를 메서드(함수) 에 전달하기
public class LambdaPassMain2 {

    public static void main(String[] args) {
        MyFunction add = (a, b) -> a + b;
        MyFunction sub = (a, b) -> a - b;

        System.out.println("변수를 통해 전달");

        calculate(add);
        calculate(sub);

        System.out.println("람다를 직접 전달");
        calculate((a, b) -> a + b);
        calculate((a, b) -> a - b);
    }

    static void calculate(MyFunction function) {
        int a = 1;
        int b = 2;

        System.out.println("계산 시작");
        int result = function.apply(a, b);
        System.out.println("계산 결과 = " + result);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 13.17.54.png" alt=""><figcaption></figcaption></figure>

```java
void calculate(MyFunction function)
```

* `calculate()` 메서드의 매개변수는 `MyFunction` 함수형 인터페이스이다. 따라서 람다를 전달할 수 있다.&#x20;

#### 람다를 변수에 담은 후에 매개변수에 전달

```java
MyFunction add = (a, b) -> a + b;
calculate(add);
```

```java
// 람다를 변수에 담은 후에 매개변수에 전달 분석
MyFunction add = (a, b) -> a + b; // 1. 람다 인스턴스 생성
MyFunction add = x001; // 2. 참조값 반환
add = x001; // 3. 참조값 대입

calculate(add);
calculate(x001);

// 메서드 호출, 매개변수에 참조값 대입
void calculate(MyFunction function = x001)
```

#### 람다를 직접 전달&#x20;

```java
calculate((a, b) -> a + b);
```

```java
// 람다를 직접 전달 분석
calculate((a, b) -> a + b); // 1. 람다 인스턴스 생성
calculate(x001); // 2. 참조값 반환 및 매개변수에 전달

// 메서드 호출, 매개변수에 참조값 대입
void calculate(MyFunction function = x001)
```

### 람다를 반환하기&#x20;

```java
package lambda.lambda2;

import lambda.MyFunction;

public class LambdaPassMain3 {

    public static void main(String[] args) {
        MyFunction add = getOperation("add");
        System.out.println("add.apply(1,2) = " + add.apply(1,2));
        MyFunction sub = getOperation("sub");
        System.out.println("sub.apply(1,2) = " + sub.apply(1,2));
        MyFunction xxx = getOperation("xxx");
        System.out.println("xxx.apply(1,2) = " + xxx.apply(1,2));
    }

    // 람다를 반환하는 메서드
    static MyFunction getOperation(String operator) {
        switch (operator) {
            case "add":
                return (a, b) -> a + b;
            case "sub":
                return (a, b) -> a - b;
            default:
                return (a, b) -> 0;
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 13.20.30.png" alt=""><figcaption></figcaption></figure>

```java
MyFunction getOperation(String operator){}
```

* `getOperation` 메서드는 반환 타입이 `MyFunction` 함수형 인터페이스이다. 따라서 람다를 반환할 수 있다.

#### 분석&#x20;

```java
// 1. 메서드를 호출한다.
MyFunction add = getOperation("add");

// 2. getOperation() 메서드 안에서 다음 코드가 호출된다.
MyFunction getOperation(String operator) {} // 반환 타입이 MyFunction 함수형 인터페이
스이다.
return (a, b) -> a + b; // 2-1. 람다 인스턴스를 생성한다.
return x001; // 2-2. 람다 인스턴스의 참조값을 반환한다.

// 3. main 메서드로 람다 인스턴스의 참조값이 반환된다.
MyFunction add = x001; // 3-1. 람다 인스턴스의 참조값을 add에 대입한다.
```

## 고차 함수&#x20;

### 람다의 전달 정리&#x20;

람다는 함수형 인터페이스를 구현한 익명 클래스의 인스턴스와 같은 개념으로 이해하면 된다. 즉, 람다를 변수에 대입한다는 것은 **람다 인스턴스의 참조값을 대입**하는 것이고, 람다를 메서드(함수)의 매개변수나 반환값으로 넘긴다는 것 역시 **람다 인스턴스의 참조값을 전달, 반환하는 것**이다.&#x20;

*   **람다를 변수에 대입**: `MyFunction add = (a, b) -> a + b;` 처럼 함수형 인터페이스 타입의 변수에 람

    다 인스턴스의 참조를 대입한다.
*   **람다를 메서드 매개변수에 전달**: 메서드 호출 시 람다 인스턴스의 참조를 직접 넘기거나, 이미 람다 인스턴스를 담

    고 있는 변수를 전달한다.

```java
// 변수에 담은 후 전달
MyFunction add = (a, b) -> a + b;
calculate(add);

// 직접 전달
calculate((a, b) -> a + b);
```

*   **람다를 메서드에서 반환**: `return (a, b) -> a + b;` 처럼 함수형 인터페이스 타입을 반환값으로 지정해

    람다 인스턴스의 참조를 돌려줄 수 있다.

### 고차 함수(Higher-Order Function)&#x20;

고차 함수는 함수를 값처럼 다루는 함수를 뜻한다.&#x20;

일반적으로 다음 두 가지 중 하나를 만족하면 고차 함수라 한다.&#x20;

* 함수를 인자로 받는 함수(메서드)&#x20;
* 함수를 리턴하는 함수(메서드)&#x20;
* 즉, 매개변수나 반환값에 함수(또는 람다)를 활용하는 함수가 고차 함수에 해당한다.
* 자바에서 람다(익명 함수)는 함수형 인터페이스를 통해서만 전달할 수 있다.
*   **자바에서 함수를 주고받는다는 것**은 "함수형 인터페이스를 구현한 어떤 객체(람다든 익명 클래스든)를 주고받는

    것"과 동의어이다. (함수형 인터페이스는 인터페이스이므로 익명 클래스, 람다 둘다 대입할 수 있다. 하지만 실질

    적으로 함수형 인터페이스에는 람다를 주로 사용한다.)

> 용어 - 고차 함수&#x20;
>
> **고차 함수(Higher-Order Function)**&#xB77C;는 이름은 **함수를 다루는 추상화 수준**이 더 높다는 데에서 유래했다.
>
> * 보통의 (일반적인) 함수는 **데이터(값)**&#xB97C; 입력으로 받고, 값을 반환한다.
> * 이에 반해, 고차 함수는 **함수를 인자로 받거나 함수를 반환**한다.
> * 쉽게 이야기하면 일반 함수는 값을 다루지만, 고차 함수는 함수 자체를 다룬다.
>
> 즉, "값"을 다루는 것을 넘어, "함수"라는 개념 자체를 값처럼 다룬다는 점에서 **추상화의 수준(계층, order)이 한 단계 높아진다**고 해서 Higher-Order(더 높은 차원의) 함수라고 부른다.&#x20;
