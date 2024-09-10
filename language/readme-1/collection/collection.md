# Collection

<figure><img src="../../../.gitbook/assets/스크린샷 2024-08-15 13.15.52.png" alt=""><figcaption></figcaption></figure>

## Collection

\-> **Collection 을 상속하는 객체들은 for(String temp : list) 형식의 for 문을 사용 가능하다.**

```java
public interface Collection<E> extends Iterable<E>
```

* `Iterable` 인터페이스는 `Iterator` 메서드를 통해 `Iterator` 인터페이스를 리턴하고 해당 인터페이스는 다음의 메서드를 가지고 있다. (foreach 구문을 사용할 수 있다)
  * `hasNext()` : 추가 데이터가 있는지 확인한다.
  * `next()` : 현재 위치를 다음 요소로 넘기고 그 값을 리턴해준다.
* **`Collection` 인터페이스가 `Iterable` 인터페이스를 확장했다는 의미는, `Iterator` 인터페이스를 사용해 데이터를 순차적으로 가져올 수 있다는 의미가 된다.**\
  **-> 순서가 존재한다는 개념은 아니다.**&#x20;

### Collection 인터페이스에 선언된 주요 메서드 목록

* `boolean add(E e)` : 요소를 추가
* `boolean addAll(Collection c)` : 매개 변수로 넘어온 컬랙션의 모든 요소를 추가
* `void clear()` : 컬렉션에 있는 모든 요소 삭제
* `boolean contains(Object o)` : 매개 변수로 넘어온 객체가 해당 컬렉션에 있는지 확인한다.
* `boolean contailsAll(Collection c)` : 매개변수로 넘어온 객체들이 해당 컬랙션에 있는지 확인한다.
  * 매개변수로 넘어온 컬렉션에 있는 요소들과 동일한 값이 전부 있어야만 true 를 리턴한다.
* `boolean equals(Object o)` : 매개 변수로 넘어온 객체와 같은 객체인지 확인한다.
* `int hashCode()` : 해시코드 값을 리턴한다.
* `boolean isEmpty()` : 컬렉션이 비어있는지 확인.
* `Iterator iterator()` : 데이터를 한 건씩 처리하기 위한 Iterator 객체를 리턴한다.
* `boolean remove(Object o)` : 매개 변수와 동일한 객체를 삭제한다.
* `boolean remove(Collection)` : 매개변수로 넘어온 객체들을 해당 컬렉션에서 삭제한다.
* `boolean retainAll(Collection)` : 매개변수로 넘어온 객체들만을 컬렉션에 남겨둔다.
* `int size()` : 요소의 개수를 리턴한다.
* `Object[] toArray()` : 컬렉션에 있는 데이터들을 배열로 복사한다.
* `<T> T[] toArray(T[])` : 컬렉션에 있는 데이터들을 지정한 타입의 배열로 복사한다.
