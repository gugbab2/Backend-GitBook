# 9. 컬렉션 프레임워크 - Map, Stack, Queue

## 컬렉션 프레임워크 - Map 소개1&#x20;

`Map` 은 키-값의 쌍을 저장하는 자료 구조이다.

* 키는 맵 내에서 유일해야 한다. 그리고 키를 통해 값을 빠르게 검색할 수 있다.&#x20;
* 키는 중복될 수 없지만, 값은 중복될 수 있다.
* `Map` 은 순서를 유지하지 않는다.

자바는 `HashMap`, `TreeMap`, `LinkedHashMap` 등 다양한 Map 구현체를 제공한다. 이들은 `Map` 인터페이스의 메서드를 구현하며, 각기 다른 특성과 성능 특징을 가지고 있다.&#x20;

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 14.42.04.png" alt=""><figcaption></figcaption></figure>

이 중에 `HashMap` 을 가장 많이 사용하며, 자세한 예제는 아래와 같다.&#x20;

```java
package collection.map;

import java.util.Collection;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class MapMain1 {

    public static void main(String[] args) {
        Map<String, Integer> studentMap = new HashMap<>();

        studentMap.put("studentA", 90);
        studentMap.put("studentB", 80);
        studentMap.put("studentC", 80);
        studentMap.put("studentD", 100);
        System.out.println("studentMap = " + studentMap);

        Integer result = studentMap.get("studentD");
        System.out.println("result = " + result);

        System.out.println("==KeySet 활용==");
        Set<String> keySet = studentMap.keySet();
        for (String key : keySet) {
            Integer value = studentMap.get(key);
            System.out.println("key = " + key + ", value = " + value);
        }

        System.out.println("==entrySet 활용==");
        Set<Map.Entry<String, Integer>> entries = studentMap.entrySet();
        for (Map.Entry<String, Integer> entry : entries) {
            String key = entry.getKey();
            Integer value = entry.getValue();
            System.out.println("key = " + key + ", value = " + value);
        }

        System.out.println("==values 활용==");
        Collection<Integer> values = studentMap.values();
        for (Integer value : values) {
            System.out.println("value = " + value);
        }
    }
}
```

#### 키 목록 조회&#x20;

`Set<String> keySet = studentMap.keySet()`

`Map` 의 키는 중복을 허용하지 않는다. 따라서 `Map` 의 모든 키 목록을 조회하는 `keySet()` 을 호출하면, 중복을 허용하지 않는 자료 구조인 `Set` 을 반환한다.

#### 키와 값 목록 조회 (Entry Key-Value Pair)&#x20;

`Map`은 키와 값을 보관하는 자료구조이다. 따라서 키와 값을 하나로 묶을 수 있는 방법이 필요하다.\
이때 `Entry`를 사용한다.

`Entry` 는 키-값의 쌍으로 이루어진 간단한 객체이다. `Entiry` 는 `Map` 내부에서 키와 값을 함께 묶어서 저장할 때 \
사용한다.

쉽게 이야기해서 우리가 `Map`에 키와 값으로 데이터를 저장하면 `Map`은 내부에서 키와 값을 하나로 묶는 `Entry` 객체 를 만들어서 보관한다. 참고로 하나의 `Map` 에 여러 `Entry` 가 저장될 수 있다.

참고로 `Entry` 는 `Map` 내부에 있는 인터페이스이다. 우리는 구현체보다는 이 인터페이스를 사용하면 된다.

#### 값 목록 조회&#x20;

`Collection<Integer> values = studentMap.values()`

`Map`의 값 목록은 중복을 허용한다. 따라서 중복을 허용하지 않는 `Set`으로 반환할 수는 없다. 그리고 입력 순서를 보장하지 않기 때문에 순서를 보장하는 `List`로 반환하기도 애매하다.&#x20;

따라서 단순히 값의 모음이라는 의미의 상위 인터 페이스인 `Collection` 으로 반환한다.

