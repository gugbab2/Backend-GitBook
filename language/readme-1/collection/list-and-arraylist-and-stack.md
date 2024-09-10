# List

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/List.html](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)\
> [https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html](https://docs.oracle.com/javase/8/docs/api/java/util/ArrayList.html)
>
> [https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

## List 기본

* `Collection` 인터페이스를 상속한 `List` 인터페이스는 `Collection` 인터페이스의 메서드와 별 차이는 없지만, **배열처럼 순서가 있다는 점이 매우 중요한 특징이다.**
  * **여기서 순서란 정렬을 의미하는 것이 아닌, 사용자가 저장하는 순서대로 저장된다 생각하면 된다.**
* **`Set` 과 달리 중복을 허용한다.**&#x20;
* `List` 인터페이스를 상속한 `Vector`, `ArrayList` 클래스의 기능과 사용법은 거의 비슷하지만, `Vector` 는 Thread Safe 하고 `ArrayList` 는 Thread Safe 하지 못하다...
* `Stack` 클래스는 `Vector` 클래스를 확장해 만들었다. `Stack` 클래스의 가장 큰 특징은, LIFO(Last In First Out) 를 지원하기 위함이다.
  * 스택 메모리에 스택프레임이 들어왔다 나가는 과정을 생각해보자!
  * 이와 같은 데이터 구조를 다뤄야 할 때 Stack 클래스를 사용하자!!

## ArrayList 기본

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
  * `RandomAccess` : 목록형 데이터를 보다 빠르게 접근할 수 있도록 최적화를 제공한다는 의미&#x20;

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

## Vector&#x20;

* Vector 는 ArrayList 와 동작 방식이 거의 동일하다.&#x20;
* 하지만 동기화를 기본적으로 한다는 점에서 매우 큰 차이가 있다.

### Vector 의 단점과 동기화 문제점&#x20;

#### 강제 동기화로 인해 느려진 성능&#x20;

* `Vector` 는 기본적으로 `synchronized` 가 걸려있기 때문에, 싱글 쓰레드 환경에서도 Lock 을 걸어버린다.&#x20;
* 이는 `ArrayList` 보다 느릴 수 밖에 없다.&#x20;

#### 완벽하지 않은 동기화&#x20;

* 그리고, 웃기게도 Vector 의 동기화 처리는 완벽한 동기화가 아니다.&#x20;
* 이게 무슨 말이냐 하면 `Vector` 클래스의 각메서드에 대해서는 `synchronized` 처리가 되어있지만, `Vector` 인스턴스 자체에는 동기화 처리가 되어있지 않기 때문에, 각각의 쓰레드가 다른 메서드를 호출하고 있다면, 동기화가 유지되지 않을 수 있다는 것이다.&#x20;
* 아래 예제를 살펴보자.
  * 첫번째 쓰레드에서는 `vec` 객체의 요소를 `add` 하고 `get` 하여 출력하고,
  * 두번째 쓰레드에서는 요소를 `remove` 한다.
  * **결과는 `ArrayIndexOutOfBoundsException` 이 발생하게 된다.**&#x20;
    * 첫번째 쓰레드에서 요소를 추가하고 읽어오려는데,&#x20;
    * 두번째 쓰레드에서 요소를 `remove` 하고 내부 배열을 줄였기 때문에, 존재하지 않은 요소가 들어있던 범위를 벗어난 인덱스를 참조하려고 했기 때문이다.&#x20;
    * **위와 같은 상황이 발생하는 이유는, 메서드 자체 실행으로는 Thread Safe 하지만, Vector 인스턴스 객체 자체에는 Thread Safe 하지 않기 때문이다.**

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Vector<Integer> vec = new Vector<>();

        new Thread(() -> {
            vec.add(1);
            vec.add(2);
            vec.add(3);
            System.out.println(vec.get(0));
            System.out.println(vec.get(1));
            System.out.println(vec.get(2));
        }).start();

        new Thread(() -> {
            vec.remove(0);
            vec.remove(0);
            vec.remove(0);
        }).start();

        // 출력
        new Thread(() -> {
            try {
                Thread.sleep(1000); // 쓰레드가 다 돌때까지 1초 대기

                System.out.println("Vector size : " + vec.size());
            } catch (InterruptedException ignored) {
            }
        }).start();
    }
}
```

### Vector 의 동기화 추가 처리하기&#x20;

* 따라서, `synchronized` 블록을 통해 `Vector` 객체 자체를 동기화 처리 해주어야 한다.&#x20;
* 아래 코드에서는 `vec` 객체에 `synchronized` 블록을 설정해줘 하나의 쓰레드가 `vec` 객체의 소유권을 가진 상태이기 때문에, Lock 이 걸려 대기 상태가 된다.&#x20;
* **결론적으로, `Vector` 클래스도 동기화를 위해서 후처리가 필요하기에, 굳이 `Vector` 클래스를 사용하지 않는다는 것이다.** \
  **-> 심지어 `deprecated` 된 클래스이다.** \
  **->** `synchronizedList` 와 비교해도, `Vector` 가 느리다..&#x20;

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Vector<Integer> vec = new Vector<>();

        new Thread(() -> {
            synchronized(vec){
                vec.add(1);
                vec.add(2);
                vec.add(3);
                System.out.println(vec.get(0));
                System.out.println(vec.get(1));
                System.out.println(vec.get(2));
            }
        }).start();

        new Thread(() -> {
            synchronized(vec){
                vec.remove(0);
                vec.remove(0);
                vec.remove(0);
            }
        }).start();

        // 출력
        new Thread(() -> {
            try {
                Thread.sleep(1000); // 쓰레드가 다 돌때까지 1초 대기

                System.out.println("Vector size : " + vec.size());
            } catch (InterruptedException ignored) {
            }
        }).start();
    }
}
```

## Stack

* `Stack` 클래스는 LIFO(Last In First Out) 기능을 구현하려 할때 필요한 클래스이다.
* **하지만 기능때문에, `Stack` 클래스를 사용하는 것은 추천하지 않는다. 더 좋은 성능의 `ArrayDeque` 가 있기 때문이다.**
* 하지만 `ArrayDeque` 는 Thread Safe 하지 못하기 때문에, Thread Safe 하게 사용하고 싶다면, 약간 성능은 떨어지지만 `Stack` 을 사용하면 된다.\
  **-> `Stack` 은 `Vactor` 클래스를 상속받았다.**
* **`Stack` 클래스는 자바에서 상속을 잘못 받은 케이스이다. 이 클래스가 JDK 1.0 부터 존재했기 때문에, 원래의 취지인 LIFO 를 생각한다면, `Vector` 에 속해서는 안된다. 하지만 자바의 하위 호환성을 위해서 유지하고 있다고 생각하면 된다.**&#x20;

### Stack 의 주요 메서드 목록

* `boolean empty()` : 객체가 비어있는지를 확인한다.
* `E peek()` : 객체의 가장 위에 있는 데이터를 리턴.
* `E pop()` : 객체의 가장 위에 있는 데이터를 지우고, 리턴
* `E push(E item)` : 매개변수로 넘어온 데이터를 가장 위에 저장
* `int search(Object o)` : 매개변수로 넘어온 데이터 위치를 리턴
