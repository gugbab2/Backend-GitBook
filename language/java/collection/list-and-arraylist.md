# List & ArrayList

## List 기본

* Collection 인터페이스를 상속한 List 인터페이스는 Collection 인터페이스의 메서드와 별 차이는 없지만, **배열처럼 순서가 있다는 점이 매우 중요한 특징이다.**
* **List 인터페이스를 상속한 Vector, ArrayList 클래스**의 기능과 사용법은 거의 비슷하지만, Vector 는 Thread Safe 하고 ArrayList 는 Thread Safe 하지 못하다...
* **Stack 클래스는 Vector 클래스를 확장해 만들었다.**  Stack 클래스의 가장 큰 특징은, LIFO(Last In First Out) 를 지원하기 위함이다. \
  \-> 스택 메모리에 스택프레임이 들어왔다 나가는 과정을 생각해보자!\
  \-> 이와 같은 데이터 구조를 다뤄야 할 때 Stack 클래스를 사용하자!!
* LinkedList 클래스는 'List' 에도 속하지만, 'Queue' 에도 속한다.

## ArrayList

```java
java.lang.Object
    java.util.AbstracCollection<E>
        java.util.AbstractList<E>
            java.util.ArrayList<E>
```

* Object -> Collection -> ArrayList 순으로 확장한 것을 확인할 수 있다.&#x20;
* ArrayList 는 아래 인터페이스를 구현한다.&#x20;

> **Serializable, Cloneable, Iterable\<E>, Collection\<E>, List\<E>, RandomAccess**
>
> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 clone() 메서드가 제대로 수행될 수 있음을 지정, 복제가 가능한 객체
>
> Iterable\<E> : foreach 문장 사용 가능
>
> Collection\<E>
>
> List\<E>&#x20;
>
> RandomAccess : 목록형 데이터보다 빠르게 접근할 수 있도록 임의로(random) 접근하는 알고리즘이 적용된다는 것을 지정

## ArrayList 생성자

* ArrayList() : 객체를 저장할 공간이 10개인 ArrayList 객체를 만든다.\
  \-> **10 이상의 데이터가 들어가면 크기를 늘이는 작업이 ArrayList 내부에서 자동으로 수행된다.**\
  ****-> 해당 작업은 애플리케이션 성능에 영향을 주는 작업으로, 크기가 예상이 된다면 초기에 지정해주는 것이 좋다.
* ArrayList(Collection\<? extend E> c) : 매개변수로 넘어온 컬렉션 객체가 저장되어 있는 ArrayList를 만든다.&#x20;
* ArrayList(int initialCapacity) : 매개변수만큼의 저장공간을 갖는 ArrayList 를 만든다.

## ArrayList 에 데이터를 담자

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

## ArrayList 에서 데이터를 꺼내자

* Int size() : Collection 을 구현한 객체에 들어가 있는 데이터의 갯수 리턴.\
  \-> 배열.length 는 배열의 저장 공간 개수를 의미하지만, \
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

## ArrayList 에 있는 데이터를 삭제하자

* void clear() : 모든 데이터 삭제
* E remove(int index) : 매개 변수에서 지정한 위치에 있는 데이터를 삭제하고 삭제한 데이터를 리턴
* boolean remove(Object o) : 매개 변수에 넘어온 객체와 동일한 첫 번째 데이터를 삭제
* boolean removeAll(Collection\<?> c) : 매개변수로 넘어온 컬렉션 객체에 있는 데이터와 동일한 모든 데이터를 삭제

## ArrayList 를 배열로 변경

* Object\[] toArray() : ArrayList 객체에 있는 값들을 Object\[] 타입의 배열로 만든다.
* **\<T> T\[] toArray(T\[] a) : ArrayList 객체에 있는 값들을 매개 변수로 넘어온 T 타입의 배열로 만든다.**\
  **-> 매개변수가 없는 toArray() 메서드는 Object 배열로 리턴을 하기에 너무 열려있는 타입의 배열을 좋지 않다.**\
  **-> 때문에 2번째 타입의 메서드를 사용하는 것이 좋다.**

## ArrayList 에 있는 데이터를 변경

* E set(int index, E element) : 지정한 위치에 해당 객체로 바꿔친다.
