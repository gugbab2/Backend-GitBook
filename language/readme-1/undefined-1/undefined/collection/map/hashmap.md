# HashMap

## HashMap 기본

```java
java.lang.Object
    java.util.AbstractMap<K,V>
        java.util.HashMap<K,V>
```

> **Serializable, Cloneable, Map\<E>**
>
> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Map\<E> : 맵의 기본 메소드 지정

* HashMap 은 Hashtable 과 달리 key-value 값에 null 을 허용한다.&#x20;
* 때문에, 더 유연하게 사용할 수는 있지만, 이를 처리하기 위해서 주의가 필요하며, 멀티스레드 환경에서 별도의 동기화가 필요하다.&#x20;

### 생성자

* `HashMap()` : 16개의 공간을 갖는 `HashMap` 객체를 생성한다.
* `HashMap(int initalCapacity)` : 매개 변수만큼의 저장 공간을 갖는, `HashMap` 객체를 생성한다.
* `HashMap(int initalCapacity, float loadFactor)` : 첫 매개 변수의 저장 공간을 갖고, 두 번째 매개변수의 로드팩터를 갖는 `HashMap` 객체를 생성한다.
* `HashMap(Map<? extend K, ? extend V> m)` : 매개 변수로 넘어온 Map 을 구현한 객체에 있는 데이터를 갖는 `HashMap` 객체를 생성한다.

### 데이터 저장 과정&#x20;

#### 1. 해시값 계산&#x20;

* `HashMap` 내부에서 키값에 대해`hashCode()` 메서드를 사용하여 해시값을 계산한다.&#x20;

#### 2. 버킷 할당&#x20;

* 해시값에 따라 요소가 저장될 버킷을 찾는다.&#x20;
* 만약 해당 버킷이 비어있다면 요소를 버킷에 바로 저장한다.&#x20;

#### 3. 중복 확인

* 버킷에 이미 값이 존재한다면 `equals()` 메서드를 사용하여, `true` 를 리턴하면 저장하지 않고, `false` 를 리턴한다면 데이터를 저장한다.&#x20;

#### 4. 해시 충돌 처리&#x20;

* 해시값이 같은 요소가 이미 버킷에 존재하는 경우, 충돌이 발생한다.&#x20;
* `HashSet` 은 체이닝(chaining) 방식을 사용하여, 같은 버킷에 `LinkedList` 를 사용하여 데이터를 저장한다.
* Java 8 부터는 해시 충돌시, 기본적으로는 `LinkedList` 를 사용하지만, 버킷에 저장된 수가 8개 이상이 되면 `red-block tree` 를 사용한다.&#x20;

## HashMap 시간복잡도(해시 충돌 여부가 성능의 Key)&#x20;

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

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많은 경우&#x20;

### 삭제&#x20;

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많은 경우&#x20;
