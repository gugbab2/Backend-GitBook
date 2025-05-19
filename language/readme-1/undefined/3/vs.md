# 람다 vs 익명 클래스

## 람다 vs 익명 클래스 1&#x20;

자바에서 익명 클래스와 람다 표현식은 모두 간단하게 기능을 구현하거나, 일회성으로 사용할 객체를 만들 때 유용하지만, 그 사용 방식과 의도에는 차이가 있다.

### 1. 문법 차이&#x20;

* 익명 클래스&#x20;
  * 익명 클래스는 클래스를 선언하고 즉시 인스턴스를 생성하는 방식이다.&#x20;
  * 반드시 `new 인스턴스명() {...}` 형태로 작성해야 하며, 메서드를 오버라이드해서 구현한다.&#x20;
  * 익명 클래스도 하나의 인스턴스이다.&#x20;

```java
// 익명 클래스 사용 예
Button button = new Button();
button.setOnClickListener(new OnClickListener() {
    @Override
    public void onClick(View v) {
        System.out.println("버튼 클릭");
    }
});
```

* 람다 표현식&#x20;
  * 람다 표현식은 함수를 간결하게 표현할 수 있는 방식이다.&#x20;
  * 함수형 인터페이스(메서드가 하나인 인터페이스) 를 간단히 구현할 때 주로 사용한다.&#x20;

```java
// 람다 표현식 사용 예
Button button = new Button();
button.setOnClickListener(v -> System.out.println("버튼 클릭"));
```

### 2. 코드의 간결함&#x20;

* 익명 클래스는 문법적으로 더 복잡하고 장황하다. 때문에, 코드의 양이 상대적으로 많다.&#x20;
* 람다 표현식은 간결하며, 불필요한 코드를 최소화한다. 또한 많은 생략 기능을 지원해서 핵심 코드만 작성할 수 있다.&#x20;

### 3. 상속 관계&#x20;

*   **익명 클래스**는 일반적인 클래스처럼 다양한 인터페이스와 클래스를 구현하거나 상속할 수 있다. 즉, 여러 메서드

    를 가진 인터페이스를 구현할 때도 사용할 수 있다.
* **람다 표현식**은 메서드를 딱 하나만 가지는 **함수형 인터페이스**만을 구현할 수 있다.
  *   람다 표현식은 **클래스를 상속**할 수 없다. 오직 함수형 인터페이스만 구현할 수 있으며, **상태(필드, 멤버 변**

      **수)**&#xB098; 추가적인 메서드 오버라이딩은 불가능하다.
  *   람다는 단순히 함수를 정의하는 것으로, 상태나 추가적인 상속 관계를 필요로 하지 않는 상황에서만 사용할

      수 있다.

### 4. 호환성&#x20;

* 익명 클래스는 자바의 오래된 버전에서도 사용할 수 있다.&#x20;
* 람다 표현식은 자바 8부터 도입되었기 때문에 그 이전 버전에서는 사용할 수 없다.&#x20;

### 5. this 키워드의 의미&#x20;

* 익명 클래스 내부에서 `this` 는 익명 클래스 자신을 가리킨다. 외부 클래스와 별도의 컨텍스트를 가진다.&#x20;
* 람다 표현식에서 `this` 는 람다를 선언한 클래스의 인스턴스를 가리킨다. 즉, 람다 표현식은 별도의 컨텍스트를 가지는 것이 아니라, 람다를 선언한 클래스의 컨텍스트를 유지한다.&#x20;
  * 쉽게 말해, 람다 내부의 `this` 는 **람다가 선언된 외부 클래스**의 `this` 와 동일하다.&#x20;

```java
package lambda.lambda6;

public class OuterMain {

    private String message = "외부 클래스";

    public void execute() {
        // 1. 익명 클래스 예시
        Runnable anonymous = new Runnable() {

            private String message = "익명 클래스";

            @Override
            public void run() {
                // 익명 클래스에서의 this 는 익명 클래스의 인스턴스를 가리킨다.
                System.out.println("[익명 클래스] this : " + this);
                System.out.println("[익명 클래스] this.class : " + this.getClass());
                System.out.println("[익명 클래스] this.message : " + this.message);

            }
        };

        // 2. 람다 예시
        Runnable lambda = () -> {
            // 람다에서의 this 는 람다가 선언된 클래스의 인스턴스(즉, 외부 클래스) 가리킴
            System.out.println("[익명 클래스] this : " + this);
            System.out.println("[익명 클래스] this.class : " + this.getClass());
            System.out.println("[익명 클래스] this.message : " + this.message);
        };
        System.out.println("--------------------------");
        anonymous.run();
        System.out.println("--------------------------");
        lambda.run();
    }

    public static void main(String[] args) {
        OuterMain outer = new OuterMain();
        System.out.println("[외부 클래스] : " + outer );
        outer.execute();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.59.30.png" alt=""><figcaption></figcaption></figure>

### 6. 캡처링(capturing)&#x20;

* 익명 클래스&#x20;
  * 익명 클래스는 외부 변수에 접근할 수 있지만, 지역 변수는 반드시 `final` 혹은 **사실상 final** 인 변수만 캡처할 수 있다.&#x20;
* 람다 표현식&#x20;
  * 람다도 익명 클래스와 같이 캡처링을 지원한다. 지역 변수는 반드시 `final` 혹은 **사실상 final** 인 변수만 캡처할 수 있다.&#x20;

```java
package lambda.lambda6;

public class CaptureMain {

