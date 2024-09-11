# Map

## Map

* 모든 데이터는 키와 값 형태로 존재한다.
* 키 없이 값만 저장될 수는 없다.
* 값 없이 키만 저장할 수도 없다.
* 키는 해당 Map 에서 고유해야만 한다.
* 값은 여러개의 키를 가질 수 있다.

## Map 인터페이스 선언된 메서드

* `V put(K key, V value)` : 첫번째 매개변수인 키를 갖는, 두번째 매개변수인 값을 갖는 데이터를 저장한다.
* `void putAll(Map<? extends K, ? extends V> m)` : 매개변수로 넘어온 Map 의 모든 데이터를 저장한다.
* `V get(Object key)` : 매개변수로 넘어온 키에 해당하는 값을 넘겨준다.
* `V remove(Object key)` : 매개변수로 넘어온 키에 해당하는 값을 넘겨주며, 해당 키와 같은 `Map` 에서 삭제한다.
* `Set<K> keySet()` : 키 목록을 Set 타입으로 리턴한다.
* `Collection<V> values()` : 값의 목록을 `Collection` 타입으로 리턴한다.
* `Set<Map.Entry<K,V>> entrySet()` : `Map` 안에 `Entry` 라는 타입의 `Set` 을 리턴한다.\
  \-> `Entry` : 단 하나의 키, 값을 저장한다
* `int size()` : `Map` 의 크기를 리턴한다.
* `void clear()` : `Map` 의 내용을 지운다.

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
