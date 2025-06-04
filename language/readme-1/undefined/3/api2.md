# 스트림 API2 - 기능

## 스트림 생성&#x20;

스트림(Stream) 은 자바 8부터 추가된 기능으로, 데이터 처리에 있어서 간결하고 효율적인 코드 작성을 가능하게 해준다. 스트림을 이용하면 컬렉션(List, Set 등) 이나 배열에 저장된 요소들을 반복문 없이도 간단하게 필터링(filter), 변환(map), 정렬(sorted) 등의 작업을 적용할 수 있다.

특히 스트림은 중간 연산, 최종 연산을 구분하며, 지연 연산을 통해 불필요한 연산을 최소화한다.

자바 스트림은 내부적으로 파이프라인 형태를 만들어 데이터를 단계별로 처리하고 결과를 효율적으로 반환한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.44.33.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.44.43.png" alt=""><figcaption></figcaption></figure>

스트림을 생성하는 대표적인 방법들을 코드로 알아보자.&#x20;

```java
package stream.operation;

import java.util.Arrays;
import java.util.List;
import java.util.stream.Stream;

public class CreateStreamMain {

    public static void main(String[] args) {
        System.out.println("1. 컬렉션으로부터 생성");
        List<String> list = List.of("a", "b", "c");
        Stream<String> stream1 = list.stream();
        stream1.forEach(System.out::println);

        System.out.println("2. 배열로부터 생성");
        String[] arr = {"a", "b", "c"};
        Stream<String> stream2 = Arrays.stream(arr);
        stream2.forEach(System.out::println);

        System.out.println("3. Stream.of() 사용");
        Stream<String> stream3 = Stream.of("a", "b", "c");
        stream3.forEach(System.out::println);

        System.out.println("4. 무한 스트림 생성 - iterate()");
        // iterate : 초기값과 다음 값을 만드는 함수 지정
        Stream<Integer> infiniteStream = Stream.iterate(0, n -> n + 2);
        infiniteStream.limit(3).forEach(System.out::println);

        System.out.println("5. 무한 스트림 생성 - generate()");
        // generate : Supplier 를 사용하여 무한하게 생성
        Stream<Double> randomStream = Stream.generate(Math::random);
        randomStream.limit(3).forEach(System.out::println);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.45.47.png" alt="" width="321"><figcaption></figcaption></figure>

#### 정리&#x20;

* 컬렉션, 배열, `Stream.of` 는 기본적으로 유한한 데이터 소스로부터 스트림을 생성한다.
* `iterate`, `generate` 는 별도의 종료 조건이 없으면 무한히 데이터를 만들어내는 스트림을 생성한다.&#x20;
  * 따라서, 필요한 만큼만(limit) 사용해야 한다. 그렇지 않으면 무한 루프처럼 계속 스트림을 뽑아내므로 주의
* 스트림은 일반적으로 한 번 사용하면 재사용할 수 없다(소진되면 끝), 따라서, `stream()` 으로 얻은 스트림을 여러 번 순회하려면, 다시 스트림을 생성해야 한다.&#x20;

## 중간 연산&#x20;

중간 연산(Intermediate Operation) 이란, 스트림 파이프라인에서 데이터를 변환, 필터링, 정렬 등을 하는 단계이다.&#x20;

* 여러 **중간 연산을 연결**하여 원하는 형태로 데이터를 가공할 수 있다.
* **결과가 즉시 생성되지 않고**, 최종 연산이 호출될 때 **한꺼번에 처리**된다는 특징이 있다. (지연 연산)

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.50.54.png" alt=""><figcaption></figcaption></figure>

중간 연산을 코드로 알아보자.&#x20;

```java
package stream.operation;

import javax.sound.midi.Soundbank;
import java.util.Comparator;
import java.util.List;
import java.util.stream.Stream;

