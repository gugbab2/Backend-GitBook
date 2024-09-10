# ArrayList

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)

## ArrayList&#x20;

```java
java.lang.Object
    java.util.AbstracCollection<E>
        java.util.AbstractList<E>
            java.util.ArrayList<E>
```

* **`Object` -> `Collection` -> `List` -> `ArrayList` 순으로 확장한 것을 확인할 수 있다.**
* `ArrayList` 는 아래 인터페이스를 구현한다.
  * `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능
  * `Cloneable` : `Object` 클래스의 `clone()` 메서드가 제대로 수행될 수 있음을 지정, 복제가 가능한 객체
  * `Iterable<E>` : forEech 구문 사용 가능&#x20;
  * `Collection<E>` : 여러개의 데이터를 한 객체에 담아 처리할 메소드 지정
  * `List<E>` : 순서가 보장되며, 중복을 허용하는 목록형데이터 집합을 의미&#x20;
  * `RandomAccess` :
    * 컬렉션이 인덱스 기반으로 빠른 임의 접근(Random Access) 을 지원한다는 신호를 제공하는 역할을 한다.&#x20;
    * 즉, `ArrayList` 같은 컬렉션은 인덱스를 사용한 요소 접근(`get(index)` 또는 `set(index, value)`)이 상수 시간(O(1)) 안에 이루어진다는 것을 나타냅니다.
    * `RandomAccess` 는 마커 인터페이스이다.&#x20;

> #### 마커 인터페이스(Mark Interface)
>
> * Java에서 마커 인터페이스는 **구체적인 메서드를 포함하지 않으며**, 특정 클래스가 어떤 기능을 가지고 있음을 표시하기 위해 사용

### ArrayList 생성자

* `ArrayList()` : 객체를 저장할 공간이 10개인 ArrayList 객체를 만든다.
  * **10 이상의 데이터가 들어가면 크기를 늘리는 작업이 `ArrayList` 내부에서 자동으로 수행된다.**\
    **->** 객체가 생성될 때 배열이 생성되는 것이 아닌, 첫 번째 요소가 추가될 때 크기가 10인 배열이 생성된다.&#x20;
  * 또한 크기가 다 차면 1.5배씩 배열의 크기는 증가한다.\
    \-> 해당 작업은 애플리케이션 성능에 영향을 주는 작업으로, 크기가 예상이 된다면 초기에 지정해주는 것이 좋다.
* `ArrayList(Collection<? extend E> c)` : 매개변수로 넘어온 컬렉션 객체가 저장되어 있는 `ArrayList`를 만든다.
* `ArrayList(int initialCapacity)` : 매개변수만큼의 저장공간을 갖는 `ArrayList` 를 만든다.

### ArrayList 에 데이터를 담자

* `boolean add(E e)` : 매개변수로 들어온 데이터를 가장 끝에 담는다.
* `void add(int index, E e)` : 매개변수로 들어온 데이터를 지정된 index 위치에 담는다.
  * **해당 위치에 데이터를 추가하게 되면, 뒤의 데이터는 복사가 일어나게 된다.** \
    **-> `ArrayList` 의 가장 큰 문제점이다.**&#x20;
* `boolean addAll(Collection<? extends E> c)` : 매개변수로 넘어온 컬렉션 데이터를 가장 끝에 담는다.
  * **`Collection` 관련 객체를 복사할 일이 있을 때에는, 생성자 혹은 `addAll()` 메서드를 사용하는 것이 좋다.**
* `boolean addAll(int index, Collection<? extend E> c)` : 매개변수로 넘어온 컬렉션 데이터를 index 에 지정된 위치부터 담는다.

```java
public class ListSample {
    public static void main(String[] args){
        ListSample sample = new ListSample();
        sample.checkArrayList1();
    }

