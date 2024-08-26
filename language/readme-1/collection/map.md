# Map

## Map 기본

* 모든 데이터는 키와 값 형태로 존재한다.
* 키 없이 값만 저장될 수는 없다..
* 값 없이 키만 저장할 수도 없다..
* **키는 해당 Map 에서 고유해야만 한다.**\
  \-> 값은 여러개의 키를 가질 수 있다.

## Map 인터페이스 선언된 메서드

* V put(K key, V value) : 첫번째 매개변수인 키를 갖는, 두번째 매개변수인 값을 갖는 데이터를 저장한다.
* void putAll(Map\<? extends K, ? extends V> m) : 매개변수로 넘어온 Map 의 모든 데이터를 저장한다.
* V get(Object key) : 매개변수로 넘어온 키에 해당하는 값을 넘겨준다.
* V remove(Object key) : 매개변수로 넘어온 키에 해당하는 값을 넘겨주며, 해당 키와 같은 Map 에서 삭제한다.
* Set\<K> keySet() : 키 목록을 Set 타입으로 리턴한다.
* Collection\<V> values() : 값의 목록을 Collection 타입으로 리턴한다.
* Set\<Map.Entry\<K,V>> entrySet() : Map 안에 Entry 라는 타입의 Set 을 리턴한다.\
  \-> Entry : 단 하나의 키, 값을 저장한다
* int size() : Map 의 크기를 리턴한다.
* void clear() : Map 의 내용을 지운다.

## Map 과 Hashtable

* Map 은 Collection View 를 사용하지만 Hashtable 은 Enumeration 객체를 통해서 데이터를 처리한다.
* Map 은 키, 값, 키-값 쌍으로 데이터를 순환하여 처리할 수 있지만, Hashtable 은 이 중에서 키-값 쌍으로 데이터를 순환하여 처리할 수 없다..
* Map 은 이터레이션을 처리하는 도중에 데이터를 삭제하는 안전한 방법을 제공하지만, Hashtable 은 그러한 기능을 제공하지 않는다..\
  \=> **이러한 특징은, Hashtable 이 Map 보다 일찍 만들어졌기 때문이다.**
* Hashtable 은 Thread Safe 하지만, Map 인터페이스를 구현한 클래스는 Thread Safe 하지 않기 때문에, 다음과 같이 선언해야 한다.

```java
Map m = Collections.synchronizedMap(new HashMap(...));

// 또는, 이름에 Concurrent 가 포함되어 있어야만 쓰레드에 안전하게 사용할 수 있다. 
// 자바 1.5에 추가된 ConcurrentHashMap, CopyOnWriteArrayList 등이 있으며 
// java.util.concurrent 패키지 소속이다.
```

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

## HashMap 생성자

* HashMap() : 16개의 공간을 갖는 HashMap 객체를 생성한다.
* HashMap(int initalCapacity) : 매개 변수만큼의 저장 공간을 갖는, HashMap 객체를 생성한다.
* HashMap(int initalCapacity, float loadFactor) : 첫 매개 변수의 저장 공간을 갖고, 두 번째 매개변수의 로드팩터를 갖는 HashMap 객체를 생성한다.
*   HashMap(Map\<? extend K, ? extend V> m) : 매개 변수로 넘어온 Map 을 구현한 객체에 있는 데이터를 갖는 HashMap 객체를 생성한다.\
    \-> HashMap 의 키는 기본자료형과 참조자료형 모두가 될 수 있다.\
    \-> _만약 참조자료형으로 키를 설정한다면 equals(), hashcode() 메서드를 오버라이딩 해주어야 한다._\
    \-> **그 이유는, HashMap에 put() 메서드를 사용해 객체가 들어가면 hashcode() 메서드 결과 값에 따른, 버켓(bucket) 이라는 목록(LinkedList) 형태의 바구니가 만들어진다.**\
    **-> 만약 서로 다른 키가 저장되었는데, hashcode() 메서드의 결과값이 동일하다면 이 버켓의 여러개의 값이 들어갈 수 있다.**\
    **-> equals() 메서드를 통해 동일한 값이 있다면, 값을 저장하지 않게 된다. ( 만약 equals() 메서드를 오버라이딩 하지 않는다면, Object.equals() 메서드를 호출 객체의 주소값만을 비교하기 때문에 항상 데이터를 저장하게 된다)**\
    **-> 따라서, get() 메서드가 호출되면, hashCode() 결과를 확인하고, equals() 메서드를 통해 버켓에 들어간 데이터를 가져온다.**

    <figure><img src="../../../.gitbook/assets/image (40).png" alt=""><figcaption></figcaption></figure>

## TreeMap

* TreeMap 객체는 저장하면서 키를 정렬한다. (숫자 > 알파벳 대문자 > 알파벳 소문자 > 한글)
* 이 순서는 String 같은 문자열이 저장되는 순서를 말하는 것이며, 객체가 저장되거나, 숫자가 저장될 때는 순서가 달라진다.
* 하지만 매우 많은 데이터를 저장할 때는, HashMap 보다 느리다 ... 왜냐면 키가 정렬되기 때문이다!\
  \-> 100, 1000 건 정도의 데이터의 키값을 정렬하고 싶다면 TreeSet 을 사용한는 것을 권장.
* TreeMap 이 키를 정렬하는 것은 SortedMap 이라는 인터페이스를 구현했기 때문이다.\
  \-> SortedMap 을 구현한 클래스들은 모두 키가 정렬되어 있어야만 한다.

## Map 을 구현한 Properties 클래스

* System 클래스에 Properties 라는 클래스가 있다.
* Properties 클래스는 HashTable 클래스를 확장하였고, Map 인터페이스에서 제공하는 모든 메서드를 사용할 수 있다. -> **기본적으로 자바에서는, 시스템의 속성을 이 클래스를 사용하여 제공한다!**

```java
public class PropertiesSample {
    public static void main(String[] args){
        PropertiesSample sample = new PropertiesSample();
        sample.checkProperties();
    }

    private void checkProperties() {
        Properties prop = System.getProperties();
        Set<Object> keySet = prop.keySet();
        for(Object tempObject : keySet){
            System.out.println(tempObject + "=" + prop.get(tempObject));
        }
    }
}
```