public class IntermediateOperationsMain {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 9, 10);

        // 1. filter
        System.out.println("1. filter - 짝수만 선택");
        numbers.stream()
                .filter(n -> n % 2 == 0)
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 2. map
        System.out.println("2. map - 각 숫자의 제곱");
        numbers.stream()
                .map(n -> n * n)
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 3. distinct
        System.out.println("3. distinct - 중복 제거");
        numbers.stream()
                .distinct()
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 4. sorted(기본 정렬)
        System.out.println("4. sorted - 기본 정렬");
        Stream.of(3,1,4,1,5,9,2,6,5)
                .sorted()
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 5. sotred (커스텀 정렬)
        System.out.println("5. sorted with Comparator - 내림차순 정렬");
        Stream.of(3,1,4,1,5,9,2,6,5)
                .sorted(Comparator.reverseOrder())
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 6. peek
        System.out.println("6. peek - 동작 확인용");
        numbers.stream()
                .peek(n -> System.out.print("before : " + n + ", "))
                .map(n -> n * n)
                .peek(n -> System.out.print("after : " + n + ", "))
                .limit(5)
                .forEach(n -> System.out.println("최종값 : " + n));
        System.out.println("\n");

        // 7. limit
        System.out.println("7. limit - 처음 5개 요소만");
        numbers.stream()
                .limit(5)
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 8. skip
        System.out.println("8. skip - 처음 5개 요소를 건너뛰기");
        numbers.stream()
                .skip(5)
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        List<Integer> numbers2 = List.of(1, 2, 3, 4, 5, 1, 2, 3);
        // 9. takeWhile (Java 9+)
        System.out.println("9. takeWhile - 5보다 작은 동안만 선택");
        numbers2.stream()
                .takeWhile(n -> n < 5)
                .forEach(n -> System.out.print(n + " "));
        System.out.println("\n");

        // 10. dropWhile (Java 9+)
        System.out.println("10. dropWhile - 5보다 작은 동안만 건너뛰기");
        numbers2.stream()
                .dropWhile(n -> n < 5)
                .forEach(n -> System.out.print(n + " "));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.52.07.png" alt="" width="375"><figcaption></figcaption></figure>

#### 중간 연산 정리&#x20;

* 중간 연산은 파이프라인 형태로 연결할 수 있으며, **스트림을 변경하지만 원본 데이터 자체를 바꾸지 않는다.**&#x20;
* 중간 연산은 **`lazy` 하게 동작하므로, 최종 연산이 실행될 때까지 실제 처리는 일어나지 않는다.**&#x20;
* **`peek` 은 디버깅 목적으로 자주 사용**하며, 실제 스트림의 요소값을 변경하거나 연산 결과를 반환하지는 않는다.&#x20;
* `takeWhile`, `dropWhile` 는 자바 9부터 추가된 기능으로, **정렬된 스트림에서 사용할 때 유용하다.**&#x20;
  * 정렬되지 않은 스트림에서 쓰면 예측하기 어렵다.&#x20;

## FlatMap&#x20;

중간 연산의 하나인 `FlatMap` 에 대해서 알아보자.&#x20;

`map` 은 각 요소를 하나의 값으로 변환하지만, `FlatMap` 은 각 요소를 스트림(또는 여러 요소)으로 변환한 뒤, 그 결과를 하나의 스트림으로 평탄화(flatten) 해준다.&#x20;

이렇게 리스트 안에 리스트가 있다고 가정하자.&#x20;

```java
[
    [1, 2],
    [3, 4],
    [5, 6]
]
```

`FlatMap` 을 사용하면 이 데이터를 다음과 같이 쉽게 평탄화 할 수 있다.&#x20;

```java
[1, 2, 3, 4, 5, 6]
```

코드를 통해서 자세히 알아보자.&#x20;

