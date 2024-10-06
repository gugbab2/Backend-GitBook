# ThreadLocal in Java

## 1. ThreadLocal API

* `ThreadLocal` 구조를 사용하면 특정 쓰레드에서만 엑세스 할 수 있는 데이터를 저장할 수 있다.&#x20;
* 특정 쓰레드와 함께 묶일 정수값을 원한다고 가정해보자&#x20;

```java
ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>(); 
```

* 다음으로, 쓰레드에서 이 값을 사용하고 싶을 때 `get()`, `set()` 메서드만 호출하면 된다. \
  \-> 간단히 말해서, `ThreadLocal` 이 쓰레드를 키로 하는 `map` 내부에 데이터를 저장한다고 생각할 수 있다.&#x20;
* 결과적으로 `threadLocalValue` 에서 `get()` 메서드를 호출하면 요청 쓰레드에 대한 `Integer` 값을 얻게 된다.&#x20;

```java
threadLocalValue.set(1);
Integer result = threadLocalValue.get(); 
```

* `withInitial()` 정적 메서드를 통해서 공급자를 전달하여 `ThreadLocal` 인스턴스를 구성할 수 있다.&#x20;

```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);
```

* `ThreadLocal` 에서 값을 제거하려면 `remove()` 메서드를 호출하면 된다.&#x20;

```java
threadLocal.remove();
```

## 2. Map 에 사용자 데이터를 저장

* 지정된 사용자 ID 별로 사용자별 Context 데이터를 저장해야 하는 프로그램을 생각해보자.&#x20;

```java
public class Context {
    private String userName;
    
    public Context(String userName) {
        this.userName = userName; 
    }
}
```

* 사용자 ID 당 하나의 쓰레드를 원한다. `Runnable` 인터페이스 구현하는 `SharedMapWithUserContext` 클래스를 만다.
* `run()` 메서드의 구현은 주어진 사용자 ID 에 대한 `Context` 객체를 반환하는 `UserRepository` 클래스를 통해 일부 데이터베이스를 호출한다.&#x20;
* 그 다음, `userId` 를 키로 `ConcurentHashMap` 에 해당 컨텍스트를 저장한다.&#x20;

```java
public class SharedMapWithUserContext implements Runnable {
    public static Map<Integer, Context> userContextPerUserId = new ConcurrentHashMap<>();
    private Integer userId; 
    private UserRepository userRepository = new UserRepository(); 
    
    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContextPerUserId.put(userId, new Context(userName));
    }
    
    // standard construct
}
```

* 두 개의 다른 `userId` 에 대해 두 개의 쓰레드를 생성하고, 시작하고, `userContextPerUserId` 맵에 두 개의 항목이 있는지 확인하여 코드를 쉽게 테스트 할 수 있다.&#x20;

```java
SharedMapWithUserContext firstUser = new SharedMapWithUserContext(1);
SharedMapWithUserContext secondUser = new SharedMapWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();

assertEquals(SharedMapWithUserContext.userContextPerUserId.size(), 2);
```

## 3. ThreadLocal 에 사용자 데이터를 저장&#x20;

* `ThreadLocal` 을 사용하여 사용자 `Context` 인스턴스를 저장하도록 예제를 수정해보자.&#x20;
* **`ThreadLocal` 을 사용할 때는 모든 `ThreadLocal` 인스턴스가 특정 쓰레드와 연관되어 있기 때문에 매우 주의해야 한다.** \
  **-> 각 쓰레드는 고유한 `ThreadLocal` 인스턴스를 갖는다.**
* 우리 예제에서는 각 특정 `userId` 에 대한 전용 쓰레드가 있으며, 이 쓰레드는 우리가 생성하므로 우리가 완벽하게 제어할 수 있다.&#x20;
* `run()` 메서드는 `set()` 메서드를 사용하여 사용자 컨텍스트를 가져와 `ThreadLocal` 변수에 저장한다.&#x20;

