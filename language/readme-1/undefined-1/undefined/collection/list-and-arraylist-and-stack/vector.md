# Vector

> 참고 링크
>
> [https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0](https://inpa.tistory.com/entry/JCF-%F0%9F%A7%B1-ArrayList-vs-Vector-%EB%8F%99%EA%B8%B0%ED%99%94-%EC%B0%A8%EC%9D%B4-%EC%9D%B4%ED%95%B4%ED%95%98%EA%B8%B0)

## Vector&#x20;

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
  **-> 심지어 `deprecated` 된 클래스이다.** \
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