```java
package stream.operation;

import java.util.ArrayList;
import java.util.List;
import java.util.stream.Stream;

public class MapVsFlatMapMain {

    public static void main(String[] args) {
        List<List<Integer>> outerList = List.of(
                List.of(1, 2),
                List.of(3, 4),
                List.of(5, 6)
        );
        System.out.println("outerList = " + outerList);

        // for
        List<Integer> forResult = new ArrayList<>();
        for (List<Integer> list : outerList) {
            for (Integer i : list) {
                forResult.add(i);
            }
        }
        System.out.println("forResult = " + forResult);
        
        // map 
        List<Stream<Integer>> mapResult = outerList.stream()
                .map(list -> list.stream())
                .toList();
        System.out.println("mapResult = " + mapResult);
        
        // flatMap
        List<Integer> flatMapResult = outerList.stream()
                .flatMap(list -> list.stream())
                .toList();
        System.out.println("flatMapResult = " + flatMapResult);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 12.59.48.png" alt=""><figcaption></figcaption></figure>

* `map` 을 쓰면 이중 구조가 그대로 유지된다. 즉, 각 요소가 `Stream` 형태가 되므로 결과가 `List<Stream<Integer>>` 가 된다.&#x20;
* `mapResult` 는 `Stream` 객체 참조값을 출력하므로 `[java.util.stream.ReferencePipeline$Head@...]` 형태로 보인다.&#x20;
* `flatMap` 을 사용하면 내부의 `Stream` 들을 하나로 합쳐 `List<Integer>` 를 얻을 수 있다.&#x20;

#### 정리&#x20;

* `flatMap` 은 중첩 구조를 일차원으로 펼치는 데 사용된다.
* 예를 들어, 문자열 리스트들이 들어있는 리스트를 평탄화하면, 하나의 연속된 문자열 리스트로 만들 수 있다.&#x20;

## 최종 연산&#x20;

최종 연산(Terminal Operation) 은 **스트림 파이파라인의 끝에 호출되어 실제 연산을 수행하고 결과를 만들어낸다.**\
**최종 연산이 실행된 후에 스트림은 소모되어 더 이상 사용할 수 없다.**

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.15.36.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.15.42.png" alt=""><figcaption></figcaption></figure>

```java
package stream.operation;

import java.util.Arrays;
import java.util.List;
import java.util.Optional;
import java.util.stream.Collectors;

public class TerminalOperationMain {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 2, 3, 4, 5, 5, 6, 7, 8, 9, 10);

        // Collectors 는 뒤에서 더 자세히(복잡한 수집이 필요할 때 사용)
        System.out.println("1. collect - List 수집");
        List<Integer> evenNumbers1 = numbers.stream()
                .filter(n -> n % 2 == 0)
                .collect(Collectors.toList());
        System.out.println("evenNumbers1 = " + evenNumbers1);

        System.out.println("2. toList - (java 16+))");
        List<Integer> evenNumbers2 = numbers.stream()
                .filter(n -> n % 2 == 0)
                .toList();
        System.out.println("evenNumbers2 = " + evenNumbers2);

        System.out.println("3. toArray - 배열로 변환");
        Integer[] arr = numbers.stream()
                .filter(n -> n % 2 == 0)
                .toArray(Integer[]::new);
        System.out.println("Arrays.toString(arr) = " + Arrays.toString(arr));

        System.out.println("4. forEach - 각 요소 처리");
        numbers.stream()
                .limit(5)
                .forEach(n -> System.out.print(n + " "));
        System.out.println();
        
        System.out.println("5. count - 요소 개수");
        long count = numbers.stream()
                .filter(n -> n > 5)
                .count();
        System.out.println("count = " + count);

        System.out.println("6. reduce - 요소들의 합");
        System.out.println("[초기값이 없는 reduce]");
        Optional<Integer> sum1 = numbers.stream()
                .reduce((a, b) -> a + b);
        System.out.println("sum1.get() = " + sum1.get());

        System.out.println("[초기값(100)이 있는 reduce]");
        int sum2 = numbers.stream()
                .reduce(100, (a, b) -> a + b);
        System.out.println("sum2.get() = " + sum2);

        System.out.println("7. min - 최소값");
        Optional<Integer> min = numbers.stream()
                .min(Integer::compareTo);
        System.out.println("min.get() = " + min.get());

        System.out.println("8. max - 최대값");
        Optional<Integer> max = numbers.stream()
                .max(Integer::compareTo);
        System.out.println("max.get() = " + max.get());

        System.out.println("9. findFirst - 첫 번째 요소");
        Optional<Integer> first = numbers.stream()
                .filter(n -> n > 5)
                .findFirst();
        System.out.println("first.get() = " + first.get());

        System.out.println("10. findAny - 아무 요소나 하나 찾기");
        Optional<Integer> any = numbers.stream()
                .filter(n -> n > 5)
                .findAny();
        System.out.println("any.get() = " + any.get());

        System.out.println("11. anyMatch - 조건을 만족하는 요소 존재 여부");
        boolean hasEven = numbers.stream()
                .anyMatch(n -> n % 2 == 0);
        System.out.println("hasEven = " + hasEven);

        System.out.println("12. allMatch - 모든 요소가 조건을 만족하는지 여부");
        boolean hasPositive = numbers.stream()
                .allMatch(n -> n > 0);
        System.out.println("hasPositive = " + hasPositive);

        System.out.println("13. noneMatch - 조건을 만족하는 요소가 없는지");
        boolean noNegative = numbers.stream()
                .noneMatch(n -> n < 0);
        System.out.println("noNegative = " + noNegative);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.16.18.png" alt="" width="375"><figcaption></figcaption></figure>

