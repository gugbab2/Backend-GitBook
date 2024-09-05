# 클래스 변수(static) 사용 주의 케이스

> #### 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-8.html#jls-8.3.1.1)

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

#### 문제점

* 위 코드에서 `visitorCount` 변수는 `static` 변수로 원자성이 보장되지 않는다.&#x20;
* 때문에, 여러 스레드가 동시에 `increment()` 메서드를 호출하면 데이터 경합이 발생할 수 있다.&#x20;

#### 해결책&#x20;

* 아래와 같이 `Atomic` 변수를 사용하거나, 동기화 블록(`synchronized`)을 사용해보자. \
  \->  `Atomic` 변수는 non-blocking, `synchronized` 는 blocking 방식을 사용한다.&#x20;

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

## 2. 의존성 주입이 어려움

* 서버 애플리케이션에서 로깅 시스템을 구현하기 위해 `static` 메서드를 사용하는 경우를 생각해보자.

```java
public class Logger {
    public static void log(String message){
        System.out.println(message);
    }
}
```

#### 문제점

* **테스트 어려움** : `static` 메서드는 의존성 주입(DI) 프레임워크를 통해 주입될 수 없으므로, 테스트 환경에서 Mock 객체로 대체하거나 로깅을 비활성화하는 것이 어렵니다.&#x20;
  * **DI 프레임워크는 일반적으로 객체의 라이프사이클을 관리한다. 하지만 `static` 멤버는 클래스가 로드될 때 초기화되며, 프레임워크가 객체를 생성하거나 관리할 기회가 없기 때문에, 의존성 주입이 어렵다.**&#x20;
* **유연성 부족** : 애플리케이션의 규모가 커지면서 다양한 로깅 요구사항이 생길 수 있다. 로깅 대상이나 방식이 바뀌더라도, `static` 메서드는 쉽게 변경되거나 확장할 수 없다.&#x20;

#### 해결책&#x20;

* 아래와 같이 인터페이스와 의존성 주입을 사용하여 로깅 시스템을 구현하는 것이 좋다.&#x20;
  * 이를 통해서 `Logger` 인터페이스를 구현한 클래스들을 의존성 주입으로 사용할 수 있어, 테스트와 확장이 더 쉬워진다.&#x20;

```java
public interface Logger {
    void log(String message);
}

public class ConsoleLogger implemenet Logger {
    @Override
    public void log(String message){
        System.out.println(message);
    }
}
```

## 3. 메모리 누수 및 GC 관리의 어려움&#x20;

* `static` 컬렉션에 많은 데이터를 캐싱하는 경우를 생각해보자.

```java
public class CacheManager {
    private static Map<String, Object> cache = new HashMap<>();
    
    public static void put(String key, Object value){
        cache.put(key, value);
    }
    
    public static Object get(String key){
        return cache.get(key);
    }
}
```

#### 문제점

* **메모리 누수** : `static` 변수는 클래스 로더에 의해 메모리에 유지되며, 애플리케이션이 종료될 때까지 계속 메모리를 점유한다. 특히 큰 데이터를 캐싱하는 경우, 이 데이터가 필요 없어도 GC 에 의해 회수되지 않고 메모리를 계속적으로 차지하게 되어 메모리 누수가 발생할 수 있다.&#x20;

#### 해결책&#x20;

* 적절한 캐시 관리 : 만료 기간을 가진 캐시, 약한 참조(WeakReference) 를 사용하여 캐시 항목을 관리하는 것이 좋다. \
  \-> `WeakHashMap` 을 사용하자&#x20;
  * **`WeahHashMap` 은 `HashMap` 과 다르게, 키에 대한 강한 참조가 존재하지 않는다면, 가비지 컬렉터는 해당 키와 연관된 엔트리를 자동으로 제거한다.**&#x20;

```java
import java.util.WeakHashMap;

public class CacheManager {
    private static WeakHashMap<String, Object> cache = new WeakHashMap<>();

    public static void put(String key, Object value) {
        cache.put(key, value);
    }

    public static Object get(String key) {
        return cache.get(key);
    }
}
```

> #### 강한참조(Strong Reference)&#x20;
>
> * **일반적인 참조이다. 객체가 강한 참조에 의해 참조되는 한, 가비지 컬렉터는 해당 객체를 절대 수거하지 않는다.**&#x20;
> * 예를 들어, `Point p = new Point(0, 0);` 에서 `p` 는 `Point` 객체에 대한 강한 참조이다.&#x20;
>
> #### 약한참조(Weak Reference)&#x20;
>
> * **약한 참조는 가비지 컬렉터가 강한 참조가 없는 객체를 수거할 수 있도록 허용한다.**&#x20;
> * 약한 참조만 존재하는 객체는 가비지 컬렉터가 즉시 수거할 수 있다.&#x20;
> * **약한 참조의 케이스는 아래와같다.**&#x20;
>   * **`null` 로 초기화**&#x20;
>   * **스코프(scope) 종료**&#x20;
>   * **객체 참조의 재할당**&#x20;
>   * **데이터 구조(collection framework ...)에서 제거**&#x20;
