# LinkedHashSet

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html](https://docs.oracle.com/javase/8/docs/api/java/util/LinkedHashSet.html)

## LinkedHashSet

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractSet<E>
            java.util.HashSet<E>
                java.util.LinkedHashSet<E>
```

> `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능
>
> `Cloneable` : `Object` 클래스의 `clone()` 메서드가 제대로 수행될 수 있음을 지정
>
> `Iterable<E>` : forEech 구문 사용 가능
>
> `Collection<E>` : 여러개의 데이터를 한 데이터에 담아 처리할 메서드를 지정
>
> `Set<E>` : 셋 데이터를 처리하는 것과 관련된 메서드를 지정

* **`LinkedHashSet` 은 기본적으로 데이터 저장 시 해시 충돌이 발생하면 `LinkedList` 로 충돌된 요소를 저장한다.**&#x20;
* **`LinkedList` 는 연결 리스트로 구현되어 있으며, 요소의 삽입 순서를 유지한다.** \
  **-> `HashSet` 도 해시 충돌 시 연결 리스트를 사용해 데이터를 저장하지만, 요소의 삽입 순서를 유지하지 않는다.**&#x20;

## LinkedHashSet 시간복잡도(해시 충돌 여부가 성능의 Key)&#x20;

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
  * 해시 충돌이 많은 경우
