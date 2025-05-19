# 람다 활용

## 필터 만들기1&#x20;

람다를 처음 사용하면, 대부분 바로바로 람다를 사용하기는 어렵다. 람다에 익숙해지는데 어느정도의 시간이 걸린다.&#x20;

이번 시간에는 람다를 활용하는 방법들을 알아보고, 또 다양한 문제를 풀어보면서 람다에 익숙해지는 시간을 가져보자.&#x20;

### 필터1&#x20;

먼저 람다를 사용하지 않고, 짝수만 거르기, 홀수만 거르기 메서드를 각각 따로 작성해보자.&#x20;

```java
package lambda.lambda5.filter;

import java.util.ArrayList;
import java.util.List;

public class FilterMainV1 {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 짝수만 고르기
        List<Integer> evenNumbers = filterEvenNumber(numbers);
        System.out.println("evenNumbers = " + evenNumbers);
        
        // 홀수만 고르기
        List<Integer> oddNumber = filterOddNumber(numbers);
        System.out.println("oddNumber = " + oddNumber);
    }

    private static List<Integer> filterEvenNumber(List<Integer> numbers) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer number : numbers) {
            boolean testResult = number % 2 == 0;
            if (testResult) {
                filtered.add(number);
            }
        }
        return  filtered;
    }

    private static List<Integer> filterOddNumber(List<Integer> numbers) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer number : numbers) {
            boolean testResult = number % 2 == 1;
            if (testResult) {
                filtered.add(number);
            }
        }
        return  filtered;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.17.07.png" alt=""><figcaption></figcaption></figure>

### 문제&#x20;

* 앞서 작성한 `filterEvenNumber()`, `filterOddNumber()` 두 메서드 대신에 `filter()` 라는 하나의 메서드만 사용해서 중복을 제거하자.
* 람다를 활용하자.&#x20;

### 필터2&#x20;

```java
package lambda.lambda5.filter;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

public class FilterMainV2 {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 짝수만 고르기
        Predicate<Integer> evenPredicate = n -> n % 2 == 0;
        List<Integer> evenNumbers = filter(numbers, evenPredicate);
        System.out.println("evenNumbers = " + evenNumbers);
        
        // 홀수만 고르기
        Predicate<Integer> oddPredicate = n -> n % 2 == 1;
        List<Integer> oddNumber = filter(numbers, oddPredicate);
        System.out.println("oddNumber = " + oddNumber);
    }

    private static List<Integer> filter(List<Integer> numbers, Predicate<Integer> predicate) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer number : numbers) {
            boolean testResult = predicate.test(number);
            if (testResult) {
                filtered.add(number);
            }
        }
        return  filtered;
    }
}
```

* `Predicate<Integer>` 를 `filter()` 에 인자로 넘긴다.&#x20;
*   `boolean testResult = predicate.test(number)` 을 사용해서 넘긴 코드 조각을 `filter()` 안에

    서 실행한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.18.59.png" alt=""><figcaption></figcaption></figure>

### 필터3&#x20;

코드를 조금 다듬어보자.&#x20;

```java
package lambda.lambda5.filter;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

public class FilterMainV3 {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 짝수만 고르기
        List<Integer> evenNumbers = filter(numbers, n -> n % 2 == 0);
        System.out.println("evenNumbers = " + evenNumbers);
        
        // 홀수만 고르기
        List<Integer> oddNumber = filter(numbers, n -> n % 2 == 1);
        System.out.println("oddNumber = " + oddNumber);
    }

    private static List<Integer> filter(List<Integer> numbers, Predicate<Integer> predicate) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer number : numbers) {
            if (predicate.test(number)) {
                filtered.add(number);
            }
        }
        return  filtered;
    }
}
```

* `evenPredicate`, `oddPredicate`, `testResult` 변수를 제거했다.&#x20;
* 해당 변수들은 학습의 이해를 쉽게 만들기 위한 변수들로 꼭 필요한 변수들이 아니다.&#x20;
* 특히 람다의 경우 주로 간단한 식을 사용하므로, 복잡할 때를 제외하고는 변수를 잘 만들지 않는다.&#x20;
  * `evenPredicate`, `oddPredicate`, `testResult` 변수를 제거하니 가독성이 더 좋아진 것을 확인할 수 있다.

## 필터 만들기2&#x20;

앞서 만든 `filter()` 메서드는 매개변수가 `List<Integer> numbers, Predicate<Integer> predicate` 이다.

따라서 숫자 리스트에 있는 값을 필터링 하는 모든 곳에서 사용할 수 있다.

### 필터4&#x20;

```java
package lambda.lambda5.filter;

import java.util.List;

public class FilterMainV4 {

    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // 짝수만 고르기
        List<Integer> evenNumbers = IntegerFilter.filter(numbers, n -> n % 2 == 0);
        System.out.println("evenNumbers = " + evenNumbers);
        
        // 홀수만 고르기
        List<Integer> oddNumber = IntegerFilter.filter(numbers, n -> n % 2 == 1);
        System.out.println("oddNumber = " + oddNumber);
    }
}
```

```java
package lambda.lambda5.filter;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

