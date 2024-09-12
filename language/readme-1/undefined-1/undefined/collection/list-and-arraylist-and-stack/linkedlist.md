# LinkedList

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedList.html)

## LinkedList

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractList<E>
            java.util.AbstractSequentialList<E>
                java.util.LinkedList<E>
```

* `AbstractList` 와 `AbstractSequentialList` 의 차이는, `add()`, `set()`, `remove()` 의 구현방법이 상이하다는 정도이다.

> `List<E>` : 목록형 데이터를 처리하는 것과 관련된 메서드 지정
>
> `Deque<E>` : Stack, Queue 와관련된 메소드 지정
>
> `Cloneable` : Object 클래스의 **clone() 메서드가 제대로 수행될 수 있음을 지정**
>
> `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능

## 성자

* `LinkedList()` : 비어 있는 `LinkedList` 객체를 생성한다.
* `LinkedList(Collection<? extemds E> c)` : 매개 변수로 받은 컬렉션 객체의 데이터를 `LinkedList` 에 담는다.
* **`LinkedList` 는 일반적인 배열 타입의 클래스와 다르게 생성자로 객체를 생성할 때 크기를 지정하지 않는다. 왜냐면 각 데이터들이 앞 뒤로 연결되는 구조이기 때문에 미리 공간을 만들어 놓을 필요가 없기 때문이다.**

## 데이터를 담자 (다수의 인터페이스를 구현해 동일 기능의 메서드가 많다)

* **`LinkedList` 객체의 가장 앞에 데이터를 추가하는 메서드**
  * `void addFirst(Object o)`
  * `boolean offerFirst(Object o)`
  * `void push(Object o )`
* **`LinkedList` 객체의 가장 뒤에 데이터를 추가하는 메서드**
  * `boolean add(Object o)`
  * `void addLast(Object o)`
  * `boolean offer(Object o)`
  * `boolean offerLast(Object o)`
* `void add(int i, Object o)` : 특정 위치에 추가
* `Object set(int i, Object o)` : 특정 위치에 수정
* `boolean addAll(Collection c)` : `Collection` 을 데이터를 추가한다.
* `boolean addAll(int i, Collection c)` : 특정 위치부터 `Collection` 데이터를 추가한다.
  * **내부적으로 메서드를 구현할 때 앞에 추가하는 메서드는 `addFirst()` 를 호출하고,**&#x20;
  * **마지막에 추가하는 메서드는 `add()`, `addLast()` 를 호출한다.**

## 데이터를 꺼내자

* `LinkedList` 객체의 맨 앞에 있는 데이터를 리턴한다.
  * `Object getFirst()`
  * `Object peekFirst()`
  * `Object peek()`
  * `Object element()`\
    \-> **내부적으로 getFirst() 메서드를 사용한다.**
* `LinkedList` 객체의 맨 뒤에 있는 데이터를 리턴한다.
  * `Object getLast()`
  * `Object peekLast()`
* `LinkedList` 객체의 지정한 위치에 있는 데이터를 리턴한다.
  * `Object get(int i)`

## 어떤 데이터가 존재하는지 확인하자

* `boolean contains(Object o)` : 매개 변수로 넘긴 데이터가 있을 경우 true 를 리턴
* `int indexof(Object o)` : 매개변수로 넘긴 데이터의 위치를 앞에서부터 검색
* `int lastIndexof(Object o)` : 매개변수로 넘긴 데이터의 위치를 뒤에서부터 검색

## 데이터를 삭제하자

* `LinkedList` 객체의 가장 앞에 있는 데이터를 삭제하고 리턴한다.
  * `Object remove()`
  * `Object removeFirst()`
  * `Object poll()`
  * `Object pollFirst()`
  * `Object pop()`
* `LinkedList` 객체의 가장 끝에 있는 데이터를 삭제하고 리턴한다.
  * `Object pollLast()`
  * `Object removeLast()`
* 매개변수에 지정된 위치에 있는 데이터를 삭제한다.
  * `Object remove(int)`
* 매개변수로 넘겨진 객체와 동일한 데이터 중 가장 앞에 데이터를 삭제
  * `boolean remove(Object)`

## 데이터를 하나씩 검색하자

* `ListIterator listIterator(int i)` : 매개변수에 지정된 위치부터의 데이터를 검색하기 위한 ListIterator 객체를 리턴한다.
  * **`ListIterator` 는 `Iterator` 객체가 다음 데이터만 검색할 수 있다는 단점을 보완하여, 이전 데이터도 검색할 수 있는 이터레이터다. (`next()`, `previous()`)**
* `Iterator descendingIterator()` : `LinkedList` 의 데이터를 끝에서부터 검색하기 위한 `Iterator` 객체를 리턴한다.

## LinkedList시간 복잡도(CRUD)&#x20;

### 생성&#x20;

* **최선**: O(1)
  * 레퍼런스를 연결하기 때문에, 어디에 생성하든지 시간 복잡도는 동일하다.&#x20;
* **최악**: O(1)

### 읽기

* **최선**: O(1)
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 앞부분에 있는 경우&#x20;
* **최악**: O(n)
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 뒷부분에 있는 경우&#x20;

### 수정

* 최선 : O(1)&#x20;
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 앞부분에 있는 경우
* 최악 : O(n)
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 뒷부분에 있는 경우

### 삭제&#x20;

* 최선 : O(1)&#x20;
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 앞부분에 있는 경우
* 최악 : O(n)
  * 앞부분부터 순차적으로 순회하기 때문에, 데이터가 뒷부분에 있는 경우
