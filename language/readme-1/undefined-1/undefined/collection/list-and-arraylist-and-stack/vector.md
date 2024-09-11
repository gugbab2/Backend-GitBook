# Vector

> 참고 링크
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html](https://docs.oracle.com/javase/8/docs/api/java/util/Vector.html)
>
> [https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

## Vector&#x20;

```java
java.lang.Object
    java.util.AbstractCollection<E>
        java.util.AbstractList<E>
            java.util.Vector<E>
```

> `Serializable` : 원격으로 객체를 전송, 파일 I/O 가능
>
> `Cloneable` : `Object` 클래스의 `clone()` 메서드가 제대로 수행될 수 있음을 지정
>
> `Iterable<E>` : forEech 구문 사용 가능 **(JDK 1.2 이전까지는 Enumeration 을 사용했다)**&#x20;
>
> `Collection<E>` : 여러개의 데이터를 한 객체에 담아 처리할 메소드 지정
>
> `List<E>` : 순서가 보장되며, 중복을 허용하는 목록형 데이터 집합을 의미&#x20;
>
> `RandomAccess` : 인덱스를 사용한 요소 접근(`get(index)` 또는 `set(index, value)`)이 상수 시간(O(1)) 안에 이루어진다는 것을 나타냅니다.

* Vector 는 ArrayList 와 동작 방식이 거의 동일하다.&#x20;
* 하지만 동기화를 기본적으로 한다는 점에서 매우 큰 차이가 있다.

### Vector 의 단점과 동기화 문제점&#x20;

#### 강제 동기화로 인해 느려진 성능&#x20;

* `Vector` 는 기본적으로 `synchronized` 가 걸려있기 때문에, 싱글 쓰레드 환경에서도 Lock 을 걸어버린다.&#x20;
* 이는 `ArrayList` 보다 느릴 수 밖에 없다.&#x20;

#### 완벽하지 않은 동기화&#x20;

* 그리고, 웃기게도 Vector 의 동기화 처리는 완벽한 동기화가 아니다.&#x20;
* 이게 무슨 말이냐 하면 `Vector` 클래스의 각메서드에 대해서는 `synchronized` 처리가 되어있지만, `Vector` 인스턴스 자체에는 동기화 처리가 되어있지 않기 때문에, 각각의 쓰레드가 다른 메서드를 호출하고 있다면, 동기화가 유지되지 않을 수 있다는 것이다.&#x20;
* 아래 예제를 살펴보자.
  * 첫번째 쓰레드에서는 `vec` 객체의 요소를 `add` 하고 `get` 하여 출력하고,
  * 두번째 쓰레드에서는 요소를 `remove` 한다.
  * **결과는 `ArrayIndexOutOfBoundsException` 이 발생하게 된다.**&#x20;
    * 첫번째 쓰레드에서 요소를 추가하고 읽어오려는데,&#x20;
    * 두번째 쓰레드에서 요소를 `remove` 하고 내부 배열을 줄였기 때문에, 존재하지 않은 요소가 들어있던 범위를 벗어난 인덱스를 참조하려고 했기 때문이다.&#x20;
    * **위와 같은 상황이 발생하는 이유는, 메서드 자체 실행으로는 Thread Safe 하지만, Vector 인스턴스 객체 자체에는 Thread Safe 하지 않기 때문이다.**

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Vector<Integer> vec = new Vector<>();

        new Thread(() -> {
            vec.add(1);
            vec.add(2);
            vec.add(3);
            System.out.println(vec.get(0));
            System.out.println(vec.get(1));
            System.out.println(vec.get(2));
        }).start();

        new Thread(() -> {
            vec.remove(0);
            vec.remove(0);
            vec.remove(0);
        }).start();

        // 출력
        new Thread(() -> {
            try {
                Thread.sleep(1000); // 쓰레드가 다 돌때까지 1초 대기

                System.out.println("Vector size : " + vec.size());
            } catch (InterruptedException ignored) {
            }
        }).start();
    }
}
```

### Vector 의 동기화 추가 처리하기&#x20;

* 따라서, `synchronized` 블록을 통해 `Vector` 객체 자체를 동기화 처리 해주어야 한다.&#x20;
* 아래 코드에서는 `vec` 객체에 `synchronized` 블록을 설정해줘 하나의 쓰레드가 `vec` 객체의 소유권을 가진 상태이기 때문에, Lock 이 걸려 대기 상태가 된다.&#x20;
* **결론적으로, `Vector` 클래스도 동기화를 위해서 후처리가 필요하기에, 굳이 `Vector` 클래스를 사용하지 않는다는 것이다.** \
  **->** `synchronizedList` 와 비교해도, `Vector` 가 느리다..&#x20;

```java
import java.util.*;

public class Main {
    public static void main(String[] args) {
        Vector<Integer> vec = new Vector<>();

        new Thread(() -> {
            synchronized(vec){
                vec.add(1);
                vec.add(2);
                vec.add(3);
                System.out.println(vec.get(0));
                System.out.println(vec.get(1));
                System.out.println(vec.get(2));
            }
        }).start();

        new Thread(() -> {
            synchronized(vec){
                vec.remove(0);
                vec.remove(0);
                vec.remove(0);
            }
        }).start();

        // 출력
        new Thread(() -> {
            try {
                Thread.sleep(1000); // 쓰레드가 다 돌때까지 1초 대기

                System.out.println("Vector size : " + vec.size());
            } catch (InterruptedException ignored) {
            }
        }).start();
    }
}
```

## Vector 시간 복잡도(CRUD)&#x20;

### 생성&#x20;

* 최선 : O(1)&#x20;
  * 요소가 배열의 끝에 추가되고, 배열의 크기 조절이 필요 없는 경우&#x20;
* 최악 : O(n)&#x20;
  * 배열이 가득 차서 새로운 배열을 생성하고 기존 요소를 복사해야 하는 경우&#x20;

### 읽기

* 최선 : O(1)&#x20;
  * 인덱스를 통해서 배열에 접근하기에 항상 O(1) 이다.&#x20;
* 최악 : O(1)&#x20;

### 수정

* 최선 : O(1)&#x20;
  * 인덱스를 통해서 배열에 접근하기에 항상 O(1) 이다.&#x20;
* 최악 : O(1)&#x20;

### 삭제&#x20;

* **최선**: O(1)
  * 삭제하려는 요소가 배열의 마지막 요소일 때, 삭제 후 배열 크기를 줄이지 않으므로 O(1)입니다.
* **최악**: O(n)
  * 삭제하려는 요소가 배열의 중간에 위치할 때, 삭제 후 뒤에 있는 요소들을 앞으로 이동시켜야 하므로 O(n)입니다.
