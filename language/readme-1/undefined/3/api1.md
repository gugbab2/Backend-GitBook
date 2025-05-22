# 스트림API1 - 기본

## 스트림 API 시작&#x20;

우리는 앞서 필터와 맵 등을 여러 함수와 함께 사용하는 MyStreamV3 를 직접 만들었다.&#x20;

```java
List<String> result = MyStreamV3.of(students)
    .filter(s -> s.getScore() >= 80)
    .map(s -> s.getName())
    .toList();
```

* 코드를 보면 개발자는 작업을 어떻게(HOW) 수행해야 하는지 보다는 무엇(WHAT) 을 수행해야 하는지, 즉 원하는 결과에 집중할 수 있다.&#x20;
* 이러한 방식을 선언적 프로그래밍 방식이라 한다.&#x20;

우리가 만든 스트림을 사용할 때를 떠올려보면 데이터들이 흘러가면서 필터되고, 매핑된다. 그래서 마치 데이터가 물 흐르듯이 흘러간다는 느낌을 받았을 것이다.&#x20;

자바도 스트림API 라는 이름으로 스트림 관련 기능들을 제공한다. (I/O 스트림이 아니다)&#x20;

자바가 제공하는 스트림 API 를 사용해보자.

```java
package stream.start;

import java.util.List;
import java.util.stream.Stream;

public class StreamStartMain {

    public static void main(String[] args) {
        List<String> names = List.of("Apple", "Banana", "Berry", "Tomato");

        // "B" 로 시작하는 이름만 필터 후 대문자로 바꿔서 리스트 수집
        Stream<String> stream = names.stream();
        List<String> result = stream
                .filter(name -> name.startsWith("B"))
                .map(s -> s.toUpperCase())
                .toList();

        System.out.println("=== 외부 반복 ===");
        for (String s : result) {
            System.out.println(s);
        }

        System.out.println("=== 내부 반복 ===");
        names.stream()
                .filter(name -> name.startsWith("B"))
                .map(s -> s.toUpperCase())
                .forEach(s -> System.out.println(s));

        System.out.println("=== 메서드 참조 ===");
        names.stream()
                .filter(name -> name.startsWith("B"))
                .map(String::toUpperCase)
                .forEach(System.out::println);
    }
}
```

#### 스트림 생성&#x20;

```java
List<String> names = List.of("Apple", "Banana", "Berry", "Tomato");
Stream<String> stream = names.stream();
```

* `List.of(...)` 로 미리 몇 가지 과일 이름을 담은 리스트를 생성했다.&#x20;
* `List` 의 `stream()` 메서드를 사용하면 **자바가 제공하는 스트림**을 생성할 수 있다.&#x20;

#### 중간 연산(Intermedeate Operations) - `filter`, `map`&#x20;

```java
.filter(name -> name.startsWith("B"))
.map(s -> s.toUpperCase())
```

* 중간 연산은 스트림에서 요소를 걸러내거나(필터링), 다른 형태로 변환(매핑) 하는 기능이다.&#x20;

#### 최종 연산(Terminal Operation) - `toList()`&#x20;

```java
List<String> result = stream
                .filter(name -> name.startsWith("B"))
                .map(s -> s.toUpperCase())
                .toList();
```

#### 외부 반복&#x20;

```java
System.out.println("=== 외부 반복 ===");
for (String s : result) {
    System.out.println(s);
}
```

#### 내부 반복&#x20;

```java
System.out.println("=== 내부 반복 ===");
names.stream()
        .filter(name -> name.startsWith("B"))
        .map(s -> s.toUpperCase())
        .forEach(s -> System.out.println(s));
```

#### 메서드 참조&#x20;

```java
System.out.println("=== 메서드 참조 ===");
names.stream()
        .filter(name -> name.startsWith("B"))
        .map(String::toUpperCase)
        .forEach(System.out::println);
```

#### 정리&#x20;

* 중간 연산(`filter`, `map`) 은 데이터를 걸러내거나 형태를 변환하며, 최종연산(`toList`, `forEach` 등) 을 통해 최종 결과를 모으거나 실행할 수 있다.&#x20;
* 스트림의 내부 반복을 통해, "어떻게 반복할지를 직접 신경 쓰기보다는, 결과가 어떻게 변환되어야 하는지" 에만 집중할 수 있다. 이런 특징을 선언형 프로그래밍(Declarative Programming) 스타일이라 한다.&#x20;
* 메서드 참조는 람다식을 더 간결하게 표현하며, 가독성을 높여준다.&#x20;

## 스트림 API 란?&#x20;

#### 정의&#x20;