#### 정리&#x20;

* 최종 연산이 호출되면, 그 동안 정의된 모든 중간 연산이 한 번에 작동되어 결과를 만든다.&#x20;
* 최종 연산을 한 번 수행하면, 스트림은 재사용할 수 없다.&#x20;
* `reduce` 를 사용할 때 초기값을 지정하면, 스트림이 비어 있어도 초기값이 결과가 된다. 초기값이 없으면 `Optional` 을 반환한다.&#x20;
  * 초기값이 없는데, 스트림이 비어 있을 경우 빈 `Optional` (`Optional.empty()` )을 반환한다.
* `findFirst()`, `findAny()` 도 결과가 없을 수 있으므로 `Optional`을 통해 값 유무를 확인해야 한다.

지금까지 스트림 형성, 중간 연산, 최종연산에 대해 알아보았다.&#x20;

스트림은 컬렉션이나 배열을 사용하는 데 있어 코드를 단순화하고, 다양한 데이터 처리 연산을 간결하게 표현할 수 있게 해준다. 상황에 맞는 중간 연산, 최종 연산을 적절히 조합해서, 가동성과 유지보수성이 높은 코드를 작성하자.&#x20;

## 기본형 특화 스트림&#x20;

스트림 API에는 **기본형(primitive) 특화 스트림**이 존재한다.

자바에서는 `IntStream`, `LongStream`, `DoubleStream` 세 가지 형태를 제공하여 **기본 자료형(`int`, `long`, `double`)에 특화된 기능**을 사용할 수 있게 한다.

* 예를 들어, `IntStream` 은 **합계 계산**, **평균**, **최솟값**, **최댓값** 등 정수와 관련된 연산을 좀 더 편리하게 제공하고, **오토박싱/언박싱** 비용을 줄여 성능도 향상시킬 수 있다.

### 기본형 특화 스트림 종류&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.21.34.png" alt=""><figcaption></figcaption></figure>

### 주요 기능 및 메서드&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.22.17.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.22.24.png" alt=""><figcaption></figcaption></figure>

코드로 알아보자.&#x20;

