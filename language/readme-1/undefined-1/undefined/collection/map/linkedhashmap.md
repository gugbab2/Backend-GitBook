# LinkedHashMap

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashMap.html)

## LinkedHashMap 기본

```java
java.lang.Object
    java.util.AbstractMap<K,V>
        java.util.HashMap<K,V>
            java.util.LinkedHashMap<K,V>
```

> Serializable : 원격으로 객체를 전송, 파일 I/O 가능
>
> Cloneable : Object 클래스의 _**clone() 메서드가 제대로 수행될 수 있음을 지정**_, 복제가 가능한 객체
>
> Map\<E> : 맵의 기본 메소드 지정

* `LinkedHashMap` 은 `HashMap` 과 저장된 순서를 유지한다.&#x20;
* **하지만 순서 유지를 위한 오버헤드가 있기 때문에, 약간의 성능 차이가 발생할 수 있다.** \
  **-> 거의 없다고 봐야한다 ..**&#x20;

## LinkedHashMap 시간복잡도(해시 충돌 여부가 성능의 Key)&#x20;

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

* **최선**: O(1)
  * 해시 충돌이 없는 상태&#x20;
* **최악**: O(n)
  * 해시 충돌이 많은 경우&#x20;

### 삭제&#x20;

* 최선 : O(1)
  * 해시 충돌이 없는 상태
* 최악 : O(n)&#x20;
  * 해시 충돌이 많은 경우