## 컬렉션 프레임워크 - Map 소개2

```java
package collection.map;

import java.util.HashMap;
import java.util.Map;

public class MapMain2 {

    public static void main(String[] args) {
        Map<String, Integer> studentMap = new HashMap<>();

        studentMap.put("studentA", 90);
        System.out.println("studentMap = " + studentMap);

        // 같은 키에 저장시 기존 값 교체
        studentMap.put("studentA", 100);
        System.out.println("studentMap = " + studentMap);

        boolean containsKey = studentMap.containsKey("studentA");
        System.out.println("containsKey = " + containsKey);

        studentMap.remove("studentA");
        System.out.println("studentMap = " + studentMap);
    }
}
```

```java
package collection.map;

import java.util.HashMap;
import java.util.Map;
import java.util.Set;

public class MapMain3 {

    public static void main(String[] args) {
        Map<String, Integer> studentMap = new HashMap<>();

        studentMap.put("studentA", 50);
        System.out.println(studentMap);

        if (!studentMap.containsKey("studentA")) {
            studentMap.put("studentA", 100);
        }
        System.out.println(studentMap);

        studentMap.putIfAbsent("studentA", 100);
        studentMap.putIfAbsent("studentB", 100);
        System.out.println(studentMap);
    }
}
```

## 컬렉션 프레임워크 - Map 구현체&#x20;

자바의 `Map` 인터페이스는 키-값 쌍을 저장하는 자료 구조이다. `Map`은 인터페이스이기 때문에, 직접 인스턴스를 생성 할 수는 없고, 대신 `Map` 인터페이스를 구현한 여러 클래스를 통해 사용할 수 있다.&#x20;

대표적으로 `HashMap`, `TreeMap`, `LinkedHashMap`이 있다.

### Map vs Set&#x20;

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 14.51.33.png" alt="" width="439"><figcaption></figcaption></figure>

그런데 `Map`을 어디서 많이 본 것 같지 않은가? `Map`의 키는 중복을 허용하지 않고, 순서를 보장하지 않는다.

`Map` 의 키가 바로 `Set` 과 같은 구조이다. 그리고 `Map` 은 모든 것이 `Key` 를 중심으로 동작한다.`Value` 는 단순히 `Key` 옆에 따라 붙은 것 뿐이다. `Key` 옆에 `Value` 만 하나 추가해주면 `Map` 이 되는 것이다.&#x20;

`Map` 과 `Set` 은 거의 같다. 단지 옆에 `Value` 를 가지고 있는가 없는가의 차이가 있을 뿐이다.

#### 참고 : 실제로 자바 `HashSet` 의 구현은 대부분 `HashMap` 의 구현을 가져다 사용한다. `Map` 에서 `Value` 만 비워두면 `Set` 으로 사용할 수 있다.

### 1. HashMap

* **구조**: `HashMap` 은 해시를 사용해서 요소를 저장한다. 키( `Key` ) 값은 해시 함수를 통해 해시 코드로 변환되고, 이 해시 코드는 데이터를 저장하고 검색하는 데 사용된다.
* **특징**: 삽입, 삭제, 검색 작업은 해시 자료 구조를 사용하므로 일반적으로 상수 시간( `O(1)` )의 복잡도를 가진다.&#x20;
* **순서**: 순서를 보장하지 않는다.

### 2. LinkedHashMap

* **구조**: `LinkedHashMap`은 `HashMap`과 유사하지만, 연결 리스트를 사용하여 삽입 순서 또는 최근 접근 순서에 따라 요소를 유지한다.
* **특징**: 입력 순서에 따라 순회가 가능하다. `HashMap` 과 같지만 입력 순서를 링크로 유지해야 하므로 조금 더 무겁다.
* **성능**: `HashMap` 과 유사하게 대부분의 작업은 `O(1)`의 시간 복잡도를 가진다.&#x20;
* **순서**: 입력 순서를 보장한다.