    public static void main(String[] args) {

        final int final1 = 10; // 명시적으로 final
        int final2 = 20; // 사실상(final): 재할당(값 변경) 없음
        int changedVar = 30; // 값이 변경되는 변수

        // 1. 익명 클래스에서의 캡처
        Runnable anonymous = new Runnable() {
            @Override
            public void run() {
                System.out.println("익명 클래스 - final1: " + final1);
                System.out.println("익명 클래스 - final2: " + final2);
                // 컴파일 오류
                //System.out.println("익명 클래스 - changedVar: " + changedVar);
            }
        };

        // 2. 람다 표현식에서의 캡처
        Runnable lambda = () -> {
            System.out.println("람다 - final1: " + final1);
            System.out.println("람다 - final2: " + final2);
            // 컴파일 오류
            //System.out.println("람다 - changedVar: " + changedVar);
        };

        // changedVar 값을 변경해서 "사실상 final"이 아님
        changedVar++;

        // 실행
        anonymous.run();
        lambda.run();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 17.00.05.png" alt=""><figcaption></figcaption></figure>

## 람다 vs 익명 클래스2&#x20;

### 7. 생성 방식&#x20;

생성 방식은 자바 내부 동작 방식으로 크게 중요하지 않다. 이런게 있구나 하고 넘어가자.&#x20;

* 익명 클래스&#x20;
  * 익명 클래스는 새로운 클래스를 정의하여 객체를 생성하는 방식이다. 즉, 컴파일 시 새로운 내부 클래스로 변환된다. 예를 들어, `OuterClass$1.class` 와 같이 이름이 지정된 클래스 파일이 생성된다.
  * 이 방식은 클래스가 메모리 상에서 별도로 관리되므로, 메모리 상에 약간의 추가 오버헤드가 생성된다.&#x20;
* 람다
  * 람다는 내부적으로 invokeDynamic 이라는 매커니즘을 사용하여 컴파일 타임에 실제 클래스 파일을 생성하지 않고, 런타임 시점에서 동적으로 필요한 코드를 처리한다.&#x20;
  * 따라서 람다는 익명 클래스보다 메모리 관리가 더 효율적이며, 생성된 클래스 파일이 없으므로 클래스 파일 관리의 복잡성도 줄어든다.&#x20;

#### 쉽게 정리하면 다음과 같다.&#x20;

* 익명 클래스&#x20;
  * 컴파일 시 실제로 `OuterClass$1.class` 와 같은 클래스 파일이 생성된다.&#x20;
  * 일반적인 클래스와 같은 방식으로 작동한다.&#x20;
  * 해당 클래스 파일 파일을 JVM 에 불러서 사용하는 과정이 필요하다.&#x20;
* 람다&#x20;
  * 컴파일 시점에 별도의 클래스 파일이 생성되지 않는다.&#x20;
  * 자바를 실행하는 시점에 동적으로 필요한 코드를 처리한다.&#x20;

#### 원본 코드&#x20;

* 람다가 포함된 코드가 있다면 자바는 다음과 같이 컴파일 한다.&#x20;

```java
public class FunctionMain {
    public static void main(String[] args) {
        Function<String, Integer> function = x -> x.length();
        System.out.println("function1 = " + function.apply("hello"));
    }
}
```

#### 컴파일 코드&#x20;

```java
public class FunctionMain {
    public static void main(String[] args) {
        Function<String, Integer> function = 람다 인스턴스 생성(구현 코드는 lambda1()연결)
        System.out.println("function1 = " + function.apply("hello"));
    }
    
    // 람다를 private 메서드로 추가
    private Integer lambda1(String x) {
        return x.length();
    }
}
```

* 컴파일 단계에서 람다를 별도의 클래스로 만드는 것이 아니라, `private` 메서드로 만들어 숨겨둔다.&#x20;
* 그리고 실행 시점에 동적으로 람다 인스턴스를 생성하고, 해당 인스턴스의 구현 코드로 앞서 만든 `lambda1()` 메서드가 호출되도록 연결한다.&#x20;

### 8. 상태 관리&#x20;

* 익명 클래스&#x20;
  * 익명 클래스는 인스턴스 내부에 상태(필드, 멤버변수) 를 가질 수 있다.\
    예를 들어, 익명 클래스 내부에 멤버 변수를 선언하고 해당 값을 변경하거나 상태를 관리할 수 있다.
  * 이처럼 상태를 필요로 하는 경우 익명 클래스가 유리하다.&#x20;
* 람다&#x20;
  * 클래스는 그 내부에 상태와 기능을 가진다. 반면에 함수는 그 내부에 상태를 가지지 않고 기능만 제공한다.&#x20;
  * 함수인 람다는 기본적으로 상태가 없으므로 스스로 상태를 유지하지는 않는다.&#x20;

### 9. 익명 클래스와 람다의 용도 구분&#x20;

* 익명 클래스&#x20;
  * 상태를 유지하거나 다중 메서드를 구현할 필요가 있는 경우&#x20;
  * 기존 클래스 또는 인터페이스를 상속하거나 구현할 때&#x20;
  * 복잡한 인터페이스 구현이 필요할 때&#x20;
* 람다&#x20;
  * 상태를 유지할 필요가 없고, 간결함이 중요한 경우&#x20;
  * 단일 메서드만 필요한 간단한 함수형 인터페이스 구현시&#x20;

#### 정리, 요약&#x20;

* 대부분의 경우 익명 클래스를 람다로 대체할 수 있다. 하지만 여러 메서드를 가진 인터페이스나 클래스의 경우에는 여전히 익명 클래스가 필요할 수 있다.&#x20;
* 자바 8 이후에는 익명 클래스, 람다 둘 다 선택할 수 있는 상황이라면 익명 클래스보다는 람다를 선택하는 것이 간결한 코드, 가독성 관점에서 대부분 더 나은 선택이다.&#x20;
