# Set & Queue

## Set 기본

* List 는 순서가 중요한 데이터를 담을 때 사용하고,&#x20;
* **Set 은 순서와 상관없이, 중복되는 것을 방지하고 원하는 값이 포함되어 있는지를 확인하는 것이 주 용도다.**
* Set 을 상속받은 클래스는 HashSet, TreeSet, LinkedHashSet 이 포함되어 있다. \
  \-> _**정렬 때문에 세가지 클래스의 성능 차이가 발생한다.**_&#x20;
  * **HashSet** : 순서가 전혀 필요 없는 데이터를 해시테이블에 저장한다. Set 중 가장 성능이 좋다.
  * **TreeSet** : 저장된 데이터의 값에 따라서 정렬되는 셋이다. red-block 이라는 트리 타입으로 값이 저장되며 HashSet 보다 약간 성능이 느리다.\
    \-> red-block : 각 노드를 빨강, 검정색으로 구분하여, 데이터를 쉽게 찾을 수 있는 이진트리를 말한다.&#x20;
  * **LinkedHashSet** : 연결된 목록 타입으로 구현된 해시 테이블에 데이터를 저장한다. 저장된 순서에 따라서 값이 정렬된다. 대신 성능이 이 셋중에 가장 나쁘다.&#x20;

## HashSet 기본

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractSet<E>
            java.util.HashSet<E>
```

* Set 클래스에는 equals(), hashCode(), removeAll() 메서드만 구현되어 있다.&#x20;
* Set 을 확장한 클래스이며, 무엇보다 중복을 허용하지 않기 때문에, 데이터가 같은지를 확인하는 것이 핵심이다.\
  \-> equals() 메서드와 hashCode() 메서드는 값의 중복을 확인하는데, 매우 중요한 역할을 한다.&#x20;

> **Serializable, Cloneable, Iterable\<E>, Collection\<E>, Set\<E>**
>
> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Iterable\<E> : foreach 문장 사용 가능
>
> Collection\<E>
>
> Set\<E>&#x20;

## HashSet 생성자

* HashSet() : 데이터를 저장할 수 있는 16개의 공간과, 0.75의 로드 팩터를 갖는 객체를 생성한다.
* HashSet(Collection\<? extends E> c) : 매개 변수로 받은 컬렉션의 객체의 데이터를 HashSet에 담는다.
* HashSet(int initialCapacity) : 매개변수로 받은 개수만큼 데이터 저장 공간과 0.75의 로드 팩터를 갖는 객체를 생성한다.
* HashSet(int initialCapacity, float loadFactor) : 첫번째 매개변수로 받은 개수만큼 데이터 저장 공간과, 두번째 매개변수로 받은 만큼의 로드팩터를 갖는 객체를 생성

> ### **로드팩터란?**
>
> * _**로드팩터 = 데이터의 개수 / 저장 공간**_
> * 만약 데이터의 개수가 증가해 로드팩터 보다 커지면, 저장공간은 증가되고 해시 재정리 작업을 해야만 한다.&#x20;
> * **데이터가 해시 재정리 작업에 들어가면, 내부에 갖고 있는 자료 구조를 다시 생성하는 단계를 거쳐야 하기에 성능에 영향을 미친다.**
> * 로드팩터의 값이 클수록 공간은 증가하지만 데이터를 찾는 시간은 증가한다.&#x20;
> * 따라서 초기 공간 개수와 로드팩터는 데이터의 크기를 고려하여 산정하는 것이 좋다

## HashSet 주요 메서드 목록

* boolean add(E e) : 데이터를 추가한다.
* void clear() : 모든 데이터를 삭제한다.
* Object clone() : HashSet 객체를 복사한다. 하지만 담겨 있는 데이터들은 복사하지 않는다.&#x20;
* boolean contains(Object o) : 지정한 객체가 존재하는지를 확인
* boolean isEmpty() : 데이터가 있는지 확인한다.&#x20;
* Iterator\<E> iterator() : 데이터를 꺼내기 위한 Iterator 객체를 리턴한다.
* boolean remove(Object o) : 매개 변수로 넘어온 객체를 삭제한다.&#x20;
* int size() : 데이터의 개수를 리턴한다.&#x20;

## HashSet 예제

* HashSet 은 중복이 제거되기는 하지만, 순서는 신경쓰지 않기 때문에, 아래 예제에서 순서가 바뀌어 나올 수도 있다.&#x20;

```java
public class SetSample {

    public static void main (String[] args){
        SetSample setSample = new SetSample();
        String[] test = new String[]{
          "a","a","b","b","c","c","d","d","e",
                "f","f","g","g","h","h","i","i","j",
                "k","k","l","l","m","m","n","n","n",
        };
        System.out.println(setSample.getTestKind(test));
    }
    
    public int getTestKind(String[] s){
        HashSet<String> hashSet = new HashSet<>();
        for(String string : s){
            hashSet.add(string);
        }

        return hashSet.size();
    }
}
```

* 추가적으로 Iterator 를 통해서도 결과를 얻을 수 있다.&#x20;

```java
public int getTestKind(String[] s){
    HashSet<String> hashSet = new HashSet<>();
    for(String string : s){
        hashSet.add(string);
    }

    Iterator<String> iterator = hashSet.iterator();
    while(iterator.hasNext()){
        System.out.println(iterator.next());
    }
    return hashSet.size();
}
```

## Queue 기본

*