```java
public class ThreadLocalWithUserContext implements Runnable {

    private static ThreadLocal<Context> userContext = new ThreadLocal<>(); 
    private Integer userId; 
    private UserRepository userRepository = new UesrRespository(); 
    
    @Override 
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId); 
        userContext.set(new Context(userName));
        System.out.println("thread context for given userId: " 
         + uesrId + " is: " + userContext.get());
    }    
    
    // standard constructor
}
```

* 주어진 `userId` 에 대한 작업을 실행하는 두 개의 쓰레드를 시작하여 테스트 할 수 있다.&#x20;

```java
ThreadLocalWithUserContext firstUser = new ThreadLocalWithUserContext(1);
ThreadLocalWithUserContext secondUser = new ThreadLocalWithUserContext(2);

new Thread(firstUser).start();
new Thread(secondUser).start();
```

* 이 코드를 실행하면 `ThreadLocal` 이 지정된 쓰레드별로 설정되었음을 출력에서 볼 수 있다.&#x20;

```java
thread context for given userId: 1 is: Context{userNameSecret='18a78f8e-24d2-4abf-91d6-79eaa198123f'}
thread context for given userId: 2 is: Context{userNameSecret='e19f6a0a-253e-423e-8b2b-bca1f471ae5c'}
```

## 4. ThreadLocal 및 쓰레드 풀&#x20;

* `ThreadLocal` 은 특정 값을 각 쓰레드에 한정하기 위한 사용하기 쉬운 API 를 제공한다.&#x20;
* 이는 Java 에서 쓰레드 안정성을 달성하는 합리적인 방법이다.&#x20;
* **그러나, `ThreadLocal` 과 쓰레드 풀을 함께 사용할 때는 특별히(!) 주의해야 한다.**&#x20;
* 이러한 잠재적인 위험을 더 잘 이해하기 위해서 다음의 시나리오를 고려해보자.&#x20;
  * 먼저, 애플리케이션은 풀에서 쓰레드를 빌린다.&#x20;
  * 그런 다음, 특정 값을 현재 쓰레드의 ThreadLocal 에 저장한다.&#x20;
  * 현재 실행이 완료되면, 애플리케이션은 빌린 쓰레드를 풀로 반환한다.&#x20;
  * 시간이 지나면서 애플리케이션은 동일한 쓰레드를 빌려 다른 요청을 처리한다.&#x20;
  * **애플리케이션은 지난번에 필요한 정리를 수행하지 않았으므로 새 요청에 동일한 ThreadLocal 데이터를 재사용할 수 있다.**&#x20;
* 이 문제를 해결하는 한 가지 방법은 ThreadLocal 을 사용한 후 수동으로 제거하는 것이다.&#x20;
* 이 접근 방식은 오류가 발생하기 쉽기에, 엄격한 검토가 필요하다.&#x20;

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class CustomThreadPoolExecutor extends ThreadPoolExecutor {

    public CustomThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit) {
        super(corePoolSize, maximumPoolSize, keepAliveTime, unit, new LinkedBlockingQueue<>());
    }

    @Override
    protected void afterExecute(Runnable r, Throwable t) {
        super.afterExecute(r, t);
        // ThreadLocal 데이터 정리
        clearThreadLocalData();
    }

    private void clearThreadLocalData() {
        // 모든 ThreadLocal 값을 초기화하거나 제거
        MyThreadLocalClass.clear();
    }
}

// ...

public class MyThreadLocalClass {

    // ThreadLocal 선언
    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void set(String value) {
        threadLocal.set(value);
    }

    public static String get() {
        return threadLocal.get();
    }

    // ThreadLocal 데이터 초기화
    public static void clear() {
        threadLocal.remove();  // ThreadLocal에서 값 제거
    }
}

// ...

public class ThreadLocalExample {

    public static void main(String[] args) {
        CustomThreadPoolExecutor executor = new CustomThreadPoolExecutor(2, 4, 60, TimeUnit.SECONDS);

        Runnable task = () -> {
            MyThreadLocalClass.set("ThreadLocal Data: " + Thread.currentThread().getName());
            System.out.println(MyThreadLocalClass.get());
            // 여기서 작업 수행...
        };

        for (int i = 0; i < 5; i++) {
            executor.execute(task);
        }

        executor.shutdown();
    }
}

```
