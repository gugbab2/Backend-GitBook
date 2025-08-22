# HashSet 은 내부에서 HashMap 사용한다.

#### 참고 링크

* [https://jay-ya.tistory.com/118](https://jay-ya.tistory.com/118)

## 1. HashSet 은 내부적으로 HashMap 으로 동작한다.&#x20;

`Map` 은 Key, Value 의 쌍으로 값이 저장되는 형태이고, Key Object 는 중복을 허용하지 않고, Value Object 는 중복을 허용하고, 순서를 보장하지 않는 데이터 구조이다.&#x20;

아래는 `HashSet` 클래스의 일부이다.&#x20;

```java
public class HashSet<E> {
    private transient HashMap<E,Object> map;
    private static final Object PRESENT = new Object();

    public HashSet() {
        map = new HashMap<>();
    }

    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

위 코드를 통해서 `HashSet` 내부에는 `HashMap` 으로 구현되어 있다는 것과, `HashSet` 에 값이 저장되면 Key Object 값에 들어가고 Value Object 에는 더미 객체가 저장된다는 점을 알 수 있을 것이다.&#x20;

이것이 가능한 이유는 Map 의 Key 는 중복된 값이 들어가지 못한다는 점과 Set 에는 중복된 값이 들어가지 못하는 공통된 특징 때문이다.&#x20;

## 2. 그렇다면 왜?&#x20;

`HashSet`, `HashMap` 모두 내부에서 "해싱(Hashing)" 이라는 동일한 원리를 기반으로 한다. 요소를 저장하고 검색하기 위해서 해시 코드를 사용한다.&#x20;

**같이 기능을 수행하는데, 동일한 구현을 또 할 필요가 있을까?**&#x20;

이러한 이유로 `HashSet` 내부에서는 `HashMap` 을 사용한다. \
~~(내 추측이지만, 이거 외에는 이유가 생각나지 않는다..)~~