```java
package stream.operation;

import java.util.IntSummaryStatistics;
import java.util.stream.DoubleStream;
import java.util.stream.IntStream;
import java.util.stream.LongStream;
import java.util.stream.Stream;

public class PrimitiveStreamMain {

    public static void main(String[] args) {
        // 1. 기본형 특화 스트림 생성
        IntStream stream = IntStream.of(1, 2, 3, 4, 5);
        stream.forEach(i -> System.out.print(i + " "));
        System.out.println();

        // 범위 생성 메서드
        IntStream range1 = IntStream.range(1, 6);
        IntStream range2 = IntStream.rangeClosed(1, 5);
        
        // 2. 통계 관련 메서드
        int sum = IntStream.range(1, 6).sum();
        System.out.println("sum = " + sum);

        double avg = IntStream.range(1, 6)
                .average()
                .getAsDouble();
        System.out.println("avg = " + avg);

        IntSummaryStatistics stats = IntStream.range(1, 6)
                .summaryStatistics();
        System.out.println("합계 : " + stats.getSum());
        System.out.println("평균 : " + stats.getAverage());
        System.out.println("최대값 : " + stats.getMax());
        System.out.println("최소값 : " + stats.getMin());
        System.out.println("개수 : " + stats.getCount());

        // 3. 타입 변환 메서드
        LongStream longStream = IntStream.range(1, 6).asLongStream();
        DoubleStream doubleStream = IntStream.range(1, 6).asDoubleStream();
        Stream<Integer> boxedStream = IntStream.range(1, 6).boxed();

        // int -> long 변환 매핑
        LongStream mappedLong = IntStream.range(1, 5)
                .mapToLong(i -> i * 10L);

        DoubleStream mappedDouble = IntStream.range(1, 5)
                .mapToDouble(i -> i * 1.5);

        // int -> 객체변환 매핑
        Stream<String> mappedObj = IntStream.range(1, 5)
                .mapToObj(i -> "Number : " + i);

        // 4. 객체 스트림 -> 기본형 특화 스트림 매핑
        Stream<Integer> integerStream = Stream.of(1, 2, 3, 4, 5);
        IntStream intStream = integerStream.mapToInt(i -> i);

        // 5. 객체 스트림 -> 기본형 특화 스트림 매핑 활용
        int result = Stream.of(1, 2, 3, 4, 5)
                .mapToInt(i -> i)
                .sum();
        System.out.println("result = " + result);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-21 13.23.30.png" alt="" width="444"><figcaption></figcaption></figure>

* **기본형 특화 스트림(IntStream, LongStream, DoubleStream)** 을 이용하면 숫자 계산(합계, 평균, 최대·최소 등)을 간편하게 처리하고, 박싱/언박싱 오버헤드를 줄여 **성능상의 이점**도 얻을 수 있다.
* **`range()`, `rangeClosed()`** 같은 메서드를 사용하면 범위를 쉽게 다룰 수 있어 **반복문 대신**에 자주 쓰인다.
* `mapToXxx`, `boxed()` 등의 메서드를 잘 활용하면 **객체 스트림**과 **기본형 특화 스트림**을 자유롭게 오가며 다양한 작업을 할 수 있다.
* `summaryStatistics()` 를 이용하면 합계, 평균, 최솟값, 최댓값 등 통계 정보를 **한 번에** 구할 수 있어 편리하다.&#x20;

### 성능 - 전통적인 for 문 vs 스트림 vs 기본형 특화 스트림

* 전통적인 for문이 보통 가장 빠르다.
* 스트림보다 전통적인 for 문이 1.5배 \~ 2배정도 빠르다.
  * 여기서 말하는 스트림은 `Integer` 같은 객체를 다루는 `Stream` 을 말한다.
  * 박싱/언박싱 오버헤드가 발생한다.
* 기본형 특화 스트림(`IntStream` 등)은 전통적인 for문에 가까운 성능을 보여준다.
  * 전통적인 for문과 거의 비슷하거나 전통적인 for문이 10% \~ 30% 정도 더 빠르다.
  * 박싱/언박싱 오버헤드를 피할 수 있다.
  * 내부적으로 최적화된 연산을 수행할 수 있다.

#### 실무 선택&#x20;

*   이런 성능 차이는 대부분의 일반적인 애플리케이션에서는 거의 차이가 없다. 이런 차이를 느끼려면 한 번에 사용

    하는 루프가 최소한 수천만 건 이상이어야 한다. 그리고 이런 루프를 많이 반복해야 한다.
*   박싱/언박싱을 많이 유발하지 않는 상황이라면 일반 스트림과 기본형 특화 스트림 간 성능 차이는 그리 크지 않을

    수 있다.
*   반면 대규모 데이터 처리나 반복 횟수가 많을 때는 기본형 스트림이 효과적일 수 있으며, 성능 극대화가 필요한 상

    황에서는 여전히 for 루프가 더 빠른 경우가 많다. 결국 최적의 선택은 구현의 가독성, 유지보수성등을 함께 고려

    해서 결정해야 한다.
*   **정리하면 실제 프로젝트에서는 극단적인 성능이 필요한 경우가 아니라면, 코드의 가독성과 유지보수성을 위해 스**

    **트림 API(스트림, 기본형 특화 스트림)를 사용하는 것이 보통 더 나은 선택이다.**