* **스트림(Stream)** 은 자바 8부터 추가된 기능으로, **데이터의 흐름을 추상화**해서 다루는 도구이다.&#x20;
* **컬렉션(Collection) 또는 배열** 등의 요소들을 연산 파이프라인을 통해 **연속적인 형태**로 처리할 수 있게 해준다.&#x20;
  * 연산 파이프라인 : 여러 연산(중간 연산, 최종 연산) 을 체이닝해서 데이터를 변환, 필터링, 계산하는 구조

#### 스트림의 특징&#x20;

1. 원본 데이터 소스를 변경하지 않음(Immutable)
2. 일회성(1회 소비)
3. 파이프라인(Pipeline) 구성&#x20;
   1. 중간 연산들이 이어지다가, 최종 연산을 만나면 연산이 수행되고 종료된다.&#x20;
4. 지연 연산(Lazy Operation)&#x20;
   1. 중간 연산은 필요할 때까지 실제로 동작하지 않고, 최종 연산이 실행될 때 한 번에 처리된다.&#x20;
5. 병렬 처리(Parallel) 용이&#x20;
   1. 스트림으로부터 병렬 스트림을 쉽게 만들 수 있어서, 멀티코어 환경에서 병렬 연산을 비교적 단순한 코드로 작성할 수 있다.&#x20;

## 파이프라인 구성&#x20;

먼저 예제를 통해  우리가 만든 스트림과 자바가 제공하는 스트림이 어떻게 다른지 살펴보자.&#x20;

요구사항은 다음과 같다.&#x20;

* 1 \~ 6 의 숫자가 입력된다.&#x20;
* 짝수를 구해라.
* 구한 짝수에 10 을 곱해서 출력해라.

```java
package stream.basic;

import lambda.lambda5.mystream.MyStreamV3;

import java.util.List;

public class LazyEvalMain1 {

    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5, 6);
        ex1(data);
        ex2(data);
    }

    private static void ex1(List<Integer> data) {
        System.out.println("== MyStreamV3 시작 ==");
        List<Integer> result = MyStreamV3.of(data)
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                })
                .toList();
        System.out.println("result = " + result);
        System.out.println("== MyStreamV3 종료 ==");
    }

    private static void ex2(List<Integer> data) {
        System.out.println("== Stream API 시작 ==");
        List<Integer> result = data.stream()
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                })
                .toList();
        System.out.println("result = " + result);
        System.out.println("== Stream API 종료 ==");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 18.08.38.png" alt="" width="375"><figcaption></figcaption></figure>

### 일괄 처리 vs 파이프라인&#x20;

**`MyStreamV3` 는 일괄 처리 방식**이고, **자바 Stream API 는 파이프라인 방식**이다.&#x20;

#### 정리&#x20;

일괄 처리(Batch Processing)&#x20;

* 공정(중간 연산)을 단계별로 쪼개서 데이터 전체를 한 번에 처리하고, 결과를 저장해두었다가 다음 공정을 또 한번 수행한다.&#x20;

파이프라인 처리(Pipeline Processing)&#x20;

* 한 요소(제품)가 한 공정을 마치면, 즉시 다음 공정으로 넘어가는 구조이다.&#x20;

### 코드 분석&#x20;

#### MyStreamV3&#x20;

1. `data(1,2,3,4,5,6)`
2. &#x20;`filter(1,2,3,4,5,6) -> 2,4,6(통과)`&#x20;
3. `map(2,4,6) -> 20,40,60`
4. `list(20,40,60)`



* `data` 에 있는 요소를 한 번에 모두 꺼내서 `filter` 에 적용한다.&#x20;
* `filter()` 가 모든 요소에 대해 순서대로 모두 실행한 후, 조건에 통과한 요소에 대해서 `map` 이 한 번에 실행된다.&#x20;
* `map()` 의 실행이 모두 끝나고 나서 한 번에 20,40,60 이 최종 `list` 에 담긴다.&#x20;
  * 즉, 모든 요소의 변환이 끝난 뒤에야 최종 결과가 생성된다. (일괄 처리)

#### MyStreamV3 로그&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 18.14.59.png" alt="" width="369"><figcaption></figcaption></figure>

#### 자바 Stream API&#x20;

1. `data(1) -> filter(1) → false → 통과 못함, skip`&#x20;
2. `data(2) -> filter(2) → true → 통과, 바로 map(2) -> 20 실행 -> list(20)`&#x20;
3. `data(3) -> filter(3) → false`&#x20;
4. `data(4) -> filter(4) → true → 바로 map(4) -> 40 -> list(20,40)`&#x20;
5. `data(5) -> filter(5) → false`&#x20;
6. `data(6) -> filter(6) → true → 바로 map(6) -> 60 -> list(20,40,60)`



* `data` 에 있는 요소를 하나씩 꺼내서 `filter`, `map` 을 적용하는 구조이다.
* 한 요소가 `filter` 를 통과하면 곧바로 `map` 이 적용되는 "파이프라인 처리"가 일어난다.
* 각각의 요소가 개별적으로 파이프라인을 따라 별도로 처리되는 방식이다.

#### 스트림 API 로그&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 18.19.22.png" alt="" width="369"><figcaption></figcaption></figure>

#### 정리&#x20;

* 두 방식의 결과는 같지만, 실제 실행 과정에서 차이가 있다.&#x20;
* 핵심은 자바 스트림은 중간 단계에서 **데이터를 모아서 한 방에 처리하지 않고**, 한 요소가 중간 연산을 통과하면 곧 바로 다음 중간 연산으로 이어지는 **파이프라인 형태를 가진다는 점이다.**

## 지연 연산&#x20;

### 최종 연산을 수행해야 작동한다.&#x20;

자바 스트림은 `toList()` 와 같은 최종 연산을 수행할 때만 작동한다.&#x20;

여기서는 **최종 연산을 제외했다.**&#x20;

```java
package stream.basic;

