# List, ArrayList, Vector, Stack

## List 기본

* Collection 인터페이스를 상속한 List 인터페이스는 Collection 인터페이스의 메서드와 별 차이는 없지만,\
  _**배열처럼 순서가 있다는 점**_**이 매우 중요한 특징이다.**\
  **-> 여기서 순서란 정렬을 의미하는 것이 아닌, 사용자가 저장하는 순서대로 저장된다 생각하면 된다.**
* **List 인터페이스를 상속한 Vector, ArrayList 클래스**의 기능과 사용법은 거의 비슷하지만, _**Vector 는 Thread Safe 하고 ArrayList 는 Thread Safe 하지 못하다...**_
* **Stack 클래스는 Vector 클래스를 확장해 만들었다.** Stack 클래스의 가장 큰 특징은, LIFO(Last In First Out) 를 지원하기 위함이다.\
  \-> 스택 메모리에 스택프레임이 들어왔다 나가는 과정을 생각해보자!\
  \-> 이와 같은 데이터 구조를 다뤄야 할 때 Stack 클래스를 사용하자!!\
  \-> 만약, 예약서비스를 운영하는데 stack 자료구조를 사용하면 사용자들의 불만을 산다 ..
* LinkedList 클래스는 'List' 에도 속하지만, 'Queue' 에도 속한다.\
  **-> 순차적으로 데이터를 넣고 뺀다면, ArrayList, Vactor 가 좋지만, 중간에 데이터가 지속적으로 변경된다면 LinkedList 가 유리하다.**\
  **-> ArrayList, Vactor 는 중간에 데이터가 추가되거나 삭제되면 이후의 값들이 모두 위치를 변경시켜야 하기 때문에, 부하를 과도할 경우 부하를 일으킨다.**

## ArrayList 기본

```java
java.lang.Object
    java.util.AbstracCollection<E>
        java.util.AbstractList<E>
            java.util.ArrayList<E>
```

* Object -> Collection -> List -> ArrayList 순으로 확장한 것을 확인할 수 있다.
* ArrayList 는 아래 인터페이스를 구현한다.

> **Serializable, Cloneable, Iterable\<E>, Collection\<E>, List\<E>, RandomAccess**
>
> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Iterable\<E> : iterable 인터페이스를 사용해 데이터를 순차적으로 가져올 수 있다.\
> \-> foreach 문 사용 가능.
>
> Collection\<E> : 여러개의 데이터를 한 객체에 담아 처리할 메소드 지정
>
> List\<E> : 목록형 데이터를 처리하는 것과 관련된 메소드 지정
>
> **RandomAccess : 목록형 데이터를 보다 빠르게 접근할 수 있도록 임의로(random) 접근하는 알고리즘이 적용된다는 것을 지정**

### ArrayList 생성자

* ArrayList() : 객체를 저장할 공간이 10개인 ArrayList 객체를 만든다.\
  \-> **10 이상의 데이터가 들어가면 크기를 늘이는 작업이 ArrayList 내부에서 자동으로 수행된다.**\
  **(객체가 생성될 때 배열이 생성되는 것이 아닌, 첫 번째 요소가 추가될 때 크기가 10인 배열이 생성된다. 또한 크기가 다 차면 1.5배씩 배열의 크기는 증가한다)**\
  \-> 해당 작업은 애플리케이션 성능에 영향을 주는 작업으로, 크기가 예상이 된다면 초기에 지정해주는 것이 좋다.
* ArrayList(Collection\<? extend E> c) : 매개변수로 넘어온 컬렉션 객체가 저장되어 있는 ArrayList를 만든다.\
  \-> Collection 을 확장하는 클래스는 List, Set, ... 많다!
* ArrayList(int initialCapacity) : 매개변수만큼의 저장공간을 갖는 ArrayList 를 만든다.

### ArrayList 에 데이터를 담자

* boolean add(E e) : 매개변수로 들어온 데이터를 가장 끝에 담는다.
* void add(int index, E e) : 매개변수로 들어온 데이터를 지정된 index 위치에 담는다.\
  \-> **해당 위치에 데이터를 추가하게 되면, 기존 데이터의 위치가 다 밀려난다.**
* boolean addAll(Collection\<? extends E> c) : 매개변수로 넘어온 컬렉션 데이터를 가장 끝에 담는다.\
  \-> **Collection 관련 객체를 복사할 일이 있을 때에는, 생성자 혹은 addAll() 메서드를 사용하는 것이 좋다.**
