# TreeSet

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html](https://docs.oracle.com/javase/8/docs/api/java/util/TreeSet.html)
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/NavigableSet.html](https://docs.oracle.com/javase/8/docs/api/java/util/NavigableSet.html)

## TreeSet

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractSet<E>
            java.util.TreeSet<E>
```

> `NavigableSet<E>` : `SortedSet` 을 확장한 인터페이스로 요소의 탐색을 더 쉽게 할 수 있는 메서드 지정
>
> `Cloneable` : `Object` 클래스의 `clone()` 메서드가 제대로 수행될 수 있음을 지정
>
> `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능

* `TreeSet` 은 내부적으로 `TreeMap` 을 호출합니다.&#x20;
* `TreeSet` 은 레드 블랙 트리 구조를 사용하므로 정렬을 유지하고, &#x20;
* 읽기, 삽입, 식제 연산의 시간복잡도가 O(log n)  이다 .
* `NavigableSet` 인터페이스에서 제공하는 메서드를 통해 정렬된  순서로 순회가 가능하다.&#x20;

## TreeSet 시간복잡도(레드 블랙 트리 사용으로 일관된 시간복잡도)&#x20;

### 생성&#x20;

* 최선 : O( log n)
  * 요소가 삽입되는 위치를 찾고, 트리의 균형을 맞추는 과정은 항상 O(log n) 이다.&#x20;
* 최악 : O(log n)&#x20;

### 읽기

* 최선 : O(log n)
  * 요소의 위치를 찾기 위해 트리를 탐색하는 과정은 O(log n)입니다.
* 최악 : O(log n) &#x20;

### 수정

* 중복을 허용하지 않기 때문에, 수정은 없다.&#x20;

### 삭제&#x20;

* 최선 : O(log n)
  * 요소가 삭제되는 위치를 찾고, 트리의 균형을 맞추는 과정은 항상 O(log n) 이다.&#x20;
* 최악 : O(log n) &#x20;
