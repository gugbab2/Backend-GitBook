# Stack

> 참고 링크
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html](https://docs.oracle.com/javase/8/docs/api/java/util/Stack.html)

## Stack

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractList<E>
            java.util.Vector<E>
                java.util.Stack<E>    
```

> `Vector` :  `Vector` 클래스를 상속받아 `Vector` 클래스 기능 사용 가능&#x20;

* `Stack` 클래스는 LIFO(Last In First Out) 기능을 구현하려 할때 필요한 클래스이다.
* **하지만 기능때문에, `Stack` 클래스를 사용하는 것은 추천하지 않는다. 더 좋은 성능의 `ArrayDeque` 가 있기 때문이다.**
* 하지만 `ArrayDeque` 는 Thread Safe 하지 않는데, 이때는 `Collections.synchronizedDeque()` 를 사용하자.&#x20;
* **`Stack` 클래스는 자바에서 상속을 잘못 받은 케이스이다. 이 클래스가 JDK 1.0 부터 존재했기 때문에, 원래의 취지인 LIFO 를 생각한다면, `Vector` 에 속해서는 안된다. 하지만 자바의 하위 호환성을 위해서 유지하고 있다고 생각하면 된다.**&#x20;

### 주요 메서드 목록

* `boolean empty()` : 객체가 비어있는지를 확인한다.
* `E peek()` : 객체의 가장 위에 있는 데이터를 리턴.
* `E pop()` : 객체의 가장 위에 있는 데이터를 지우고, 리턴
* `E push(E item)` : 매개변수로 넘어온 데이터를 가장 위에 저장
* `int search(Object o)` : 매개변수로 넘어온 데이터 위치를 리턴
