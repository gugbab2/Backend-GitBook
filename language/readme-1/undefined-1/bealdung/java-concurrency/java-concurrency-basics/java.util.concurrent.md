# java.util.concurrent

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html)[https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)[https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html)\
> [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html)\
> [https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)\
> \
> [https://mangkyu.tistory.com/259](https://mangkyu.tistory.com/259)\
> [https://www.baeldung.com/java-util-concurrent](https://www.baeldung.com/java-util-concurrent)

## 1. Executor

* 동시에 여러 요청을 처리해야 하는 경우에 매번 새로운 쓰레드를 만드는 것은 비효율적이다.&#x20;
* 그래서 쓰레드를 미리 만들어두고 재사용하기 위한 쓰레드 풀(Thread Pool) 이 등장하게 되었는데,&#x20;
* `Executor` 인터페이스는 쓰레드 풀의 구현을 위한 인터페이스이다.&#x20;
* **이러한 `Executor` 인터페이스를 간단히 정리하면 다음과 같다.**&#x20;
  * **등록된 작업(`Runnable`) 을 실행하기 위한 인터페이스**&#x20;
  * **작업 등록과 작업 실행 중에서 작업 실행만을 책임짐**&#x20;
* 쓰레드는 크게 작업의 등록과 실행으로 나누어진다. 그 중에서도 **`Executor` 인터페이스는 인터페이스 분리 원칙(Interface Segregation Principle) 에 맞게 등록된 작업을 실행하는 책임만 갖는다.** \
  \-> 그래서 전달 받은 작업(`Runnable`) 을 실행하는 메서드만 가지고 있다.&#x20;

```java
public interface Executor {
    void execute(Runnable command);
}
```

* `Executor` 인터페이스는 개발자들이 해당 작업의 실행과 쓰레드의 사용 및 스케줄링 등으로부터 벗어날 수 있도록 도와준다.&#x20;
* 단순히 전달 받은 `Runnable` 작업을 사용하는 코드를 `Executor` 로 구현하면 다음과 같다.&#x20;

```java
public class RunExecutor implements Executor {
    @Override 
    public void execute(final Runnable command) {
        command.run(); a
    }
}

@Test
void executorRun() {
    final Runnable runnable = () 
            -> System.out.println("Thread : " + Thread.cuncurrentThread.getName());

    Executor executor = new RunExecutor(); 
    executor.execute(runnable);
}
```

* 하지만 위와 같은 코드는  단순히 객체의 메서드를 호출하는 것이므로, 새로운 쓰레드가 아닌 메인 쓰레드에서 실행된다.&#x20;
* 만약 위의 코드를 새로운 쓰레드에서 실행시키려면 Executor 의 execute 메서드를 다음과 같이 수정하면 된다.&#x20;

```java
public class StartExecutor implements Executor {
    @Override 
    public void execute(final Runnable command) {
        new Thread(command).start(); 
    }
}

@Test
void executorRun() {
    final Runnable runnable = () 
            -> System.out.println("Thread : " + Thread.cuncurrentThread.getName());

    Executor executor = new StartExecutor(); 
    executor.execute(runnable);
}
```

## 2. ExecutorService&#x20;

* `ExecutorService` 는 작업(`Runnable`, `Callable`) 등록을 위한 인터페이스이다.&#x20;
* **`ExecutorService` 는 `Executor` 를 상속 받아서 작업 등록 뿐 아니라 실행을 위한 책임도 갖는다.**
* 그래서 쓰레드 풀은 기본적으로 `ExecutorService` 인터페이스를 구현한다.&#x20;
  * 대표적으로 `ThreadPoolExecutor` 가 `ExecutorService` 의 구현체인데,&#x20;
  * `ThreadPoolExecutor` 내부에 있는 블로킹 큐에 작업들을 등록해둔다.&#x20;

<figure><img src="../../../../../../.gitbook/assets/img.gif" alt="" width="540"><figcaption></figcaption></figure>

* 위와 같이 크기가 2인 쓰레드 풀이 있다고 가정했을 때, 각각의 쓰레드는 작업을 할당 받아 처리한다.&#x20;
* 만약 사용 가능한 쓰레드가 없다면 작업은 블로킹 큐에 대기한다. 그러다 쓰레드가 작업이 끝나면 다음 작업을 할당받게 되는 것이다.&#x20;
* 이러한 `ExecutorService` 가 제공하는 퍼블릭 메서드들은 다음과 같이 분류 가능하다.&#x20;
  * 라이프사이클 관리를 위한 기능들
  * 비동기 작업을 위한 기능들 (Thread)&#x20;

### 라이프사이클 관리를 위한 기능들&#x20;

#### 제공 메서드&#x20;

* `void shutdown()`
  * 새로운 작업들을 더 이상 받아들이지 않음&#x20;
  * 호출 전에 이미 실행중인 작업들은 그대로 실행이 끝나고 종료된다.&#x20;
* `List<Runnable> shutdownNow()`
  * `shutdown` 기능에 더해 이미 실행 중인 작업들을 인터럽트를 발생시킨다. &#x20;
  * 추후 실행을 위해 대기중인 작업 목록을 반환한다.&#x20;
* `boolean isShutdown()`&#x20;
  * `Executor` 의 `shutdown` 여부를 반환함&#x20;
* `boolean isTerminated()`&#x20;
  * `shuutdown` 실행 후, 모든 작업의 종료 여부를 반환함&#x20;
* `boolean awaitTerminated(long timeout, TimeUnit unit)`&#x20;
  * `shutdown` 실행 후, 지정한 시간 동안 모든 작업이 종료될 때 까지 대기한다.&#x20;
  * 지정한 시간 내에 모든 작업이 종료되었는지 여부를 반환함

#### 라이프사이클 관리 기능 사용 예제&#x20;

* `ExecutorService` 를 만들어 작업을 실행하면, `shutdown` 이 호출되기 전까지 계속해서 작업을 대기하게 된다.&#x20;
* **그러므로 작업이 완료 되었다면 반드시 `shutdown` 을 명시적으로 호출해주어야 한다.**&#x20;

```java
@Test
void shutdown() {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Runnable runnable = () -> System.out.println("Thread : " + Thread.currentThread().getName());
    executorService.execute(runnable); 
    
    // shutdown 호출 
    executorService.shutdown(); 
    
    // shutdown 호출 이후에는 새로운 작업들을 받을 수 없음, 에러 발생
    RejectedExecutionException result = assertThrows(RejectedExecutionException.class, () -> executorService.execute(runnable));
    assertThat(result).isInstanceOf(RejectedExecutionException.class);
}
```

* 만약 작업 실행 후 `shutdown` 을 해주지 않으면 다음과 같이 프로세스가 끝나지 않고, 계속해서 다음 작업을 기다리게 된다.&#x20;
* 다음의 `Main` 메서드를 실행해보면 애플리케이션이 끝나지 않음을 확인할 수 있다.&#x20;

```java
public static void main(String[] args) {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Runnable runnable = () -> System.out.println("Thread: " + Thread.currentThread().getName());
    executorService.execute(runnable);
}
```

* shutdown 과 shutdownNow 시에 중요한 것은, 만약 실행중인 작업들에서 인터럽트 여부에 따른 처리 코드가 없다면 계속 실행된다는 것이다.&#x20;
* 그러므로 필요하다면 다음과 같이 인터럽트 시 추가적인 조치를 취해 주어야 한다.&#x20;

<pre class="language-java"><code class="lang-java">@Test
void shutdownNow() throws InterruptedException { 
    Runnable runnable = () -> {
<strong>        System.out.println("Start");
</strong>        while(true){
            if(Thread.currentThread().isInterrupted()){
                System.out.println("Interrupt");
                break; 
            }
        }
        System.out.println("End");
    };
    
    ExecutorService executorService = Executors.newFixedThreadPool(10); 
    executorService.execute(runnable); 
    
    executorService.shutdownNow();
    Thread.sleep(1000L);
}
</code></pre>

### 비동기 작업을 위한 기능들&#x20;

* `ExecutorService` 는 `Runnable` 과 `Callable` 을 작업으로 사용하기 위한 메서드를 제공한다.&#x20;
* 동시에 여러 작업들을 실행시키는 메서드도 제공하고 있는데, **비동기 작업의 진행을 추적할 수 있도록 `Future` 을 반환한다.**&#x20;
* 반환 된 `Future` 들은 모두 실행된 것이므로 반환된 `isDone` 은 `true` 이다.&#x20;
* 하지만 작업들은 정상적으로 종료 되었을수도 있고, 예외에 의해 종료되었을 수도 있기 때문에, 항상 성공한 것은 아니다.&#x20;

#### 제공 메서드&#x20;

* `submit`&#x20;
  * 실행할 작업들을 추가하고, 작업의 상태와 결과를 포함하는 `Future` 를 반환한다.&#x20;
  * `Future` 의 `get` 을 호출하면 성공적으로 작업이 완료된 후 결과를 얻을 수 있다.&#x20;
* `invokeAll`
  * 모든 결과가 나올 때 까지 대기하는 블로킹 방식의 요청&#x20;
  * 동시에  주어진 작업들을 모두 실행하고, 전부 끝나면 각가의 상태와 결과를 갖는 `List`(`Future`) 를 반환함&#x20;
* `invokeAny`
  * 가장 빨리 실행된 결과가 나올 때 까지 대기하는 블로킹 방식의 요청&#x20;
  * 동시에 주어진 작업들을 모두 실행하고, 가장 빨리 완료된 하나의 결과를 `Callable`의 타입으로 반환함

#### 비동기 작업 기능 사용 예제 (invokeAll) &#x20;

* ~~ExecutorService 의 구현체로는 AbstractExecutorService 가 있는데, ExecutorService 의 메서드들 (submit, invokeAll, invokeAny) 에 대한 기본 구현들을 제공한다.~~&#x20;
* `invokeAll` 은 최대 쓰레드 풀의 크기만큼 작업을 동시에 실행시킨다.&#x20;
* 그러므로 쓰레드가 충분하다면 동시에 실행되는 작업 중 가장 오래 걸리는 시간 만큼 시간이 소요된다.&#x20;
* 하지만, 만약 쓰레드가 부족하다면 대기되는 작업들이 발생하므로 가장 오래 걸리는 작업의 시간에 더해 추가적으로 시간이 필요하다.&#x20;

```java
@Test
void invokeAll() throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(10); 
    Instant start = Instant.now(); 
    
    Callable<String> hello = () -> {
        Thread.sleep(1000L);
        final String result = "Hello";
        System.out.println("result : " + result);
        return result;
    };
    
    Callable<String> world = () -> {
        Thread.sleep(4000L);
        final String result = "World ";
        System.out.println("result : " + result);
        return result;
    };

    Callable<String> gugbab2 = () -> {
        Thread.sleep(2000L);
        final String result = "gugbab2 ";
        System.out.println("result : " + result);
        return result;
    };
    
    // Arrays.asList로 변경
    List<Future<String>> futures = executorService.invokeAll(Arrays.asList(hello, world, gugbab2));  
    for (Future<String> f : futures) {
        System.out.println(f.get());
    }
    
    // 시간 계산에서 getSeconds()를 제거
    System.out.println("time : " + Duration.between(start, Instant.now()).toMillis() + " ms");
    
    executorService.shutdown(); 

}
```

#### 비동기 작업 기능 사용 예제 (invokeAny)

* invokeAny 는 가장 빨리 끝난 작업 결과만을 구하므로, 동시에 실행한 작업들 중에서 가장 짧게 걸리는 작업만큼 시간이 걸린다.&#x20;
* 또한, 가장 빠르게 처리된 작업 외의 나머지 작업들은 완료되지 않았으므로 취소처리 된다.&#x20;

```java
@Test
void invokeAny() throws InterruptedException, ExecutionException {
    ExecutorService executorService = Executors.newFixedThreadPool(10);
    Instant start = Instant.now();

    Callable<String> hello = () -> {
        Thread.sleep(1000L);
        final String result = "Hello";
        System.out.println("result = " + result);
        return result;
    };

    Callable<String> mang = () -> {
        Thread.sleep(2000L);
        final String result = "Mang";
        System.out.println("result = " + result);
        return result;
    };

    Callable<String> kyu = () -> {
        Thread.sleep(3000L);
        final String result = "kyu";
        System.out.println("result = " + result);
        return result;
    };

    String result = executorService.invokeAny(Arrays.asList(hello, mang, kyu));
    System.out.println("result = " + result + " time = " + Duration.between(start, Instant.now()).getSeconds());

    executorService.shutdown();
}
```

## 3. ScheduledExcutorService&#x20;

* `ScheduledExecutorService` 는 `ExecutorService` 를 상속받는 인터페이스로써,&#x20;
* 특정 시간 이후 or 주기적으로 작업을 실행시키는 메서드가 추가되었다.&#x20;
* 그래서 특정 시간대에 작업을 실행하거나 주기적으로 작업을 실행하고 싶을 때 등에 사용할 수 있다.&#x20;

#### 제공 메서드&#x20;

* `schedule`
  * 특정 시간 이후에 작업을 실행시킴&#x20;
* `scheduleAtFixedRate`
  * 특정 시간 이후 처음 작업을 진행시킴&#x20;
  * 작업이 **실행되고** 특정 시간마다 작업을 진행시킴&#x20;
* `schedultWithFixedDelay`
  * 특정 시간 이후 처음 작업을 진행시킴&#x20;
  * 작업이 **완료되고** 특정 시간마다작업을 진행시킴&#x20;

```java
@Test
void schedule() throws InterruptedException {
    Runnable runnable = () -> System.out.println("Thread: " + Thread.currentThread().getName());
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();

    executorService.schedule(runnable, 1, TimeUnit.SECONDS);

    Thread.sleep(2000L);
    executorService.shutdown();
}

@Test
void scheduleAtFixedRate() throws InterruptedException {
    Runnable runnable = () -> {
        System.out.println("Start scheduleAtFixedRate:    " + LocalTime.now());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Finish scheduleAtFixedRate:    " + LocalTime.now());
    };
    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();

    executorService.scheduleAtFixedRate(runnable, 0, 2, TimeUnit.SECONDS);

    Thread.sleep(10000L);
    executorService.shutdown();
}


@Test
void scheduleWithFixedDelay() throws InterruptedException {
    Runnable runnable = () -> {
        System.out.println("Start scheduleWithFixedDelay:    " + LocalTime.now());
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("Finish scheduleWithFixedDelay:    " + LocalTime.now());
    };

    ScheduledExecutorService executorService = Executors.newSingleThreadScheduledExecutor();
    executorService.scheduleWithFixedDelay(runnable, 0, 2, TimeUnit.SECONDS);

    Thread.sleep(10000L);
    executorService.shutdown();
}
```

## 4. Executors

* 앞서 살펴본 `Executor`, `ExecutorService`, `ScheduleExecutorService` 는 쓰레드 풀을 위한 인터페이스이다.&#x20;
* 직접 쓰레드를 다루는 것은 번거로우므로, 이를 도와주는 팩토리 클래스인 `Executors` 가 등장하게 되었다.&#x20;
* **`Executors` 는 고수준의 동시성 프로그래밍 모델로써 `Executor`, `ExecutorService`, `ScheduleExecutorService` 를 구현한 쓰레드 풀을 손쉽게 생성해준다.**&#x20;
* &#x20;**Executors 를 통해 쓰레드의 개수 및 종류를 정할 수 있으며, 이를 통해 쓰레드의 생성과 실행 및 관리가 매우 용이해진다.**&#x20;
* 하지만 쓰레드 풀 생성 시 주의를 해야한다.&#x20;
  * 만약 newFixedThreadPool 을 사용해 2개의 쓰레드를 갖는 쓰레드 풀을 생성했는데, 3개의 작업이라면, 1개의 작업을 대기 후 실행해야 한다..&#x20;

#### 제공 메서드&#x20;

* `newFixedThreadPool`
  * 고정된 쓰레드 개수를 갖는 쓰레드 풀을 생성함&#x20;
  * `ExecutorService` 인터페이스를 구현한 `ThreadPoolExecutor` 객체가 생성됨
* `newCachedThreadPool`
  * 필요할 때 필요한 만큼의 쓰레드 풀을 생성함&#x20;
  * 이미 생성된 쓰레드가 있다면 이를 재활용 할 수 있다.&#x20;
* `newScheduleThreadPool`&#x20;
  * 일정 시간 뒤 혹은 주기적으로 실행되어야 하는 작업을 위한 쓰레드 풀을 생성함&#x20;
  * `ScheduledExecutorService` 인터페이스를 구현한 `ScheduledThreadPoolExecutor` 객체가 생성됨&#x20;
* `newSingleThreadExecutor`, `newSingleThreadScheduledExecutor`
  * 1개의 쓰레드만을 갖는 쓰레드 풀을 생성함&#x20;
  * 각각 `newFixedThreadPool`, `newScheduledThreadPool` 에 1개의 쓰레드만을 생성하도록 한 것이다.&#x20;

## 5. Future&#x20;

* `Future` 는 **비동기 작업의 결과를 나타내는 데 사용된다.**&#x20;
* 비동기적으로 실행되는 작업의 상태나 결과를 나중에 확인할 수 있도록 하는 인터페이스이다.&#x20;
* 주로 `ExecutorService` 에서 비동기 작업을 제출할 떄 반환된다. `Future` 를 사용하면 작업이 완료될 때까지 기다리거나, 작업의 취소, 상태 확인 등을 할 수 있다.&#x20;

#### 제공 메서드&#x20;

* `T get()`
  * 작업이 완료될 때까지 기다렸다가 결과를 반환한다.&#x20;
  * 작업이 완료되면 그 결과를 반환하고, 작업이 완료되지 않았다면 해당 쓰레드가 작업을 완료할 때까지 기다린다.&#x20;
* `T get(long timeout, TimeUnit unit)`&#x20;
  * 지정한 시간(timeout) 동안 작업이 완료되기를 기다린다.&#x20;
  * 지정된 시간이 초과되면 `TimoutException` 을 던지고, 그렇지 않으면 작업 결과를 반환한다.&#x20;
* `boolean cancel(boolean mayInterruptIfRunning)`
  * `mayInterruptIfRunning` 이 `true` 이면 작업이 실행 중일 때도 인터럽트로 작업을 중단할 수 있고,&#x20;
  * `false` 이면 작업이 시작되지 않았거나, 대기 중일 때만 취소된다.&#x20;
* `boolean isCancelled()`
  * 작업이 취소되었는지 여부를 반환한다.&#x20;
* `boolean isDone()`
  * 작업이 완료되었는지 여부를 반환한다.&#x20;