* boolean addAll(int index, Collection\<? extend E> c) : 매개변수로 넘어온 컬렉션 데이터를 index 에 지정된 위치부터 담는다.

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
}
```

### ArrayList 에서 데이터를 꺼내자

* Int size() : Collection 을 구현한 객체에 들어가 있는 데이터의 갯수 리턴.\
  \-> 배열.length 는 배열의 저장 공간 개수를 의미하지만,\
  \-> size() 메서드는 객체 안에 들어가 있는 데이터의 갯수를 의미한다.
* E get(int index) : 매개변수에 지정한 위치에 있는 데이터 리턴.
* Int indexOf(Object o) : 매개변수로 지정한 객체와 동일한 데이터의 위치를 리턴.
* int lastInedexOf(Object o) : 매개변수로 지정한 객체와 동일한 마지막 데이터의 위치를 리턴.

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

* void clear() : 모든 데이터 삭제
* E remove(int index) : 매개 변수에서 지정한 위치에 있는 데이터를 삭제하고 삭제한 데이터를 리턴
* boolean remove(Object o) : 매개 변수에 넘어온 객체와 동일한 첫 번째 데이터를 삭제
* boolean removeAll(Collection\<?> c) : 매개변수로 넘어온 컬렉션 객체에 있는 데이터와 동일한 모든 데이터를 삭제

### ArrayList 를 배열로 변경

* Object\[] toArray() : ArrayList 객체에 있는 값들을 Object\[] 타입의 배열로 만든다.
* **\<T> T\[] toArray(T\[] a) : ArrayList 객체에 있는 값들을 매개 변수로 넘어온 T 타입의 배열로 만든다.**\
  **-> 매개변수가 없는 toArray() 메서드는 Object 배열로 리턴을 하기에 너무 열려있는 타입의 배열을 좋지 않다.**\
  **(매개변수로 넘어온 T 타입 배열로 타입을 한정 지을 수 있다)** \
  **-> 때문에 2번째 타입의 메서드를 사용하는 것이 좋다.**

```java
public static void main(String[] args){
    ListSample sample = new ListSample();
    sample.checkArrayList6();
}

public void checkArrayList6(){
    ArrayList<String> list = new ArrayList<>();    //초기의 크기는 10으로
    list.add("A");
    list.add("B");
    list.add("C");
    String[] tempArray = new String[0];
    String[] strList = list.toArray(tempArray);
    for(String temp : strList){
        System.out.println("temp = " + temp);
    }
}
```

### ArrayList 에 있는 데이터를 변경

* E set(int index, E element) : 지정한 위치에 해당 객체로 바꿔친다.

## Vector&#x20;

* Vector 는 ArrayList 와 사용법이나, 저장되는 방식이 비슷하다.
* 하지만 몇가지 부분에서 차이가 있는데, 그 차이는 다음과 같다.&#x20;

### ArrayList vs Vector

1. **Thread Safety (스레드 안전성)**:
   * **`ArrayList`**: 스레드 안전(thread-safe)이 **아닙니다**. 멀티스레드 환경에서 `ArrayList`를 사용할 경우, 동기화를 수동으로 처리해야 합니다. 동기화 없이 여러 스레드가 동시에 `ArrayList`를 수정하면, 데이터 무결성이 깨질 수 있습니다.
   * **`Vector`**: 기본적으로 스레드 안전(thread-safe)합니다. `Vector`의 모든 메서드는 `synchronized`로 구현되어 있어, 여러 스레드가 동시에 접근하더라도 데이터 무결성이 유지됩니다.
2. **성능**:
   * **`ArrayList`**: 스레드 안전성을 보장하지 않기 때문에, `Vector`에 비해 성능이 더 좋습니다. 특히, 단일 스레드 환경에서 `ArrayList`는 더 빠른 성능을 제공합니다.
   * **`Vector`**: 동기화를 위한 추가적인 비용이 발생하므로, 멀티스레드 환경이 아니더라도 성능이 저하될 수 있습니다.
3. **동적 확장**:
   * **`ArrayList`**: 내부 배열이 가득 차면, 크기가 **1.5배**로 늘어납니다.
   * **`Vector`**: 내부 배열이 가득 차면, 크기가 **2배**로 늘어납니다. 그러나 이 방식은 `ArrayList`에 비해 메모리 사용 측면에서 비효율적일 수 있습니다.
4. **사용 권장 여부**:
   * **`ArrayList`**: Java 1.2 이후 `ArrayList`가 `Vector`를 대체하게 되었고, 단일 스레드 환경에서는 `ArrayList`가 더 일반적으로 사용됩니다.
   * **`Vector`**: 현재는 거의 사용되지 않으며, 멀티스레드 환경에서는 `Collections.synchronizedList(new ArrayList<>())`를 사용하거나, 더 나아가 `CopyOnWriteArrayList`와 같은 동시성 컬렉션을 사용하는 것이 권장됩니다.

## Stack

* Stack 클래스는 LIFO(Last In First Out) 기능을 구현하려 할때 필요한 클래스이다.
* 하지만 기능때문에, Stack 클래스를 사용하는 것은 추천하지 않는다. 더 좋은 성능의 _**ArrayDeque**_ 가 있기 때문이다.
* 하지만 ArrayDeque 는 Thread Safe 하지 못하기 때문에, Thread Safe 하게 사용하고 싶다면, 약간 성능은 떨어지지만 Stack 을 사용하면 된다.\
  \-> Stack 은 Vactor 클래스를 상속받았다.

### Stack 의 주요 메서드 목록

* boolean empty() : 객체가 비어있는지를 확인한다.
* E peek() : 객체의 가장 위에 있는 데이터를 리턴.
* E pop() : 객체의 가장 위에 있는 데이터를 지우고, 리턴
* E push(E item) : 매개변수로 넘어온 데이터를 가장 위에 저장
* int search(Object o) : 매개변수로 넘어온 데이터 위치를 리턴
