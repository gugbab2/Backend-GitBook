# 람다가 필요한 이유

## 람다가 필요한 이유1

람다를 본격적으로 학습하기 전에, 먼저 람다가 필요한 이유에 대해서 알아보자. 람다를 이해하려면 먼저 내부 클래스에 대한 개념을 확실히 알아두어야 한다.&#x20;

#### 리펙토링 전&#x20;

```java
package lambda.start;

public class Ex0Main {

    public static void helloJava() {
        System.out.println("프로그램 시작");
        System.out.println("Hello Java");
        System.out.println("프로그램 종료");
    }

    public static void helloSpring() {
        System.out.println("프로그램 시작");
        System.out.println("Hello Spring");
        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {
        helloJava();
        helloSpring();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 10.16.10.png" alt=""><figcaption></figcaption></figure>

코드 중복이 보일 것이다. 코드를 리펙토링 해서 코드의 중복을 제거해보자.&#x20;

`helloJava()`, `helloSpring()` 메서드를 하나로 통합하면 된다.&#x20;

#### 리펙토링 후&#x20;

```java
package lambda.start;

public class Ex0RefMain {

    public static void hello(String str) {
        System.out.println("프로그램 시작");
        System.out.println(str);
        System.out.println("프로그램 종료");
    }

    public static void main(String[] args) {
        hello("Hello Java");
        hello("Hello Spring");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 10.17.26.png" alt=""><figcaption></figcaption></figure>

코드를 분석해보자.&#x20;

기존 코드에서 변하는 부분과 변하지 않는 부분을 분리해야 한다.&#x20;

여기서 핵심은 **변하는 부분과 변하지 않는 부분을 분리**하는 것이다.&#x20;

**변하지 않는 부분은 그대로 유지하고 변하는 부분을 어떻게 해결**할 것 인가에 집중하면 된다.&#x20;

단순한 문제지만, **프로그램에서 중복을 제거하고, 좋은 코드를 유지하는 핵심은 변하는 부분과 변하지 않는 부분을 분리하는 것이다.** 이렇게 변하는 부분과 변하지 않는 부분을 분리하고, 변하는 부분을 외부에서 전달 받으면 메서드(함수) 의 재사용성을 높일 수 있다.&#x20;

### 값 매개변수화(Value Parameterization)&#x20;

문자값(**Value**), 숫자값(**Value**)처럼 구체적인 값을 메서드(함수) 안에 두는 것이 아니라, **매개변수**(파라미터)를 통해

외부에서 전달 받도록 해서, 메서드의 동작을 달리하고, 재사용성을 높이는 방법을 **값 매개변수화(Value Parameterization)**&#xB77C; 한다.

## 람다가 필요한 이유2

#### 리펙토링 전&#x20;

```java
package lambda.start;

import lambda.Procedure;

import java.util.Random;

public class Ex1RefMain {

    public static void hello(Procedure procedure) {
        long startNs = System.nanoTime();

        procedure.run();

        long endNs = System.nanoTime();
        System.out.println("실행 시간: " + (endNs - startNs) + "ns");
    }

    static class Dice implements Procedure {
        @Override
        public void run() {
            int randomValue = new Random().nextInt(6) + 1;
            System.out.println("주사위 = " + randomValue);
        }
    }

    static class Sum implements Procedure {
        @Override
        public void run() {
            for (int i = 1; i <= 3; i++) {
                System.out.println("i = " + i);
            }
        }
    }

    public static void main(String[] args) {
        Procedure dice = new Dice();
        Procedure sum = new Sum();

        hello(dice);
        hello(sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 10.25.56.png" alt=""><figcaption></figcaption></figure>

이 코드를 이전에 리팩토링 한 예와 같이 하나의 메서드에서 실행할 수 있도록 리펙토링 해보자.&#x20;

참고로 이전 문제는 변하는 문자열 값을 매개변수화 해서 외부에서 전달하면 됬다. 이번에는 문자열 같은 단순한 값이 아니라 코드 조각을 전달해야 한다.&#x20;

#### 리펙토링 후&#x20;

```java
package lambda;

public interface Procedure {
    void run();
}
```

```java
package lambda.start;

import lambda.Procedure;

import java.util.Random;

public class Ex1RefMain {

    public static void hello(Procedure procedure) {
        long startNs = System.nanoTime();

        procedure.run();

        long endNs = System.nanoTime();
        System.out.println("실행 시간: " + (endNs - startNs) + "ns");
    }

    static class Dice implements Procedure {
        @Override
        public void run() {
            int randomValue = new Random().nextInt(6) + 1;
            System.out.println("주사위 = " + randomValue);
        }
    }

    static class Sum implements Procedure {
        @Override
        public void run() {
            for (int i = 1; i <= 3; i++) {
                System.out.println("i = " + i);
            }
        }
    }

    public static void main(String[] args) {
        Procedure dice = new Dice();
        Procedure sum = new Sum();

        hello(dice);
        hello(sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-15 10.25.40.png" alt=""><figcaption></figcaption></figure>

코드를 분석해보자.&#x20;

여기서는 단순히 데이터를 전달하는 수준을 넘어서, 코드 조각을 전달해야 한다.&#x20;

**어떻게 외부에서 코드 조각을 전달할 수 있을까?**&#x20;

코드 조각은 보통 메서드(함수)에 정의한다. 따라서 코드 조각을 전달하기 위해서는 메서드가 필요하다. \
그런데 지금까지 학습한 내용으로는 메서드만 전달할 수 있는 방법이 없다. 대신에 인스턴스를 전달하고, 인스턴스에 있는 메서드를 호출하면 된다.&#x20;

이 문제를 해결하기 위한 인터페이스(`Procedure`)를 정의하고 구현 클래스(`Dice`, `Sum`)를 만들었다.

#### 리펙토링 완료

```java
public static void hello(Procedure procedure) {
    long startNs = System.nanoTime();

    procedure.run();

    long endNs = System.nanoTime();
    System.out.println("실행 시간: " + (endNs - startNs) + "ns");
}
```

* 리펙토링한 `hello()` 메서드에는 `Procedure procedure` 매개변수(파라미터)를 통해 인스턴스를 전달할 수 있다. 이 인스턴스의 `run()` 메서드르르 실행하면 필요한 코드 조각을 실행할 수 있다.&#x20;

```java
public static void main(String[] args) {
    Procedure dice = new Dice();
    Procedure sum = new Sum();

    hello(dice);
    hello(sum);
}
```

* 이때 다형성을 활용해서 외부에서 전달되는 인스턴스에 따라 각각 다른 코드 조각이 실행된다.

### 동작 매개변수화(Behavior Parameterization)&#x20;

#### 값 매개변수화(Value Parameterization)&#x20;

*   문자값(**Value**), 숫자값(**Value**)처럼 구체적인 값을 메서드(함수) 안에 두는 것이 아니라, **매개변수**(파라미터)를

    통해 외부에서 전달 받도록 해서, 메서드의 동작을 달리하고, 재사용성을 높이는 방법을 **값 매개변수화**라 한다.
* 값 매개변수화, 값 파라미터화 등으로 부른다.

#### 동작 매개변수화(Behavior Parameterization)&#x20;

*   코드 조각(코드의 동작 방법, 로직, **Behavior**)을 메서드(함수) 안에 두는 것이 아니라, **매개변수**(파라미터)를 통

    해서 외부에서 전달 받도록 해서, 메서드의 동작을 달리하고, 재사용성을 높이는 방법을 동작 매개변수화라 한다.
* 동작 매개변수화, 동작 파라미터화, 행동 매개변수화(파라미터화), 행위 파라미터화 등으로 부른다.

자바에서 동작 매개변수화를 하려면 클래스를 정의하고 해당 클래스를 인스턴스로 만들어서 전달해야 한다.\
자바8에서 등장한 람다를 사용하면 코드 조각을 매우 편리하게 전달 할 수 있는데, 람다를 알아보기 전에 기존에 자바로 할 수 있는 다양한 방법들을 먼저 알아보자.

## 람다가 필요한 이유3&#x20;

### 익명 클래스 사용1&#x20;

이번에는 익명 클래스를 사용해서 같은 기능을 구현해보자.

```java
package lambda.start;

import lambda.Procedure;

import java.util.Random;

public class Ex1RefMainV2 {

    public static void hello(Procedure procedure) {
        long startNs = System.nanoTime();

        procedure.run();

        long endNs = System.nanoTime();
        System.out.println("실행 시간: " + (endNs - startNs) + "ns");
    }

    public static void main(String[] args) {
        Procedure dice = new Procedure() {
            @Override
            public void run() {
                int randomValue = new Random().nextInt(6) + 1;
                System.out.println("주사위 = " + randomValue);
            }
        };
        Procedure sum = new Procedure() {
            @Override
            public void run() {
                for (int i = 1; i <= 3; i++) {
                    System.out.println("i = " + i);
                }
            }
        };

        hello(dice);
        hello(sum);
    }
}
```

### 익명 클래스 사용2 - 참조값 직접 전달&#x20;

익명 클래스의 참조값을 지역 변수에 담아둘 필요 없이, 매개변수에 직접 전달해보자.&#x20;

```java
package lambda.start;

import lambda.Procedure;

import java.util.Random;

public class Ex1RefMainV3 {

    public static void hello(Procedure procedure) {
        long startNs = System.nanoTime();

        procedure.run();

        long endNs = System.nanoTime();
        System.out.println("실행 시간: " + (endNs - startNs) + "ns");
    }

    public static void main(String[] args) {

        hello(new Procedure() {
            @Override
            public void run() {
                int randomValue = new Random().nextInt(6) + 1;
                System.out.println("주사위 = " + randomValue);
            }
        });

        hello(new Procedure() {
            @Override
            public void run() {
                for (int i = 1; i <= 3; i++) {
                    System.out.println("i = " + i);
                }
            }
        });
    }
}
```

### 람다(lambda)&#x20;

자바에서 메서드의 매개변수에 인수를 전달할 수 있는 것은 크게 2가지이다.&#x20;

* `int`, `double` 과 같은 기본형 타입&#x20;
* `Procedure`, `Member` 와 같은 참조형 인스턴스 타입&#x20;

결국 메서드에 인수로 전달할 수 있는 것은 간단한 값이나, 인스턴스의 참조이다.&#x20;

지금처럼 코드 조각을 전달하기 위해 클래스를 정의하고 메서드를 만들고 또 인스턴스까지 생성하는 복잡한 과정을 거쳐야 할까? \
생각해보면 클래스나 인스턴스와 관계 없이 다음과 같이 직접 코드 블럭을 전달할 수 있다면 더 간단하지 않을까?

### 리팩토링 - 람다&#x20;

```java
package lambda.start;

import lambda.Procedure;

import java.util.Random;

public class Ex1RefMainV4 {

    public static void hello(Procedure procedure) {
        long startNs = System.nanoTime();

        procedure.run();

        long endNs = System.nanoTime();
        System.out.println("실행 시간: " + (endNs - startNs) + "ns");
    }

    public static void main(String[] args) {

        hello(() -> {
            int randomValue = new Random().nextInt(6) + 1;
            System.out.println("주사위 = " + randomValue);
        });

        hello(() -> {
            for (int i = 1; i <= 3; i++) {
                System.out.println("i = " + i);
            }
        });
    }
}
```

*   람다를 사용한 코드를 보면 클래스나 인스턴스를 정의하지 않고, 매우 간편하게 코드 블럭을 직접 정의하고, 전달

    하는 것을 확인할 수 있다.

## 함수 vs 메서드&#x20;

함수(Function)와 메서드(Method)는 둘 다 어떤 작업(로직)을 수행하는 코드의 묶음이다. \
하지만 일반적으로 **객체지향 프로그래밍(OOP)** 관점에서 다음과 같은 차이가 있다. 먼저 간단한 예시를 알아보자.

#### 객체(클래스)와의 관계&#x20;

* 함수(Function)&#x20;
  * 독립적으로 존재하며, 클래스(객체) 와 직접적인 연관이 없다.&#x20;
  * 객체지향 언어가 아닌 C 등의 절차적 언어에서는 모든 로직이 함수 단위로 구성된다.&#x20;
  *   객체지향 언어라 하더라도, 예를 들어 Python이나 JavaScript처럼 클래스 밖에서도 정의할 수 있는 "함수"

      개념을 지원하는 경우, 이를 그냥 함수라고 부른다.
* 메서드(Method)
  * **클래스(또는 객체)에 속해 있는 "함수"이다.**
  * 객체의 상태(필드, 프로퍼티 등)에 직접 접근하거나, 객체가 제공해야 할 기능을 구현할 수 있다.
  *   Java, C++, C#, Python 등 대부분의 객체지향 언어에서 **클래스 내부에 정의된 함수**는 보통 "메서드"라고

      부른다.