public class IntegerFilter {

    public static List<Integer> filter(List<Integer> list, Predicate<Integer> predicate) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer num : list) {
            if (predicate.test(num)) {
                filtered.add(num);
            }
        }
        return  filtered;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.23.06.png" alt=""><figcaption></figcaption></figure>

* 범용성 있게 다양한 곳에서 사용할 수 있는 `IntegerFilter` 클래스를 만들었다.&#x20;
* 하지만 `Integer` 숫자에만 사용할 수 있는 한계가 있다.&#x20;

### 필터5 - 제네릭 도입&#x20;

제네릭을 사용하면 클래스 코드의 변경 없이 다양한 타입을 적용할 수 있다.&#x20;

앞서 만든 `IntegerFilter` 에 제네릭을 도입해보자.&#x20;

```java
package lambda.lambda5.filter;

import java.util.List;

public class FilterMainV5 {

    public static void main(String[] args) {
        // 숫자 사용 필터
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        List<Integer> numberResult = GenericFilter.filter(numbers, n -> n % 2 == 0);    // 제네릭 메서드 타입 추론
        System.out.println("numberResult = " + numberResult);

        // 문자 사용 필터
        List<String> strings = List.of("A", "BB", "CCC");
        List<String> stringResult = GenericFilter.filter(strings, s -> s.length() > 2);
        System.out.println("stringResult = " + stringResult);
    }
}
```

```java
package lambda.lambda5.filter;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Predicate;

public class GenericFilter {

    public static<T> List<T> filter(List<T> list, Predicate<T> predicate) {
        List<T> filtered = new ArrayList<>();
        for (T num : list) {
            if (predicate.test(num)) {
                filtered.add(num);
            }
        }
        return  filtered;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.25.02.png" alt=""><figcaption></figcaption></figure>

* 제네릭을 도입한 덕분에 `Integer`, `String` 같은 다양한 타입의 리스트에 필터링 기능을 사용할 수 있게 되었다.&#x20;
* `GenericFilter` 는 제네릭을 사용할 수 있는 모든 타입의 리스트를 람다 조건으로 필터링 할 수 있다. 따라서 매우 유연한 필터링 기능을 제공한다.

## 맵 만들기1&#x20;

맵(map) 은 대응, 변환을 의미하는 매핑(Mapping) 의 줄임말이다.

매핑은 어떤 것을 다른 것으로 변환하는 과정을 의미한다.&#x20;

프로그래밍에서는 각 요소를 다른 값으로 변환하는 작업을 매핑(mapping, map) 이라 한다.&#x20;

쉽게 이야기해서 어떤 하나의 데이터를 다른 데이터로 변환하는 작업이라고 생각하면 된다.&#x20;

리스트에 있는 특정 값을 다른 값으로 매핑(변환) 해보자.&#x20;

### 맵1&#x20;

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;

public class MapMain1 {

    public static void main(String[] args) {
        List<String> list = List.of("1", "12", "123", "1234");

        // 문자열을 숫자로 변환 
        List<Integer> numbers = mapStringToInteger(list);
        System.out.println("numbers = " + numbers);

        // 문자열의 길이로 변환
        List<Integer> nlengths = mapStringToLength(list);
        System.out.println("nlengths = " + nlengths);
    }

    private static List<Integer> mapStringToInteger(List<String> list) {
        List<Integer> numbers = new ArrayList<>();
        for (String s : list) {
            Integer value = Integer.valueOf(s);
            numbers.add(value);
        }
        return numbers;
    }

    private static List<Integer> mapStringToLength(List<String> list) {
        List<Integer> numbers = new ArrayList<>();
        for (String s : list) {
            Integer value = s.length();
            numbers.add(value);
        }
        return numbers;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.28.12.png" alt=""><figcaption></figcaption></figure>

* 문자열을 숫자로 변환 (`mapStringToInteger`)
* 문자열을 문자열의 길이로 변환 (`mapStringToLength`)

### 문제&#x20;

* MapMainV1 클래스를 복사해서 MapMainV2 클래스를 만들자.&#x20;
* 앞서 작성한 `mapStringToInteger()`, `mapStringToLength() 두 메서드 대신에 map()` 이라는 하나의 메서드만 사용해서 중복을 제거하자.&#x20;
* 람다를 사용하자.&#x20;

### 맵2&#x20;

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class MapMain2 {

    public static void main(String[] args) {
        List<String> list = List.of("1", "12", "123", "1234");

        // 문자열을 숫자로 변환 
        Function<String, Integer> toNumber = s -> Integer.valueOf(s);
        List<Integer> numbers = map(list, toNumber);
        System.out.println("numbers = " + numbers);

        // 문자열의 길이로 변환
        Function<String, Integer> toLength = s -> s.length();
        List<Integer> nlengths = map(list, toLength);
        System.out.println("nlengths = " + nlengths);
    }

    private static List<Integer> map(List<String> list, Function<String, Integer> function) {
        List<Integer> numbers = new ArrayList<>();
        for (String s : list) {
            Integer value = function.apply(s);
            numbers.add(value);
        }
        return numbers;
    }
}
```