import lambda.lambda5.mystream.MyStreamV3;

import java.util.List;

public class LazyEvalMain2 {

    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5, 6);
        ex1(data);
        ex2(data);
    }

    private static void ex1(List<Integer> data) {
        System.out.println("== MyStreamV3 시작 ==");
        MyStreamV3.of(data)
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                });
        System.out.println("== MyStreamV3 종료 ==");
    }

    private static void ex2(List<Integer> data) {
        System.out.println("== Stream API 시작 ==");
        data.stream()
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                });
        System.out.println("== Stream API 종료 ==");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 18.26.45.png" alt=""><figcaption></figcaption></figure>

* `MyStreamV3` 에서는 최종 연산(`toList`, `forEach`) 를 호출하지 않았는데도, 중간 연산(`filter`, `map`) 이 바로바로 실행되어 모든 로그가 확인된다.&#x20;
* **반면에 자바 스트림은 최종 연산이 호출되지 않으면 아무 일도 수행하지 않는다.**
* 이것이 스트림 API 의 지연 연산을 가장 극명하게 보여주는 예시이다.&#x20;
  *   중간 연산들은 "이런 일을 할 것이다"라는 파이프라인 설정을 해놓기만 하고, 정작 **실제 연산은 최종 연산이**

      **호출되기 전까지 전혀 진행되지 않는다.**

#### 즉시 연산&#x20;

* 우리가 만든 MyStreamV3 는 즉시(Eager) 연산을 사용하고 있다.&#x20;
* 그 결과, 최종 연산 없이도 중간 연산이 실행되어, 필요 이상의 연산이 수행되기도 한다.&#x20;

#### 지연 연산&#x20;

* 쉽게 이야기하면 스트림 API는 매우 게으르다(Lazy). 정말 꼭! 필요할 때만 연산을 수행하도록 최대한 미루고 미룬다.
* 그래서 연산을 반드시 수행해야 하는 최종 연산을 만나야 본인이 가지고 있던 중간 연산들을 수행한다.
* 이렇게 꼭 필요할 때 까지 연산을 최대한 미루는 것을 **지연(Lazy) 연산** 이라 한다.

## 지연 연산 최적화

### 지연 연산과 파이프라인 최적화 예시&#x20;

자바의 스트림은 지연 연산, 파이프라인 방식 등 왜 이렇게 복잡하게 설계되어 있을까?

실제로 지연 연산과 파이프라인을 통해 어떤 최적화를 할 수 있는지 알아보자.

데이터 리스트 중에 짝수를 찾고, 찾은 짝수에 10을 곱하자. 이렇게 계산한 짝수 중에서 첫 번째 항목 하나만 찾는다고\
가정해보자.

예를 들어, `[1, 2, 3, 4, 5, 6]` 이 있다면 첫 번째 짝수는 2이므로 2에 10을 곱한 20 하나만 찾으면 된다.

예시를 위해 `MyStreamV3` 에 다음 내용을 추가하자

```java
package lambda.lambda5.mystream;

public class MyStreamV3<T> {

    ...
    
    // 추가
    public T getFirst() {
        return internalList.get(0);
    }
}
```

최종 연산에서 첫 번째 항목을 찾도록 해보자.

