# HashSet

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html](https://docs.oracle.com/javase/8/docs/api/java/util/HashSet.html)

## HashSet&#x20;

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractSet<E>
            java.util.HashSet<E>
```

> `Set<E>` : 셋 데이터를 처리하는 것과 관련된 메서드를 지정
>
> `Cloneable` : `Object` 클래스의 `clone()` 메서드가 제대로 수행될 수 있음을 지정
>
> `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능

* **`Set` 을 확장한 클래스이며, 무엇보다 중복을 허용하지 않기 때문에, 데이터가 같은지를 확인하는 것이 핵심이다.**
* **때문에, `equals()` 메서드와 `hashCode()` 메서드는 값의 중복을 확인하는데, 매우 중요한 역할을 한다.**
  * `equals()` : 값이 같은지
  * `hashCode()` : 레퍼런스가 같은지&#x20;

### 생성자

* `HashSet()` : 데이터를 저장할 수 있는 16개의 공간과, 0.75의 로드 팩터를 갖는 객체를 생성한다.
* `HashSet(Collection<? extends E> c)` : 매개 변수로 받은 컬렉션의 객체의 데이터를 HashSet에 담는다.
* `HashSet(int initialCapacity)` : 매개변수로 받은 개수만큼 데이터 저장 공간과 0.75의 로드 팩터를 갖는 객체를 생성한다.
* `HashSet(int initialCapacity, float loadFactor)` : 첫번째 매개변수로 받은 개수만큼 데이터 저장 공간과, 두번째 매개변수로 받은 만큼의 로드팩터를 갖는 객체를 생성

> #### **로드팩터란?**
>
> * **로드팩터 = 데이터의 개수(Data size) / 저장 공간(Capacity)**
> * 만약 데이터의 개수가 증가해 로드팩터의 값이 본래 로드팩터 값(초기 지정)보다 커지면, 저장공간은 증가되고 해시 재정리 작업(rehashing)을 해야만 한다.
> * 데이터가 해시 재정리 작업에 들어가면, 내부에 갖고 있는 자료 구조를 다시 생성하는 단계를 거쳐야 하기에 성능에 영향을 미친다.
> * **로드 팩터가 낮을 때는 빠른 성능(검색/삽입/삭제)과 낮은 해시충돌 발생, 하지만 메모리 낭비가 있을 수 있다.**&#x20;
> * **로드 팩터가 높을 때는 메모리 효율성을 좋지만, 해시 충돌이 많이 발생해 성능 저하(검색/삽입/삭제) 가능성이 높다.**&#x20;

### 데이터 저장 과정&#x20;

#### 1. HashMap 호출&#x20;

* `HashSet` 에 데이터가 저장될 때, 내부적으로 `HashMap.put(E e, Boolean.TRUE)` 의 형태로 `HashMap` 을 호출한다. &#x20;

#### 2. 해시값 계산&#x20;

* HashMap 내부에서 키값에 대해`hashCode()` 메서드를 사용하여 해시값을 계산한다.&#x20;

#### 3. 버킷 할당&#x20;

* 해시값에 따라 요소가 저장될 버킷을 찾는다.&#x20;
* 만약 해당 버킷이 비어있다면 요소를 버킷에 바로 저장한다.&#x20;

#### 4. 중복 확인

* 버킷에 이미 값이 존재한다면 `equals()` 메서드를 사용하여, `true` 를 리턴하면 저장하지 않고, `false` 를 리턴한다면 데이터를 저장한다.&#x20;

#### 5. 해시 충돌 처리&#x20;

* 해시값이 같은 요소가 이미 버킷에 존재하는 경우, 충돌이 발생한다.&#x20;
* `HashSet` 은 체이닝(chaining) 방식을 사용하여, 같은 버킷에 `LinkedList` 를 사용하여 데이터를 저장한다.
* Java 8 부터는 해시 충돌시, 기본적으로는 `LinkedList` 를 사용하지만, 버킷에 저장된 수가 8개 이상이 되면 `red-block tree` 를 사용한다.&#x20;

### 주요 메서드 목록

* `boolean add(E e)` : 데이터를 추가한다.
* `void clear()` : 모든 데이터를 삭제한다.
* `Object clone()` : HashSet 객체를 복사한다. 하지만 담겨 있는 데이터들은 복사하지 않는다.
* `boolean contains(Object o)` : 지정한 객체가 존재하는지를 확인
* `boolean isEmpty()` : 데이터가 있는지 확인한다.
* `Iterator<E> iterator()` : 데이터를 꺼내기 위한 Iterator 객체를 리턴한다 .
* `boolean remove(Object o)` : 매개 변수로 넘어온 객체를 삭제한다.
* `int size()` : 데이터의 개수를 리턴한다.

### 예제

* HashSet 은 중복이 제거되기는 하지만, 순서는 신경쓰지 않기 때문에, 아래 예제에서 순서가 바뀌어 나올 수도 있다.

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

* 추가적으로 Iterator 를 통해서도 결과를 얻을 수 있다.

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

## HashSet 시간복잡도(해시 충돌 여부가 성능의 Key)&#x20;

### 생성&#x20;

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많거나, rehasing 이 일어나는 경우&#x20;

### 읽기

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많은 경우&#x20;

### 수정

* 중복을 허용하지 않기 때문에, 수정은 없다.&#x20;

### 삭제&#x20;

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많은 경우&#x20;
