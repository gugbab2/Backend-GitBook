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
