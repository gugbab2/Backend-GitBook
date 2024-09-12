# Hashtable

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html](https://docs.oracle.com/javase/8/docs/api/java/util/Hashtable.html)

## Hashtable

```java
java.lang.Object
    java.util.Dictionary<K,V>
        java.util.Hashtable<K,V>
```

> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Map\<E> : 맵의 기본 메소드 지정

* key-value pair 를 저장하는 자료구조로, 해시 함수를 사용하여 데이터를 저장하는 방식이다.&#x20;
* Hashtable 는 HashMap 과 동작 방식이 거의 동일하다. 하지만 몇가지 차이점이 있는데, 그 차이는 다음과 같다.&#x20;
  * **메서드에서 동기화를 기본적으로 한다.**&#x20;
  * **key-value 값으로 null 을 허용하지 않는다.**&#x20;
* 동기화를 기본적으로 한다는 부분은매우 큰 차이이다.

### Hashtable 의 단점과 동기화 문제점 (Vector 와 비슷한 문제점이다)&#x20;

#### 강제 동기화로 인해 느려진 성능&#x20;

* Vector 와 마찬가지로 메서드에 기본적으로 synchronized 가 걸려 있다.&#x20;

#### 상대적으로 많은 메모리 사용&#x20;

* 체이닝을 위한 연결리스트와 리사이징 과정으로 인해 메모리 사용량이 상대적으로 많은 수 있다.&#x20;
* 대량의 데이터가 해시 충돌로 인해 리스트가 길어지면 메모리와 성능에 영향을 끼질 수 있다.&#x20;
* HashMap 에도 동일한 단점이 적용된다.&#x20;

#### 완벽하지 않은 동기화

* Vector 와 마찬가지로 메서드에만 동기화가 걸려있기 때문에,&#x20;
* 클래스 전체로 동기화를 걸어야 할 수 있다.&#x20;

#### 레거시 클래스&#x20;

* Hashtable 은 Java 1.0 에서 제공된 클래스로, 레거시 클래스이다.&#x20;
* Java 1.2 부터 동일한 역할을 하는 HashMap 클래스가 도입되었고, 더 나은 성능과 기능을 제공한다.&#x20;

## Hashtable 시간복잡도(해시 충돌 여부가 성능의 Key)&#x20;

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
