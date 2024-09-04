# 클래스 변수(static) 사용 주의 케이스

## 1. 멀티스레드 환경에서의 데이터 경합&#x20;

* 서버 애플리케이션에서 방문자 수를 추적하기 위해 static 변수를 사용한다고 가정해보자

```java
public class VisitorCounter {
    private static int visitorCount = 0;

    public static void increment() {
        visitorCount++;
    }

    public static int getVisitorCount() {
        return visitorCount;
    }
}

```

* 위 코드에서 `visitorCount` 변수는 `static` 변수로 원자성이 보장되지 않는다.&#x20;
* 때문에, 여러 스레드가 동시에 `increment()` 메서드를 호출하면 데이터 경합이 발생할 수 있다.&#x20;

#### 해결책&#x20;

* 아래와 같이 `Atomic` 변수를 사용하거나, 동기화 블록을 사용해보자.&#x20;

```java
import java.util.concurrent.atomic.AtomicInteger;

public class VisitorCounter {
    private static AtomicInteger visitorCount = new AtomicInteger(0);

    public static void increment() {
        visitorCount.incrementAndGet();
    }

    public static int getVisitorCount() {
        return visitorCount.get();
    }
}
```