    public void checkArrayList1(){
        ArrayList<String> list1 = new ArrayList<>();    //초기의 크기는 10으로 
        list1.add("a");
        list1.add("b");
        list1.add("c");
        list1.add("d");
        list1.add("e");
        list1.add(1, "A");
        
        for(String temp : list1){
            System.out.println("temp = " + temp);
        }
    }

```

### ArrayList 에서 데이터를 꺼내자

* `Int size()` : Collection 을 구현한 객체에 들어가 있는 데이터의 갯수 리턴.
  * `배열.length` 는 배열의 저장 공간 개수를 의미하지만,
  * `size()` 메서드는 객체 안에 들어가 있는 데이터의 갯수를 의미한다.
* `E get(int index)` : 매개변수에 지정한 위치에 있는 데이터 리턴.
* `int indexOf(Object o)` : 매개변수로 지정한 객체와 동일한 데이터의 위치를 리턴.
* `int lastInedexOf(Object o)` : 매개변수로 지정한 객체와 동일한 마지막 데이터의 위치를 리턴.

```java
public void checkArrayList5(){
    ArrayList<String> list = new ArrayList<>();    //초기의 크기는 10으로
    list.add("A");
    list.add("B");
    int listSize = list.size();
    for(int i=0; i<listSize; i++){
        System.out.println("list.get("+i+")="+list.get(i));
    }
}
```

### ArrayList 에 있는 데이터를 삭제하자

* `void clear()` : 모든 데이터 삭제
* `E remove(int index)` : 매개 변수에서 지정한 위치에 있는 데이터를 삭제하고 삭제한 데이터를 리턴
* `boolean remove(Object o)` : 매개 변수에 넘어온 객체와 동일한 첫 번째 데이터를 삭제
* boolean removeAll(Collection\<?> c) : 매개변수로 넘어온 컬렉션 객체에 있는 데이터와 동일한 모든 데이터를 삭제

### ArrayList 를 배열로 변경

* `Object[] toArray()` : Object 타입의 배열로 반환&#x20;
  * **`Object` 타입으로 받기 때문에, 형 변환이 필요하다.**&#x20;

```java
ArrayList<String> list = new ArrayList<>(); 
list.add(" apple");
list.add("banana");
list.add("cherry");

Object[] array = list.toArray(); 

for(Object ele : array){
    System.out.println(ele);
}
```

* `<T> T[] toArray(T[] a)` : 특정 타입의 배열로 변환  **(일반적으로 많이 사용하는 방법)**&#x20;
  * **원하는 타입을 지정해 줄 수 있기 떄문에, 형 변환이 필요하지 않다.**
  * **전달된 배열의 크기가 ArrayList 의 크기 보다 작으면 새로운 배열이 생성된다.**&#x20;
  * **제공된 배열의 크기가 적절한 경우에는 기존 배열이 사용된다.**&#x20;

```java
ArrayList<String> list = new ArrayList<>(); 
list.add(" apple");
list.add("banana");
list.add("cherry");

String[] array = list.toArray(new String[0]); 

for(String: array){
    System.out.println(ele);
}
```

### ArrayList 에 있는 데이터를 변경

* `E set(int index, E element)` : 지정한 위치에 해당 객체로 바꿔친다.

### ArrayList 동기화 처리하기&#x20;

* `ArrayList` 를 멀티 쓰레드 환경에서 사용할 수 있도록 `Collections.synchronizedList()` 메서드가 제공된다.
* 아래 코드와 같이 컬렉션 내 많은 객체를 동기화 처리 해준다.&#x20;

```java
/* ArrayList 동기화 처리 */
List<String> l1 = Collections.synchronizedList(new ArrayList<>());

/* LinkedList 동기화 처리 */
List<String> l2 = Collections.synchronizedList(new LinkedList<>());

/* HashSet 동기화 처리 */
Set<String> s = Collections.synchronizedSet(new HashSet<>());

/* HashMap 동기화 처리 */
Map<String> m = Collections.synchronizedMap(new HashMap<>());
```
