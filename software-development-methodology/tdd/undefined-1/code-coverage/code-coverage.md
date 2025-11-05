# Code Coverage?

> 참고 링크 \
> [https://techblog.uplus.co.kr/code-coverage-c252e271df60](https://techblog.uplus.co.kr/code-coverage-c252e271df60)

## 1. Code Coverage?&#x20;

* 코드 커버리지는 소프트웨어 테스트에서 사용되는 지표 중 하나로, 코드의 실행 여부를 기반으로 코드의 품질과 완성도를 측정하는 방법이다.&#x20;
* 즉, 테스트 케이스를 실행할 때 코드 내의 각 요소가 실행되는 빈도를 측정하여, 테스트 케이스가 커버하는 코드 비율을 나타내는 지표이다.&#x20;
  * 보통 코드 커버리지는 테스트 케이스의 실행 여부를 기준으로 계산된다.&#x20;
  * 이를 통해, 테스트 케이스가 커버하지 않은 코드 블록을 식별해 개발자가 이를 보완할 수 있도록 도와준다.&#x20;
* 보통 코드 커버리지는 여러 가지 요소를 측정한다. 가장 일반적인 것은 다음과 같다.&#x20;
  * Statement Coverage : 실행된 문장 수 / 전체 문장 수&#x20;
  * Branch Coverage : 실행된 조건문 블록 수 / 전체 조건문 블록 수&#x20;
  * Function Coverage : 실행된 함수 수 / 전체 함수 수&#x20;
  * Decision Coverage : 수행된 분기 수 / 전체 분기 수&#x20;
  * Condition Coverage : 수행된 조건 수 / 전체 조건 수&#x20;
* 여러가지 코드 커버리지 종류가 있지만, 이번 글에서는 **JaCoCo(자바 코드 커버리지 측정 라이브러리)** 기준으로 설명한다.&#x20;

## 2. JaCoCo Code Coverage 종류&#x20;

* JaCoCo 를 통해서 코드 커버리지를 측정하면 아래와 같은 리포트를 확인할 수 있다.&#x20;
  * JaCoCo 리포트에서 제공하는 항목들을 보면 Instructions, Branches, Cxty, Lines, Methods, Classes 가 있다.

<figure><img src="../../../../.gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Class Coverage&#x20;

* 클래스가 1번 이상 사용되었는지를 기준으로 커버리지를 계산한다.&#x20;
* 클래스 사용 기준은 클래스 내 메서드 중 1개 이상 사용, 생성자, `Static initializer` 호출 시 클래스가 사용된 것으로 판단한다.&#x20;

### Method Coverage&#x20;

* 어떤 메소드가 최소 1번 이상 호출되었는지를 기준으로 커버리지를 계산한다.&#x20;
* 이때, 메서드 내부의 모든 코드가 실행되었는질는 판단 기준에서 제외된다.&#x20;

> Method Coverage(%) = (실행된 메서드의 수 / 전체 메서드의 수) \* 100&#x20;

#### 예제&#x20;

* 아래 `Calculator` 클래스에는 `add`, `substract`, `multuply`, `divide` 4개의 메서드가 존재한다.\
  &#xNAN;**(생성자 포함)**&#x20;
* 아래 테스트 케이스와 같이, 4개의 메서드를 모두 호출한다면 메서드 커버리지는 100% 가 된다.&#x20;
  * 따라서 메서드 커버리지는 클래스에서 지원하는 메서드에 대한 테스트 케이스 누락 여부를 판단할 수 있는 지표가 된다.&#x20;

```java
package com.lguplus.msa;

public class Calculator {

    public Calculator(){ } // 코드 실행 라인 #1

    public int add(int a, int b) {
        return a + b; // 코드 실행 라인 #2
    }

    public int subtract(int a, int b) {
        return a - b; // 코드 실행 라인 #3
    }

    public int multiply(int a, int b) {
        return a * b; // 코드 실행 라인 #4
    }

    public int divide(int a, int b) {
        if (b == 0) { // 코드 실행 라인 #5
            throw new IllegalArgumentException("0으로 나누면 안되요."); // 코드 실행 라인 #6
        }
        return a / b; // 코드 실행 라인 #7
    }
}
```

```java
@Test
assertEquals(3, calc.add(1,2));
assertEquals(1, calc.subtract(2,1));
assertEquals(2, calc.multiply(2,1));
assertEquals(2, calc.divide(2,1));
//-> 메소드 커버리지 100%
```

<figure><img src="../../../../.gitbook/assets/image (2) (1).png" alt=""><figcaption></figcaption></figure>

* 만약 `Multiply` 메서드 테스트 케이스가 누락된다면, 아래와 같이 누락된 메서드가 표시된다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (3) (1).png" alt=""><figcaption></figcaption></figure>

### Line Coverage&#x20;

* 테스트 중에 실행된 소스 코드 라인 수와 전체 소스 코드 라인 수 중 몇 개의 라인이 실행되었는지를 측정하며, 이를 통해 테스트에서 놓친 코드 라인이 얼마나 있는지 확인할 수 있다.&#x20;

> Line Coverage(%) = (실행된 코드 라인 수 / 전체 소스 코드 라인 수) \* 100

#### 예제

* 위 `Calculator` 클래스 예제를 좀 더 살펴보자.&#x20;
* 위 테스트 케이스에 의해 메서드 커버리지는 100% 가 된 상태이다.&#x20;
* 이때 라인 커버리지도 100% 를 만족하는 상태는 아니다..&#x20;

```java
@Test
...
assertEquals(2, calc.divide(2,1));
//-> 라인 커버리지 86%(6/7)
```

* 이유는 위 테스트 케이스의 경우 `divide` 메서드 내에 있는 `if` 조건문을 만족하지 않아 `Exception` 구문이 실행되지 않기 때문이다.&#x20;
* 따라서, 아래와 같이 인자 `b`가 0인 테스트 케이스를 추가해 주면 `divide` 메서드의 모든 코드라인이 수행되어 라인 커버리지는 100% 가 된다.&#x20;

```java
@Test
...
assertEquals(2, calc.divide(2,1));
assertEquals(2, calc.divide(2,0)); // 추가, b가 0인 경우, Exception 구문 테스트.
//-> 라인 커버리지 100%
```

#### JaCoCo 라인 커버리지 결과&#x20;

```java
@Test
assertEquals(2, calc.divide(2,1));
```

<figure><img src="../../../../.gitbook/assets/image (4) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (5) (1).png" alt=""><figcaption></figcaption></figure>

```java
@Test
...
assertEquals(2, calc.divide(2,1));
assertEquals(2, calc.divide(2,0)); // 추가, b가 0인 경우, Exception 구문 테스트.
```

<figure><img src="../../../../.gitbook/assets/image (6) (1).png" alt=""><figcaption></figcaption></figure>

### Branch Coverage&#x20;

* 소스 코드 내 분기문(if, swich 등) 에 대해 테스트 실행 흐름이 어떻게 되는지 측정하는 것이다.&#x20;
* 특정한 조건에 따라 코드 실행 흐름이 달라질 때, 모든 분기문이 정상적으로 실행되는지 확인할 수 있다.&#x20;

> 브랜치 커버리지(%) = (실행된 분기 수 / 전체 분기 수) \* 100

#### 예제

* 위 계산기 예제 코드에서 분기문은 `divide` 메서드 내에 있다.&#x20;
  * 이 경우 아래 테스트 케이ㅣ스로 분기문은 참, 거짓인 케이스를 확인하면 브랜치 커버리지는 100% 가 된다.&#x20;

```java
public int divide(int a, int b) {
    if (b == 0) { // 코드 실행 라인 #5
        throw new IllegalArgumentException("0으로 나누면 안되요."); // 코드 실행 라인 #6
    }
    return a / b; // 코드 실행 라인 #7
}
```

```java
@Test
assertEquals(2, calc.divide(2,1)); // (Case.1) b = 1, False
assertEquals(2, calc.divide(2,0)); // (Case.1) b = 0, True
//-> 브랜치 커버리지 100%
```

* 더 복잡한 예제를 확인해보자.

```java
public int sum(int a, int b) {
  int sum = 0
  if (a > 0 && b < 10) {
    sum = a + b;
  }
  return sum;
}
```

*   위 `sum` 메서드 내 인자 `a`,`b` 의 값에 따른 분기문이 존재한다. 이 조건문은 아래 케이스로 모두 테스트가 가능하다.&#x20;

    <figure><img src="../../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

#### JaCoCo 브랜치 커버리지 결과

```java
@Test
assertEquals(2, calc.divide(2,1)); // (Case.1) b = 1, False
```

<figure><img src="../../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

```java
@Test
assertEquals(2, calc.divide(2,1)); // (Case.1) b = 1, False
assertEquals(2, calc.divide(2,0)); // (Case.1) b = 1, True
```

<figure><img src="../../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

* foo 메서드를 추가해 브랜치 커버리지를 확인해보자.&#x20;

```java
@Test
assertEquals(6, calc.foo(1,5)); //true && true
assertEquals(0, calc.foo(0,11)); //false && false
```

<figure><img src="../../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

* 아래와 같이 누락된 조건 분기문의 테스트 케이스를 추가하면 브랜치 커버리지가 100% 가 된다.&#x20;

```java
@Test
assertEquals(6, calc.foo(1,5)); //true && true
assertEquals(0, calc.foo(0,11)); //false && false
assertEquals(0, calc.foo(1,11)); // true, false 테스트케이스 추가
```

<figure><img src="../../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

> #### 라인 커버리지 vs 브랜치 커버리지&#x20;
>
> * 브랜치 커버리지가 100% 인 경우 모든 코드 라인이 수행되므로 라인 커버리지도 100% 가 된다.&#x20;
> * **그러나 반대로 라인 커버리지가 100% 라고 해서 브랜치 커버리지가 100% 가 되지는 않는다..**&#x20;
>   * 복합 조건이 있는 경우를 생각해보자.

### Instructions Coverage&#x20;

* JaCoCo 에서 제공하는 가장 작은 단위의 커버리지 측정 방법이다.&#x20;
* 자바 바이트코드 단위에서 실행되었거나 실행되지 않은 코드의 양에 대한 정보를 측정한다.

#### 바이트코드 테스트까지는 가지 않을 것 같아서 패스 ..&#x20;

### 참고: Complexity by Type&#x20;

* 추가로 JaCoCo 리포트에 Cxty(Complexity by Type) 필드가 있다. Cxty 는 코드의 복잡도를 나타내는 매트릭이다.&#x20;
* 이 매트릭은 클래스나 인터페이스와 같은 코드 유형의 복잡도를 측정하는 데 사용하며, 순환복잡도를 계산해낸다. (자세한 공식은 패스\~)
* **당연히 복잡도는 낮은게 좋다. Cxty 가 높다면 복잡도를 낮출 수 있는 방안에 대해서 고민해보아야 한다.**&#x20;

## 3. 코드 커버리지, 어느 정도가 좋을까?&#x20;

### 높은 코드 커버리지 장점

* 코드 커버리지는 테스트 코드가 소스 코드의 얼마나 많은 부분을 실행하고 테스트하는지를 나타내는 지표이다.&#x20;
* 따라서 코드 커버리지를 높이는 것은 다음과 같은 이점이 제공해준다.&#x20;

1. 버그감소 : 당연한 이야기..&#x20;
2. 코드 품질 향상 : 테스트를 통해 버그가 발견되므로 코드 안정성과 신뢰성이 향상된다.&#x20;
3. 유지보수 용이성 : 코드 변경 사항에 대한 테스트 코드를 작성하는 것이 쉬워진다.&#x20;
4. 개발 생산성 향상 : 개발자가 버그를 수정하고 새로운 기능을 추가하는데 소요되는 시간이 줄어든다.

### 적정한 코드 커버리지 목표수준&#x20;

* 100% 면 가장 좋다.&#x20;
* 하지만 우리에게 시간은 한정되어 있고, 코드 커버리지 100% 가 무결점 소프트웨어를 의미하지는 않는다.&#x20;
* 그 이유는 다음과 같은 문제가 발생할 수 있기 때문이다.&#x20;

1. 요구사항에 있는 기능이 누락된 경우&#x20;
2. 구현에 오류가 있는 경우&#x20;
3. 테스트 케이스가 누락된 경우

* **따라서 단순히 코드 커버리지 숫자를 높이기 위해 테스트 케이스를 생성하는 일은 지양해야 하며, 요구사항 및 구현 코드에 대한 이해를 바탕으로 올바른 테스트 케이스를 작성해야 한다.**&#x20;
* 코드 커버리지를 높이는 일은 매우 고된 일이다.. \
  (더 어려운 점은 얼마만큼 코드 커버리지를 높여야 하는지에 대한 이상적인 가이드가 없다는 점이다..)&#x20;
* 정답은 아니지만 [Google Testing Blog](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html) 에서 코드 커버리지는 코드의 비지니스 영향/중요도, 코드를 얼마나 자주 건드리는지, 코드의 복잡성, 유지 기간 및 도메인 변수에 따라 도메인 오너가 수준을 정하는 것이 좋다고 가이드하고 있다.&#x20;
* **또한 구글에서 제시하고 있는 코드 커버리지 수준은 다음과 같이 가이드하고 있다.**&#x20;
  * **60%를 "적절한 수준"**
  * **75%를 "칭찬받을만한 수준"**&#x20;
  * **90%를 "모범적인 수준"**

### LG 유플러스 기준&#x20;

* **Method Coverage**를 기본 목표로 삼고 있지만, 이는 메서드 내 모든 코드 실행을 보장하지 않기 때문에 추가적인 커버리지 지표가 필요하다.
* **Method Coverage 100%를 기본 목표**로 하고
* **Branch Coverage 향상을 위한 도전적인 목표를 설정해 나간다. (70% -> 80% -> 90%)**
