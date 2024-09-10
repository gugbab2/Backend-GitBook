# ArrayDeque

## ArrayDeque

* 동적 배열을 사용하여 데이터를 저장하는 **Java 의 double-ended queue(양방향 큐) 이다.**&#x20;
* 내부적으로 데이터를 저장하고 관리하는 방식은 배열을 사용하되, 원형 배열처럼 동작하여 메모리 공간을 효율적으로 사용한다.&#x20;
* 이를 통해 양쪽 끝에서 삽입과 삭제를 빠르게 수행할 수 있다.&#x20;

```java
// 스택처럼 사용 
public class ArrayDequeStackExample {
    public static void main(String[] args) {
        Deque<Integer> stack = new ArrayDeque<>();

        // 스택에 데이터 추가 (push)
        stack.push(10);
        stack.push(20);
        stack.push(30);
        
        System.out.println("스택 내용: " + stack); // 출력: [30, 20, 10]

        // 스택에서 데이터 제거 (pop)
        int top = stack.pop();
        System.out.println("pop된 값: " + top);  // 출력: 30

        // 스택 내용 확인
        System.out.println("현재 스택: " + stack); // 출력: [20, 10]
    }
}

// 큐처럼 사용 
public class ArrayDequeQueueExample {
    public static void main(String[] args) {
        Deque<Integer> queue = new ArrayDeque<>();

        // 큐에 데이터 추가 (offer)
        queue.offer(10);
        queue.offer(20);
        queue.offer(30);
        
        System.out.println("큐 내용: " + queue); // 출력: [10, 20, 30]

        // 큐에서 데이터 제거 (poll)
        int first = queue.poll();
        System.out.println("poll된 값: " + first);  // 출력: 10

        // 큐 내용 확인
        System.out.println("현재 큐: " + queue); // 출력: [20, 30]
    }
}

```

## Stack vs Deque(ArrayDeque)

* Java 공식 문서에서도 `Stack` 대신 `Deque`를 사용하는 것을 권장하고 있다. `Deque`는 더 현대적이고 유연한 인터페이스를 제공하며, `ArrayDeque`는 스택과 큐의 특성을 모두 제공하기 때문에 훨씬 더 나은 대안이다.
* Stack 을 사용하지 말아야 하는 이유는 다음과 같다.&#x20;
  * Java 에서 `Vector` 는 메서드에는 `synchronized` 가 걸려있지만, 클래스 자체에는 `synchronized` 가 걸려있지 않기 때문에, 완벽한 Thread Safe 가 아니라고 할 수 있다. \
    \-> `Vector` 는 `deprecated` 된 클래스이다.&#x20;
    * 모든 작업에 Lock 이 사용된다.
      * 단일 스레드 실행 성능이 저하 될 수 있다.\
        \-> Thread Safe 가 무조건 좋은 것은 아니다..
      * 단순한 Iterator 의 탐색 작업에서도 get() 메서드 실행 시 매번 Lock 이 발생하게 되므로 오버헤드가 커진다.
  * 추가적으로 Stack 은 Vector 상속받았기 때문에 다중 상속을 지원하지 않는다.