* `Function<String, Integer>` 를 `map()` 에 인자로 넘긴다.&#x20;
* `Integer value = mapper.apply(s)` 을 사용해서 넘긴 코드 조각을 `map()` 안에서 실행한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.31.19.png" alt=""><figcaption></figcaption></figure>

### 맵3&#x20;

코드를 정리해보자.

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class MapMain3 {

    public static void main(String[] args) {
        List<String> list = List.of("1", "12", "123", "1234");

        // 문자열을 숫자로 변환 
        List<Integer> numbers = map(list, s1 -> Integer.valueOf(s1));
        System.out.println("numbers = " + numbers);

        // 문자열의 길이로 변환
        List<Integer> nlengths = map(list, s -> s.length());
        System.out.println("nlengths = " + nlengths);
    }

    private static List<Integer> map(List<String> list, Function<String, Integer> function) {
        List<Integer> numbers = new ArrayList<>();
        for (String s : list) {
            numbers.add(function.apply(s));
        }
        return numbers;
    }
}
```

## 맵 만들기2&#x20;

앞서 만든 `map()` 메서드는 매개변수가 `List<String> list, Function<String, Integer> mapper` 이다.

따라서 문자열 리스트를 숫자 리스트로 변환(매핑)할 때 사용할 수 있다.

다양한 곳에서 활용할 수 있으므로, 별도의 유틸리티 클래스로 만들어보자.

### 맵4&#x20;

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class MapMain4 {

    public static void main(String[] args) {
        List<String> list = List.of("1", "12", "123", "1234");

        // 문자열을 숫자로 변환 
        List<Integer> numbers = StringToIntegerMapper.map(list, s1 -> Integer.valueOf(s1));
        System.out.println("numbers = " + numbers);

        // 문자열의 길이로 변환
        List<Integer> nlengths = StringToIntegerMapper.map(list, s -> s.length());
        System.out.println("nlengths = " + nlengths);
    }
}
```

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class StringToIntegerMapper {

    public static List<Integer> map(List<String> list, Function<String, Integer> mapper) {
        List<Integer> numbers = new ArrayList<>();
        for (String s : list) {
            numbers.add(mapper.apply(s));
        }
        return numbers;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.34.13.png" alt=""><figcaption></figcaption></figure>

* 범용성 있게 다양한 곳에서 사용할 수 있는 `StringToIntegerMapper` 클래스를 만들었다.&#x20;
* 하지만 `String` 리스트를 `Integer` 리스트로 변환할 때만 사용할 수 있는 한계가 있다.

### 맵5 - 제네릭 도입&#x20;

제네릭을 사용하면 클래스 코드의 변경 없이 다양한 타입을 적용할 수 있다.&#x20;

앞서 만든 `StringToIntegerMapper` 에 제네릭을 도입해보자.&#x20;

```java
package lambda.lambda5.map;

import java.util.List;

public class MapMain5 {

    public static void main(String[] args) {
        List<String> fruits = List.of("apple", "banana", "orange");

        // String -> String
        List<String> upperFruits = GenericMapper.map(fruits, s -> s.toUpperCase());
        System.out.println("upperFruits = " + upperFruits);

        // String -> Integer
        List<Integer> lengthFruits = GenericMapper.map(fruits, s -> s.length());
        System.out.println("lengthFruits = " + lengthFruits);

        // Integer -> String
        List<Integer> integers = List.of(1, 2, 3);
        List<String> starList = GenericMapper.map(integers, n -> "*".repeat(n));
        System.out.println("starList = " + starList);
    }
}
```

```java
package lambda.lambda5.map;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;

public class GenericMapper {

    public static<T,R> List<R> map(List<T> list, Function<T, R> mapper) {
        List<R> numbers = new ArrayList<>();
        for (T t : list) {
            numbers.add(mapper.apply(t));
        }
        return numbers;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.36.25.png" alt=""><figcaption></figcaption></figure>

* 제네릭을 도입한 덕분에 다양한 타입의 리스트의 값을 변환(매핑) 하여 사용할 수 있게 되었다.&#x20;
* `GenericMapper` 는 제네릭을 사용할 수 있는 모든 타입의 리스트를 람다 조건으로 변환(매핑) 할 수 있다. \
  따라서, 매우 유연한 매핑(변환) 기능을 제공한다.&#x20;

## 필터와 맵 활용1&#x20;

### 필터와 맵 활용 - 문제1

* 리스트에 있는 값 중에 짝수만 남기고, 남은 짝수 값의 2배를 반환하라.&#x20;
* `direct()` 에 람다, 앞서 작성한 유틸리티를 사용하지말고 `for`, `if` 등으로 코드를 직접 작성해라.&#x20;
*   `lambda()` 에 앞서 작성한 필터와 맵 유틸리티를 사용해서 코드를 작성해라.&#x20;

    ```java
    package lambda.lambda5.mystream;

    import lambda.lambda5.filter.GenericFilter;
    import lambda.lambda5.map.GenericMapper;

    import java.util.ArrayList;
    import java.util.List;

    public class Ex1_Number {

        public static void main(String[] args) {
            // 짝수만 남기고, 남은 값의 2배를 반환
            List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

            List<Integer> directResult = direct(numbers);
            System.out.println("directResult = " + directResult);

            List<Integer> lambdaResult = lambda(numbers);
            System.out.println("lambdaResult = " + lambdaResult);
        }
        static List<Integer> direct(List<Integer> numbers) {
            List<Integer> result = new ArrayList<>();

            for (Integer number : numbers) {
                if (number % 2 == 0) {
                    result.add(number * 2);
                }
            }
            return result;
        }

        static List<Integer> lambda(List<Integer> numbers) {
            List<Integer> filteredList = GenericFilter.filter(numbers, n -> n % 2 == 0);
            List<Integer> mappedList = GenericMapper.map(filteredList, n -> n * 2);
            return mappedList;
        }
    }
    ```



<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.39.22.png" alt=""><figcaption></figcaption></figure>

`direct()`, `lambda()` 는 서로 다른 프로그래밍 스타일을 보여준다.&#x20;

`direct()` 는 프로그램을 **어떻게** 수행해야 하는지 수행 절차를 명시한다.&#x20;

* 쉽게 이야기해서 개발자가 로직 하나하나를 **어떻게** 실행해야 하는지 명시한다.&#x20;
* 이런 프로그래밍 방식을 **명령형 프로그래밍**이라 한다.&#x20;
* 명령형 스타일은 익숙하고 직관적이나, 로직이 복잡해질수록 반복 코드가 많아질 수 있다.

`lambda()` 는 무엇을 수행해야 하는지 원하는 결과에 초점을 맞춘다.&#x20;

* 쉽게 이야기해서 특정 조건어로 필터하고, 변환하라고 선언하면, 구체적인 부분은 내부에서 수행된다.&#x20;
* 개발자는 필터에서 변환하는 것 즉 **무엇을** 해야 하는가에 초점을 맞춘다.
* 이런 프로그래밍 방식을 **선언적 프로그래밍**이라 한다.
* 선언형 스타일은 `무엇을` 하고자 하는지가 명확하게 드러난다. 따라서 코드 가독성과 유지보수가 쉬워진다.

### 명령형 vs 선언적 프로그래밍&#x20;

#### 명령형 프로그래밍(Imperative Programming)&#x20;

* 정의 : 프로그램이 **어떻게(HOW)** 수행되어야 하는지, 즉 **수행 절차**를 명시하는 방식이다.&#x20;
* 특징&#x20;
  * **단계별 실행**: 프로그램의 각 단계를 명확하게 지정하고 순서대로 실행한다.
  * **상태 변화**: 프로그램의 상태(변수 값 등)가 각 단계별로 어떻게 변화하는지 명시한다.
  * **낮은 추상화**: 내부 구현을 직접 제어해야 하므로 추상화 수준이 낮다.
  * **예시**: 전통적인 `for` 루프, `while` 루프 등을 명시적으로 사용하는 방식
* **장점**: 시스템의 상태와 흐름을 세밀하게 제어할 수 있다.

#### 선언적 프로그래밍(Declarative Programming)&#x20;

* 정의 : 프로그램이 **무엇(WHAT)**&#xC744; 수행해야 하는지, 즉 **원하는 결과**를 명시하는 방식이다.
* 특징&#x20;
  * **문제 해결에 집중**: 어떻게 문제를 해결할지보다 **무엇**을 원하는지에 초점을 맞춘다.
  * **코드 간결성**: 간결하고 읽기 쉬운 코드를 작성할 수 있다.
  * **높은 추상화**: 내부 구현을 숨기고 원하는 결과에 집중할 수 있도록 추상화 수준을 높인다.
  * **예시**: `filter`, `map` 등 람다의 고차 함수를 활용, HTML, SQL 등
* **장점**: 코드가 간결하고, 의도가 명확하며, 유지보수가 쉬운 경우가 많다.

#### 정리&#x20;

* **명령형 프로그래밍**은 프로그램이 수행해야 할 각 단계와 처리 과정을 상세하게 기술하여, **어떻게** 결과에 도달할지를 명시한다.&#x20;
* **선언적 프로그래밍**은 원하는 결과나 상태를 기술하며, 그 결과를 얻기 위한 내부 처리 방식은 추상화되어 있어 개발자가 **무엇을** 원하는지에 집중할 수 있게 한다.&#x20;
* 특히, **람다**와 같은 도구를 사용하면, 코드를 간결하게 작성하여 선언적 스타일로 문제를 해결할 수 있다.

## 필터와 맵 활용2&#x20;

```java
package lambda.lambda5.mystream;

public class Student {

    private String name;
    private int score;

    public Student(String name, int score) {
        this.name = name;
        this.score = score;
    }

    public String getName() {
        return name;
    }

    public int getScore() {
        return score;
    }

    @Override
    public String toString() {
        return "Student{" +
                "name='" + name + '\'' +
                ", score=" + score +
                '}';
    }
}
```

### 필터와 맵 활용 - 문제2&#x20;

앞서 만든 필터와 맵을 함께 활용해서 문제를 풀어보자.

* 점수가 80점 이상인 학생의 이름을 추출해라.
* `direct()` 에 람다를 사용하지 않고 `for`, `if` 등의 코드를 직접 작성해라.
* `lambda()` 에 앞서 작성한 필터와 맵을 사용해서 코드를 작성해라.

```java
package lambda.lambda5.mystream;

import lambda.lambda5.filter.GenericFilter;
import lambda.lambda5.map.GenericMapper;
import java.util.ArrayList;
import java.util.List;

public class Ex2_Student {

    public static void main(String[] args) {

        // 점수가 80점 이상인 학생의 이름을 추출해라.
        List<Student> students = List.of(
                new Student("Apple", 100),
                new Student("Banana", 80),
                new Student("Berry", 50),
                new Student("Tomato", 40)
        );

        List<String> directResult = direct(students);
        System.out.println("directResult = " + directResult);

        List<String> lambdaResult = lambda(students);
        System.out.println("lambdaResult = " + lambdaResult);
    }
    private static List<String> direct(List<Student> students) {
        List<String> highScoreNames = new ArrayList<>();

        for (Student student : students) {
            if (student.getScore() >= 80) {
                highScoreNames.add(student.getName());
            }
        }
        return highScoreNames;
    }

    private static List<String> lambda(List<Student> students) {
        List<Student> filtered = GenericFilter.filter(students, student -> student.getScore() >= 80);
        List<String> mapped = GenericMapper.map(filtered, student -> student.getName());
        return mapped;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 15.50.08.png" alt=""><figcaption></figcaption></figure>

`direct()` 는 어떻게 수행해야 하는지 수행 절차를 명시한다.&#x20;

`lambda()` 코드는 선언적이다.&#x20;

람다응 사용한 덕분에, 코드를 간결하게 작성하고, 선언적 스타일로 문제를 해결할 수 있었다.&#x20;

## 스트림 만들기1

지금까지는 필터와 맵 기능을 별도의 유틸리티에서 각각 따로 제공했다.&#x20;

그래서 두 기능을 함께 사용할 때, 필터링 된 결과를 다시 맵에 전달하는 번거로운 과정을 거쳐야했다.&#x20;

이번에는 앞서 만든 필터와 맵을 함께 편리하게 사용할 수 있도록 하나의 객체에 기능을 통합해보자.&#x20;

### 스트림1&#x20;

필터와 맵을 사용할 때를 떠올려보면 데이터들이 흘러가면서 필터되고, 매핑된다. 그래서 마치 데이터가 물 흐르듯이 흘러간다는 느낌을 받았을 것이다. 참고로 흐르는 좁은 시냇물을 영어로 스트림이라고 한다.&#x20;

이렇듯 데이터가 흘러가면서 필터도 되고, 매핑도 되는 클래스의 이름을 스트림(`Stream` )이라고 짓자.

```java
package lambda.lambda5.mystream;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;

public class MyStreamV1 {

    private List<Integer> internalList;

    public MyStreamV1(List<Integer> internalList) {
        this.internalList = internalList;
    }

    public MyStreamV1 filter(Predicate<Integer> predicate) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer element : internalList) {
            if (predicate.test(element)) {
                filtered.add(element);
            }
        }
        return new MyStreamV1(filtered);
    }

    public MyStreamV1 map(Function<Integer, Integer> mapper) {
        List<Integer> mapped = new ArrayList<>();
        for (Integer element : internalList) {
            mapped.add(mapper.apply(element));
        }
        return new MyStreamV1(mapped);
    }

    public List<Integer> toList() {
        return internalList;
    }
}
```

* 예제에서 스트림은 자신의 데이터 리스트를 가진다. 여기서는 쉽게 설명하기 위해 Integer 를 사용했다.&#x20;
* 스트림은 자신의 데이터를 필터(filter) 하거나 매핑(map) 해서 새로운 스트림을 만들 수 있다.&#x20;
* 스트림은 내부의 데이터 리스트를 toList() 로 반환할 수 있다.&#x20;
*   `filter()`, `map()` 에 앞서 개발한 `GenericFilter`, `GenericMapper` 의 기능을 사용해도 되지만, 여기서

    는 직접 작성하겠다.

이렇게 만든 스트림을 사용해보자.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamV1Main {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        returnValue(numbers);
        methodChain(numbers);
    }

    private static void returnValue(List<Integer> numbers) {
        MyStreamV1 stream = new MyStreamV1(numbers);
        MyStreamV1 filteredStream = stream.filter(n -> n % 2 == 0);
        MyStreamV1 mappedStream = filteredStream.map(n -> n * 2);
        List<Integer> result = mappedStream.toList();
        System.out.println("result = " + result);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.12.17.png" alt=""><figcaption></figcaption></figure>

### 메서드 체인&#x20;

`returnValue()` 코드를 보자.&#x20;

```java
private static void returnValue(List<Integer> numbers) {
    MyStreamV1 stream = new MyStreamV1(numbers);
    MyStreamV1 filteredStream = stream.filter(n -> n % 2 == 0);
    MyStreamV1 mappedStream = filteredStream.map(n -> n * 2);
    List<Integer> result = mappedStream.toList();
    System.out.println("result = " + result);
}
```

* 스트림 객체를 통해 필터와 맵을 편리하게 사용할 수 있는 것은 맞지만, 이전에 `GenericFilter`, `GenericMapper` 와 비교해서 크게 편리해진 것 같지는 않다.&#x20;

#### 이전에 사용한 코드&#x20;

```java
List<Student> filtered = GenericFilter.filter(students, s -> s.getScore() >= 80);
List<String> mapped = GenericMapper.map(filtered, s -> s.getName());
```

우리가 만든 `MyStreamV1` 은 `filter`, `map` 을 호출할 때 자기 자신의 타입을 반환한다. 따라서 자기 자신의 메서드를 연결해서 호출할 수 있다.

#### methodChain() 을 추가하자.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamV1Main {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        returnValue(numbers);
        methodChain(numbers);
    }

    private static void returnValue(List<Integer> numbers) {
        MyStreamV1 stream = new MyStreamV1(numbers);
        MyStreamV1 filteredStream = stream.filter(n -> n % 2 == 0);
        MyStreamV1 mappedStream = filteredStream.map(n -> n * 2);
        List<Integer> result = mappedStream.toList();
        System.out.println("result = " + result);
    }

    private static void methodChain(List<Integer> numbers) {
        List<Integer> result = new MyStreamV1(numbers)
                .filter(n1 -> n1 % 2 == 0)
                .map(n -> n * 2)
                .toList();
        System.out.println("result = " + result);
    }
}
```

자기 자신의 타입을 반환한 덕분에 메서드를 연결하는 메서드 체인 방식을 사용할 수 있다.&#x20;

덕분에 지저분한 변수들을 제거하고, 깔끔하게 필터와 맵을 사용할 수 있게 되었다.&#x20;

참고로 `methodChain()` 의 작동 방식은 `returnValue()` 와 완전히 동일하다. 단지 중간 변수들이 없을 뿐이다.

## 스트림 만들기2&#x20;

### 스트림2&#x20;

```java
package lambda.lambda5.mystream;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;

// static factory 추가
public class MyStreamV2 {

    private List<Integer> internalList;

    private MyStreamV2(List<Integer> internalList) {
        this.internalList = internalList;
    }

    // static factory
    public static MyStreamV2 of(List<Integer> internalList) {
        return new MyStreamV2(internalList);
    }

    public MyStreamV2 filter(Predicate<Integer> predicate) {
        List<Integer> filtered = new ArrayList<>();
        for (Integer element : internalList) {
            if (predicate.test(element)) {
                filtered.add(element);
            }
        }
        return new MyStreamV2(filtered);
    }

    public MyStreamV2 map(Function<Integer, Integer> mapper) {
        List<Integer> mapped = new ArrayList<>();
        for (Integer element : internalList) {
            mapped.add(mapper.apply(element));
        }
        return new MyStreamV2(mapped);
    }

    public List<Integer> toList() {
        return internalList;
    }
}
```

* 기존 생성자를 외부에서 사용하지 못하도록 `private` 으로 설정했다.&#x20;
* 이제 `MyStreamV2` 를 생성하려면 `of()` 메서드를 사용해야 한다.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamV2Main {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        List<Integer> result = MyStreamV2.of(numbers)
                .filter(n -> n % 2 == 0)
                .map(n -> n * 2)
                .toList();
        System.out.println("result = " + result);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.20.25.png" alt=""><figcaption></figcaption></figure>

### 정적 팩토리 메서드 - static factory method&#x20;

정적 펙토리 메서드는 객체 생성을 담당하는 static 메서드로, 생성자(constructor) 대신 인스턴스를 생성하고 반환하는 역할을 한다. 즉, 일반적인 생성자(Constructor) 대신에 클래스의 인스턴스를 생성하고 초기화하는 로직을 캡슐화하여 제공하는 정적(static) 메서드이다.&#x20;

주요 특징은 다음과 같다.&#x20;

* **정적 메서드**: 클래스 레벨에서 호출되며, 인스턴스 생성 없이 접근할 수 있다.
* **객체 반환**: 내부에서 생성한 객체(또는 이미 존재하는 객체)를 반환한다.
*   **생성자 대체**: 생성자와 달리 메서드 이름을 명시할 수 있어, 생성 과정의 목적이나 특징을 명확하게 표현할 수 있

    다.
* **유연한 구현**: 객체 생성 과정에서 캐싱, 객체 재활용, 하위 타입 객체 반환 등 다양한 로직을 적용할 수 있다.

생성자는 이름을 부여할 수 없다. 반면에 정적 펙토리 메서드는 의미있는 이름을 부여할 수 있어, 가독성이 더 좋아지는 장점이 있다. 참고로 인자들을 받아 간단하게 객체를 생성할 때는 주로 of(...) 라는 이름을 사용한다.&#x20;

#### 예시) 회원 등급별 생성자가 다른 경우

```java
// 일반 회원 가입시 이름, 나이, 등급
new Member("회원1", 20, NORMAL);

// VIP 회원 가입시 이름, 나이, 등급, 선물 주소지
new Member("회원1", 20, VIP, "선물 주소지");
```

* 예를 들어, VIP 회원의 경우 객체 생성 시 선물 주소지가 추가로 포함된다고 가정하자.&#x20;
* 이런 부분을 생성자만 사용해서 처리하기는 헷갈릴 수 있다.&#x20;

```java
// 일반 회원 가입시 인자 2개
Member.createNormal("회원1", 20)

// VIP 회원 가입시 인자 3개
Member.createVip("회원2", 20, "선물 주소지")
```

* 정적 팩토리를 사용하면 메서드 이름으로 명확하게 회원과 각 회원에 따른 인자를 구분할 수 있다.

추가로 객체를 생성하기 전에 이미 있는 객체를 찾아서 반환하는 것도 가능하다.

* 예를 들어, `Integer.valueOf()` : `-128 ~ 127` 범위는 내부에 가지고 있는 `Integer` 객체를 반환한다.&#x20;

> 참고 : 정적 펙토리 메서드 패턴을 사용하면 생성자에 이름을 부여할 수 있기 때문에, 보통 가독성이 더 좋아진다. 하지만 반대로 이야기하면 이름도 부여해야 하고, 준비해야 하는 코드도 더 많다. 객체의 생성이 단순한 경우에는 생성자를 직접 사용하는 것이 단순함의 관점에서 보면 더 나은 선택일 수 있다.&#x20;
>
> 항상 개발은 트레이드오프를 고려해야 한다.&#x20;

## 스트림 만들기3&#x20;

### 스트림3&#x20;

```java
package lambda.lambda5.mystream;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;

// Generic 추가
public class MyStreamV3<T> {

    private List<T> internalList;

    private MyStreamV3(List<T> internalList) {
        this.internalList = internalList;
    }

    // static factory
    public static <T> MyStreamV3<T> of(List<T> internalList) {
        return new MyStreamV3<>(internalList);
    }

    public MyStreamV3<T> filter(Predicate<T> predicate) {
        List<T> filtered = new ArrayList<>();
        for (T element : internalList) {
            if (predicate.test(element)) {
                filtered.add(element);
            }
        }
        return MyStreamV3.of(filtered);
    }

    public <R> MyStreamV3<R> map(Function<T, R> mapper) {
        List<R> mapped = new ArrayList<>();
        for (T element : internalList) {
            mapped.add(mapper.apply(element));
        }
        return MyStreamV3.of(mapped);
    }

    public List<T> toList() {
        return internalList;
    }
}
```

* `MyStreamV3` 은 내부에 `List<T> internalList` 를 가진다. 따라서 `MyStreamV3<T>` 로 선언한다.&#x20;
* `map()` 은 `T` 를 다른 타입인 `R` 로 반환한다. `R` 을 사용하는 곳은 `map` 메서드 하나이므로 `map` 메서드 앞에 추가로 제네릭 `<R>` 을 선언한다.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamV3Main {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Student> students = List.of(
                new Student("Apple", 100),
                new Student("Banana", 80),
                new Student("Berry", 50),
                new Student("Tomato", 40)
        );

        // 점수가 80점 이상인 학생의 이름을 추출해라
        List<String> result1 = ex1(students);
        System.out.println("result1 = " + result1);

        // 점수가 80점 이상이면서, 이름이 5글자인 학생의 이름을 대문자로 추출해라.
        List<String> result2 = ex2(students);
        System.out.println("result2 = " + result2);
    }

    private static List<String> ex1(List<Student> students) {
        return MyStreamV3.of(students)
                .filter(s -> s.getScore() >= 80)
                .map(s -> s.getName())
                .toList();
    }

    private static List<String> ex2(List<Student> students) {
        return MyStreamV3.of(students)
                .filter(s -> s.getScore() >= 80)
                .filter(s -> s.getName().length() == 5)
                .map(s -> s.getName())
                .map(name -> name.toUpperCase())
                .toList();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.30.22.png" alt=""><figcaption></figcaption></figure>

* 제네릭을 도입한 덕분에 `MyStreamV3` 은 `Student` 를 `String` 으로 변환할 수 있었다.&#x20;
* `ex2()` 는 필터와 맵을 연속해서 사용할 수 있다는 것을 보여주는 예다. 메서드 체인 덕분에 필요한 기능을 얼마든지 연결해서 사용할 수 있다.&#x20;

## 스트림 만들기4

이번에는 스트림의 최종 결과까지 스트림에서 함께 처리하도록 개선해보자.&#x20;

스트림의 최종 결과를 다음과 같이 하나씩 출력해향 하는 요구사항이 있다.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamLoopMain {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Student> students = List.of(
                new Student("Apple", 100),
                new Student("Banana", 80),
                new Student("Berry", 50),
                new Student("Tomato", 40)
        );

        List<String> result = MyStreamV3.of(students)
                .filter(s -> s.getScore() >= 80)
                .map(s -> s.getName())
                .toList();

        // 외부 반복
        for (String s : result) {
            System.out.println("name : " + s);
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.32.58.png" alt=""><figcaption></figcaption></figure>

이 경우 결과 리스트를 `for` 문을 통해서 하나씩 반복하며 출력하면 된다.&#x20;

그런데 생각해보면 `filter`, `map` 등도 스트림 안에서 데이터 리스트를 하나씩 처리(함수를 적용) 하는 기능이다. 따라서 최종 결과를 출력하는 일도 스트림 안에서 처리할 수 있을 것 같다.

기존 `MyStreamV3` 에 `forEach()` 라는 메서드를 추가하자.&#x20;

```java
package lambda.lambda5.mystream;

import java.util.ArrayList;
import java.util.List;
import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;

// Generic 추가
public class MyStreamV3<T> {

    private List<T> internalList;

    private MyStreamV3(List<T> internalList) {
        this.internalList = internalList;
    }

    // static factory
    public static <T> MyStreamV3<T> of(List<T> internalList) {
        return new MyStreamV3<>(internalList);
    }

    public MyStreamV3<T> filter(Predicate<T> predicate) {
        List<T> filtered = new ArrayList<>();
        for (T element : internalList) {
            if (predicate.test(element)) {
                filtered.add(element);
            }
        }
        return MyStreamV3.of(filtered);
    }

    public <R> MyStreamV3<R> map(Function<T, R> mapper) {
        List<R> mapped = new ArrayList<>();
        for (T element : internalList) {
            mapped.add(mapper.apply(element));
        }
        return MyStreamV3.of(mapped);
    }

    public List<T> toList() {
        return internalList;
    }

    // 추가
    public void forEach(Consumer<T> consumer) {
        for (T element : internalList) {
            consumer.accept(element);
        }
    }
}
```

```java
package lambda.lambda5.mystream;

import java.util.List;

public class MyStreamLoopMain {

    public static void main(String[] args) {
        // 짝수만 남기고, 남은 값의 2배를 반환
        List<Student> students = List.of(
                new Student("Apple", 100),
                new Student("Banana", 80),
                new Student("Berry", 50),
                new Student("Tomato", 40)
        );

        List<String> result = MyStreamV3.of(students)
                .filter(s -> s.getScore() >= 80)
                .map(s -> s.getName())
                .toList();

        // 외부 반복
        for (String s : result) {
            System.out.println("name : " + s);
        }

        // 추가
        // 내부 반복
        MyStreamV3.of(students)
                .filter( s -> s.getScore() >= 80)
                .map(s -> s.getName())
                .forEach(name -> System.out.println("name : " + name));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-19 16.35.27.png" alt=""><figcaption></figcaption></figure>

### 내부 반복 vs 외부 반복&#x20;

스트림을 사용하기 전에 일반적인 반복 방식은 `for`문, `while`문과 같은 반복문을 직접 사용해서 데이터를 순회하는 \
**외부 반복(External Iteration)** 방식이었다. 예를 들어 다음 코드처럼 **개발자가 직접** 각 요소를 반복하며 처리한다.

#### 외부 반복&#x20;

```java
List<String> result = ...
for (String s : result) {
System.out.println("name: " + s);
}
```

스트림에서 제공하는 `forEach()` 메서드로 데이터를 처리하는 방식은 **내부 반복(Internal Iteration)** 이라고 부른다. 외부 반복처럼 직접 반복 제어문을 작성하지 않고, 반복 처리를 스트림 내부에 위임하는 방식이다. **스트림 내부에서** 요소들을 순회하고, 우리는 처리 로직(람다) 만 정의해주면 된다.

#### 내부 반복&#x20;

```java
MyStreamV3.of(students)
    .filter( s -> s.getScore() >= 80)
    .map(s -> s.getName())
    .forEach(name -> System.out.println("name : " + name));
```

* 반복 제어를 스트림이 대신 수행하므로, 사용자는 반복 로직을 신경 쓸 필요가 없다.
* 코드가 훨씬 간결해지며, 선언형 프로그래밍 스타일을 적용할 수 있다.

#### 정리&#x20;

* 내부 반복 방식은 반복의 제어를 스트림에게 위임하기 때문에 코드가 간결해진다. **즉, 개발자는 "어떤 작업" 을 할지를 집중적으로 작성하고, "어떻게 순회할지" 는 스트림이 담당**하도록 하여 생산성과 가독성을 높일 수 있다. 한마디로 **선언형 프로그래밍 스타일이다.**&#x20;
* **외부 반복**은 개발자가 직접 반복 구조를 제어하는 반면, **내부 반복**은 반복을 내부에서 처리한다. 따라서 코드의 가독성과 유지보수성을 향상시킨다.

### 내부 반복 vs 외부 반복 선택&#x20;

많은 경우 내부 반복을 사용할 수 있다면 내부 반복이 선언형 프로그래밍 스타일로 직관적이기 때문에 더 나은 선택이다. 다만 때때로 외부 반복을 선택하는 것이 더 나은 경우도 있다.&#x20;

#### 외부 반복을 선택하는 것이 더 나은 경우&#x20;

* 단순히 한두 줄 수행만 필요한 경우&#x20;
* 반복 제어에 대한 복잡하고 세밀한 조정이 필요할 경우&#x20;