### 3. TreeMap&#x20;

* **구조**: `TreeMap` 은 레드-블랙 트리를 기반으로 한 구현이다.
* **특징**: 모든 키는 자연 순서 또는 생성자에 제공된 `Comparator` 에 의해 정렬된다.
* **성능**: `get` , `put` , `remove` 와 같은 주요 작업들은 `O(log n)` 의 시간 복잡도를 가진다.&#x20;
* **순서**: 키는 정렬된 순서로 저장된다.

```java
package collection.map;

import java.util.*;

public class JavaMapMain {

    public static void main(String[] args) {
        run(new HashMap<>());
        run(new LinkedHashMap<>());
        run(new TreeMap<>());
    }

    private static void run(Map<String, Integer> map) {
        System.out.println("map = " + map.getClass());
        map.put("C", 10);
        map.put("B", 20);
        map.put("A", 30);
        map.put("1", 40);
        map.put("2", 50);

        Set<String> keySet = map.keySet();
        Iterator<String> iterator = keySet.iterator();
        while (iterator.hasNext()) {
            String key = iterator.next();
            System.out.print(key + " = " + map.get(key) + " ");
        }
        System.out.println();
    }
}
```

#### 실행 결과

* `HashMap`: 입력한 순서를 보장하지 않는다.
* `LinkedHashMap`: 키를 기준으로 입력한 순서를 보장한다.
* `TreeMap` : 키 자체의 데이터 값을 기준으로 정렬한다.

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 14.55.55.png" alt=""><figcaption></figcaption></figure>

### 자바 HashMap 작동 원리

자바의 `HashMap`은 `HashSet`과 작동 원리가 같다.

`Set` 과 비교하면 다음과 같은 차이가 있다.

* `Key`를 사용해서 해시 코드를 생성한다.
* `Key`뿐만 아니라 값( `Value` )을 추가로 저장해야 하기 때문에 `Entry` 를 사용해서 `Key`, `Value` 를 하나로 \
  묶어서 저장한다.

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 14.58.36.png" alt="" width="440"><figcaption></figcaption></figure>

때문에 `Map` 의 `Key` 로 사용되는 객체는 `hashCode()` , `equals()` 를 반드시 구현해야 한다.

## Deque 자료 구조&#x20;

"Deque"는 "Double Ended Queue"의 약자로, 이 이름에서 알 수 있듯이, Deque는 양쪽 끝에서 요소를 추가하거나 \
제거할 수 있다. Deque는 일반적인 큐(Queue)와 스택(Stack)의 기능을 모두 포함하고 있어, 매우 유연한 자료 구조이다.

데크, 덱 등으로 부른다.

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 15.02.39.png" alt=""><figcaption></figcaption></figure>

### Deque 구현체와 성능 테스트&#x20;

`Deque`의 대표적인 구현체는 `ArrayDeque`, `LinkedList` 가 있다. \
이 둘 중에 `ArrayDeque`가 모든 면에서 더 빠르다.

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-21 15.03.52.png" alt=""><figcaption></figcaption></figure>

둘의 차이는 `ArrayList` vs `LinkedList`의 차이와 비슷한데, \
작동 원리가 하나는 배열을 하나는 동적 노드 링크를 사용하기 때문이다.

`ArrayDeque`는 추가로 특별한 원형 큐 자료 구조를 사용하는데, 덕분에 앞, 뒤 입력 모두 O(1)의 성능을 제공한다. 물론 `LinkedList`도 앞 뒤 입력 모두 O(1)의 성능을 제공한다.\
이론적으로 `LinkedList`가 삽입 삭제가 자주 발생할 때 더 효율적일 수 있지만, 현대 컴퓨터 시스템의 메모리 접근 패턴, CPU 캐시 최적화 등을 고려할 때 배열을 사용하는 `ArrayDeque`가 실제 사용 환경에서 더 나은 성능을 보여주는 경우가 많다.