```java
package stream.basic;

import lambda.lambda5.mystream.MyStreamV3;

import java.util.List;

public class LazyEvalMain3 {

    public static void main(String[] args) {
        List<Integer> data = List.of(1, 2, 3, 4, 5, 6);
        ex1(data);
        ex2(data);
    }

    private static void ex1(List<Integer> data) {
        System.out.println("== MyStreamV3 시작 ==");
        Integer result = MyStreamV3.of(data)
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                })
                .getFirst();
        System.out.println("result = " + result);
        System.out.println("== MyStreamV3 종료 ==");
    }

    private static void ex2(List<Integer> data) {
        System.out.println("== Stream API 시작 ==");
        Integer result = data.stream()
                .filter(i -> {
                    boolean isEven = i % 2 == 0;
                    System.out.println("filter() 실행 : " + i + "(" + isEven + ")");
                    return isEven;
                })
                .map(i -> {
                    int mapped = i * 10;
                    System.out.println("map() 실행 : " + i + " -> " + mapped);
                    return mapped;
                })
                .findFirst().get();
        System.out.println("result = " + result);
        System.out.println("== Stream API 종료 ==");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-20 18.32.35.png" alt=""><figcaption></figcaption></figure>

#### MyStreamV3&#x20;

*   `MyStreamV3` 는 모든 데이터에 대해 짝수를 걸러내고(`filter`) 나서, 걸러진 결과에 대해 전부 `map()` 연산

    (곱하기 10)을 수행한 뒤에야 `getFirst()` 가 20을 반환했다.
*   즉, **모든 요소(1\~6)에 대해 필터 → 모두 통과한 요소에 대해 map**을 끝까지 수행한 후 결과 목록 중 첫 번째 원소

    (20)을 꺼낸 것이다.
* 여기서는 `filter`, `map` 에 대해 총 9번의 연산이 발생했다. (filter 6번, map 3번)

#### 스트림 API&#x20;

* **자바 스트림 API**는 `findFirst()`라는 **최종 연산**을 만나면, 조건을 만족하는 요소(2 -> 20)을 멈추고 **곧바로 결과를 반환** 해버린다.
  * `filter(1) → false` (버림)
  * `filter(2) → true → map(2) -> 20 → findFirst()`는 결과를 찾았으므로 종료
  * 따라서 이후의 요소(3, 4, 5, 6)에 대해서는 더 이상 `filter`, `map`을 호출하지 않는다.
  * 콘솔 로그에도 1, 2까지만 출력된 것이 확인된다.
*   여기서는 `filter`, `map` 에 대해 총 3번의 연산이 발생했다. (filter 2번, map 1번)



이를 "**단축 평가**"(short-circuit) 라고 하며, 조건을 만족하는 결과를 찾으면 더 이상 연산을 진행하지 않는 방식이다.

* **지연 연산**과 **파이프라인 방식**이 있기 때문에 가능한 최적화 중 하나이다.

### 지연 연산 정리&#x20;

스트림 API에서 **지연 연산**(Lazy Operation, 게으른 연산)이란, `filter`, `map`같은 **중간 연산**들은 `toList`와 같은 **최종 연산(Terminal Operation)** 이 호출되기 전까지 실제로 실행되지 않는다는 의미이다.

* 즉, 중간 연산들은 결과를 **바로 계산**하지 않고, "무엇을 할지"에 대한 **설정만** 저장해 둔다.
  * 쉽게 이야기해서 람다 함수만 내부에 저장해두고, 해당 함수를 실행하지는 않는다.
*   그리고 최종 연산(예: `toList()`, `forEach()`, `findFirst()` 등)이 실행되는 순간, `그때서야` 중간 연산이

    순차적으로 **한 번에** 수행된다. (저장해둔 람다들을 실행한다.)

이를 통해 얻을 수 있는 장점은 다음과 같다. &#x20;

1. **불필요한 연산의 생략(단축, Short-Circulting)**&#x20;
2. **메모리 사용 효율**&#x20;
3. **파이프라인 최적화**
   1.  스트림은 요소를 하나씩 꺼내면서(=순차적으로) `filter`, `map` 등 연산을 **묶어서** 실행할 수 있다.

       즉, 요소 하나를 꺼냈으면 → 그 요소에 대한 `filter`→ 통과하면 `map` → ... → 최종 연산까지 진행하고, 다

       음 요소로 넘어간다.
   2.  이렇게 하면 메모리를 절약할 수 있고, 짜잘짜잘하게 중간 단계를 저장하지 않아도 되므로, 내부적으로 효율

       적으로 동작한다.
   3.  추가로 `MyStreamV3` 와 같은 배치 처리 방식에서는 각 단계별로 모든 데이터를 한 번에 처리하기 때문에

       지연 연산을 사용해도 이런 최적화가 어렵다. 자바 스트림 API는 필요한 데이터를 하나씩 불러서 처리하는

       파이프라인 방식이므로 이런 최적화가 가능하다.
