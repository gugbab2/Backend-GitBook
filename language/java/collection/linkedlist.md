# LinkedList

## LinkedList 기본

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractList<E>
            java.util.AbstractSequentialList<E>
                java.util.LinkedList<E>
```

* AbstractList 와 AbstractSequentialList 의 차이는, add(), set(), remove() 의 구현방법이 상이하다는 정도이다.&#x20;

> **Serializable, Cloneable, Iterable\<E>, Collection\<E>, Deque\<E>, List\<E>, Queue\<E>**
>
> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Iterable\<E> : foreach 문장 사용 가능
>
> Collection\<E> : 여러개의 데이터를 한 데이터에 담아 처리할 메서드를 지정
>
> Deque\<E> : 맨 앞, 맨 뒤의 값을 용이하게 처리하는 큐와 관련된 메소드 지정
>
> List\<E> : 목록형 데이터를 처리하는 것과 관련된 메서드 지정
>
> Queue\<E> : 큐를 처리하는 것과 관련된 메서드 지정

## LinkedList 생성자&#x20;

* LinkedList() : 비어 있는 LinkedList 객체를 생성한다.&#x20;
* LinkedList(Collection\<? extemds E> c) : 매개 변수로 받은 컬렉션 객체의 데이터를 LinkedList 에 담는다. \
  **=> LinkedList 는 일반적인 배열 타입의 클래스와 다르게 생성자로 객체를 생성할 때 크기를 지정하지 않는다. 왜냐면 각 데이터들이 앞 뒤로 연결되는 구조이기 때문에 미리 공간을 만들어 놓을 필요가 없기 때문이다.**

## LinkedList 에 데이터를 담자 (다수의 인터페이스를 구현해 동일 기능의 메서드가 많다)

* **LinkedList 객체의 가장 앞에 데이터를 추가하는 메서드**
  * void addFirst(Object o)
  * boolean offerFirst(Object o)
  * void push(Object o )
* **LinkedList 객체의 가장 뒤에 데이터를 추가하는 메서드**
  * boolean add(Object o)
  * void addLast(Object o)&#x20;
  * boolean offer(Object o)
  * boolean offerLast(Object o)
* void add(int i, Object o) : 특정 위치에 추가
* Object set(int i, Object o) : 특정 위치에 수정
* boolean addAll(Collection c) : Collection 을 데이터를 추가한다.
* boolean addAll(int i, Collection c) : 특정 위치부터 Collection 데이터를 추가한다.\
  \=> **내부적으로 메서드를 구현할 때 앞에 추가하는 메서드는 addFirst() 를 호출하고** \
  &#x20;    **마지막에 추가하는 메서드는 add() or addLast() 를 호출한다.** \
  **=> add 관련 메서드를 호출하는 것이 혼란을 줄일 수 있는 방법이다.**&#x20;

## LinkedList 에서 데이터를 꺼내자

* LinkedList 객체의 맨 앞에 있는 데이터를 리턴한다.
  * Object getFirst()
  * Object peekFirst()
  * Object peek()
  * Object element()\
    \=> **내부적으로 getFirst() 메서드를 사용한다.**&#x20;
* LinkedList 객체의 맨 뒤에 있는 데이터를 리턴한다.
  * Object getLast()
  * Object peekLast()
* LinkedList 객체의 지정한 위치에 있는 데이터를 리턴한다.&#x20;
  * Object get(int i)&#x20;

## LinkedList 에 어떤 객체가 존재하는지 확인하자

* boolean  contains(Object o) : 매개 변수로 넘긴 데이터가 있을 경우 true 를 리턴
* int indexof(Object o) : 매개변수로 넘긴 데이터의 위치를 앞에서부터 검색
* int lastIndexof(Object o) : 매개변수로 넘긴 데이터의 위치를 뒤에서부터 검색

## LinkedList 에 데이터를 삭제하자&#x20;

* LinkedList 객체의 가장 앞에 있는 데이터를 삭제하고 리턴한다.
  * Object remove()
  * Object removeFirst()
  * Object poll()
  * Object pollFirst()
  * Object pop()
* LinkedList 객체의 가장 끝에 있는 데이터를 삭제하고 리턴한다.&#x20;
  * Object pollLast()
  * Object removeLast()
* 매개변수에 지정된 위치에 있는 데이터를 삭제한다.&#x20;
  * Object remove(int)
* 매개변수로 넘겨진 객체와 동일한 데이터 중 가장 앞에 데이터를 삭제
  * boolean remove(Object)\
    \=> **remove 가 붙이 메서드를 내부적으로 사용한다.**&#x20;

## LinkedList 데이터를 하나씩 검색하자

* ListIterator listIterator(int i) : 매개변수에 지정된 위치부터의 데이터를 검색하기 위한 ListIterator 객체를 리턴한다. \
  \-> ListIterator 는 Iterator 객체가 다음 데이터만 검색할 수 있다는 단점을 보완하여, 이전 데이터도 검색할 수 있는 이터레이터다. (next(), previous())
* Iterator descendingIterator() : LinkedList 의 데이터를 끝에서부터 검색하기 위한 Iterator 객체를 리턴한다.&#x20;
