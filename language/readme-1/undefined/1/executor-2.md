# 스레드 풀과 Executor 프레임워크2

## ExecutorService 우아한 종료 - 소개&#x20;

* 고객의 주문을 처리하는 서버를 운영중이라 할 때, 만약 서버 기능을 업데이트 하기 위해 서버를 재시작 한다고 해보자.
* 이때 서버 애플리케이션이 고객의 주문으로 처리하고 있는 도중에 갑자기 재시작 된다면, 해당 고객의 주문이 제대로 진행되지 못할 것이다.&#x20;
* 가장 이상적인 방향은 새로운 주문 요청은 막고, 이미 진행중인 주문은 모두 완료한 다음 서버를 재시작 하는 것이 가장 좋을 것이다.&#x20;
* 이처럼 서비스를 안정적으로 종료하는 것도 매우 중요하다..&#x20;
* 이렇게 문제 없이 우아하게 종료되는 방식을 **graceful shutdown** 이라 한다.&#x20;

### ExecutorService 의 종료 메서드

#### 서비스 종료&#x20;

* `void shutdown()`&#x20;
  * 새로운 작업을 받지 않고, 이미 제출된 작업을 모두 완료한 후 종료&#x20;
  * 논 블로킹 메서드
* `List<Runnable> shutdownNow()`&#x20;
  * 실행 중인 작업을 중단하고, 대기 중인 작업을 반환하며 즉시 종료한다.&#x20;
  * 실행 중인 작업을 중단하기 위해서 인터럽트를 발생시킨다.&#x20;
  * 논 블로킹 메서드&#x20;

#### 서비스 상태 확인&#x20;

* `boolean isShutdown()`
  * 서비스가 종료되었는지 확인한다.
* `boolean isTerminated()`
  * `shutdown()` , `shutdownNow()` 호출 후, 모든 작업이 완료되었는지 확인한다.

#### 작업 완료 대기&#x20;

* `boolean awaitTermination(long timeout, TimeUnit unit) throws InterruptedException`
  * 서비스 종료시 모든 작업이 완료될 때까지 대기한다. 이때 지정된 시간까지만 대기한다.&#x20;
  * 블로킹 메서드

#### close()&#x20;

* `close()` 는 자바 19부터 지원하는 서비즈 종료 메서드이다. 이 메서드는 `shutdown()` 과 같다고 생각하면 된다.
  * 더 정확히는 `shutdown()` 을 호출하고, 하루를 기다려도 작업이 완료되지 않으면 `shutdownNow()` 를 호출한다.
  * 호출한 스레드에 인터럽트가 발생해도 `shutdownNow()` 를 호출한다.

## ExecutorService 우아한 종료 - 구현&#x20;

* `shutdown()` 을 호출해서 이미 들어온 모든 작업을 다 처리하고 서비스를 우아하게 종료 하는 것이 가장 이상적이지만, 갑자기 요청이 너무 많이 들어와서 큐에 대기중인 작업이 너무 많아 작업 와료 어렵거나, 작업이 너무 오래 걸리거나, 또는 버그가 발생해서 특정 작업이 끝나지 않을 수 있다.&#x20;
* 이렇게 되면 서비스가 너무 늦게 종료되거나, 종료되지 않는 문제가 발생할 수 있다.&#x20;

#### close()&#x20;

* `close()` 의 경우 이렇게 구현되어 있다. `shutdown()` 을 호출하고, 하루를 기다려도 작업이 완료되지 않으면 `shitdownNow()` 를 호출한다. 하지만 대부분 하루를 기다릴 수 는 없다.&#x20;
* 우선은 `shutdown()` 을 통해 우아한 종료를 시도하고, 10초간 종료되지 않으면 `shutdownNow()` 를 통해 강제 종료하는 방식을 구현해보자.&#x20;

### 코드 분석&#x20;

#### 작업 처리에 필요한 시간&#x20;

* `taskA` , `taskB` , `taskC` : 1초&#x20;
* `longTask` : 100초

#### 서비스 종료&#x20;

```java
es.shutdown();
```

* 새로운 작업을 받지 않는다. 처리 중이거나, 쿠에 이미 대기중인 작업은 처리한다. 이후에 풀의 스레드를 종료한다.&#x20;
* `shutdown()` 은 블로킹 메서드가 아니다.. 서비스가 종료될 때 까지 `main` 스레드가 대기하지 않는다. `main` 스레드는 바로 다음 코드를 호출한다.&#x20;

```java
if (!es.awaitTermination(10, TimeUnit.SECONDS)){...}
```

* 블로킹 메서드이다.&#x20;
* `main` 스레드는 대기하면서 서비스 종료를 10초간 기다린다.&#x20;
  * 만약 10초 안에 모든 작업이 완료된다면 `true` 를 반환한다.&#x20;
* 여기서 `taskA`, `taskB`, `taskC` 의 수행이 완료된다. 그런데 `longTask` 는 10초가 지나도 완료되지 않았다.&#x20;
  * 따라서 `false` 를 반환한다.&#x20;

#### 서비스 정상 종료 실패 -> 강제 종료 시도&#x20;

```java
// 정상 종료가 너무 오래 걸리면
log("서비스 정상 종료 실패 -> 강제 종료 시도");
es.shutdownNow();
// 작업(자원정리)이 취소될 때 까지 대기한다.
if(!es.awaitTermination(10, TimeUnit.SECONDS)){
    log("서비스가 종료되지 않았습니다.");
}
```

* 정상 종료가 10초 이상 너무 오래 걸렸다.&#x20;
* `shutdownNow()` 를 통해 강제 종료에 들어간다. `shutdown()` 과 마찬가지로 블로킹 메서드가 아니다.&#x20;
* 강제 종료를 하면 작업 중인 스레드에 인터럽트가 발생한다. 다음 로그를 통해 인터럽트를 확인할 수 있다.&#x20;
* 인터럽트가 발생하면서 스레드도 작업을 종료하고, `shutdownNow()` 를 통한 강제 `shutdown` 도 완료된다.

#### 서비스 종료 실패&#x20;

* 그런데 마지막에 강제 종료인 `es.shutdownNow()` 를 호출한 다음 왜 10초를 기다릴까?&#x20;
* **`shutdownNow()` 가 작업 중인 스레드에 인터럽트를 호출하는 것은 맞다. 인터럽트를 호출하더라도 여러가지 이유로 작업에 시간이 걸릴 수 있다.**&#x20;
  * **인터럽트 이후에 자원을 정리하는 어떤 간단한 작업을 수행할 수 있다. 이런 시간을 기다려주는 것이다.**&#x20;
* **극단적이지만 최악의 경우 스레드가 인터럽트를 받을 수 없는 코드를 수행할 수 있다. 이 경우 인터럽트 예외가 발생하지 않고 스레드도 계속 수행한다.**&#x20;
  * 때문에, 10초간 대기해도 작업이 완료되지 않는다면 **"서비스가 종료되지 않았습니다."** 라는 로그를 남겨두어야 나중에 문제를 확인할 수 있다 .

#### 인터럽트를 받을수 없는 코드 &#x20;

```java
package thread.executor;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;

public class ExecutorShutdownMain {

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(2);
        es.execute(new RunnableTask("taskA"));
        es.execute(new RunnableTask("taskB"));
        es.execute(new RunnableTask("taskC"));
        es.execute(new RunnableTask("longTask", 100_000));  // 100초 대기
        printState(es);
        log("== shutdown 시작 ==");
        shutdownAndAwaitTermination(es);
        log("== shutdown 완료 ==");
        printState(es);
    }

    private static void shutdownAndAwaitTermination(ExecutorService es) {
        es.shutdown();  // non-blocking, 새로운 작업을 받지 않고 처리중이거나, 큐에 이미 대기중에 작업은 처리한다. 이후에 풀의 스레드를 종료한다.
        try {
            // 이미 대기중인 작업들을 모두 완료할 때까지 10초 기다린다.
            if(!es.awaitTermination(10, TimeUnit.SECONDS)){
                // 정상 종료가 너무 오래 걸리면
                log("서비스 정상 종료 실패 -> 강제 종료 시도");
                es.shutdownNow();
                // 작업(자원정리)이 취소될 때 까지 대기한다.
                if(!es.awaitTermination(10, TimeUnit.SECONDS)){
                    log("서비스가 종료되지 않았습니다.");
                }
            }
        } catch (InterruptedException e) {
            // awaitTermination() 으로 대기중인 서비스가 현재 스레드가 대기할 수 있다.
            es.shutdownNow();
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 18.46.35.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 18.46.41.png" alt="" width="563"><figcaption></figcaption></figure>

## Executor 스레드 풀 관리 - 소개&#x20;

* `ExecutorService` 의 기본 구현체인 `ThreadPoolExecutor` 의 생성자는 다음 속성을 사용한다.&#x20;
  * `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
  * `maximumPoolSize` : 스레드 풀에서 관리되는 최대 스레드 수&#x20;
  * `keepAliveTime`, `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 초과 스레드가 생존할 수 있는 대기 시간, 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.&#x20;
  * `BlockingQueue workQueue` : 작업을 보관한 블로킹 큐&#x20;
* 아래 코드를 살펴보자.&#x20;

```java
package thread.executor.poolsize;

import thread.executor.RunnableTask;

import java.util.concurrent.*;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class PoolSizeMainV1 {
    public static void main(String[] args) {
        ArrayBlockingQueue<Runnable> workQueue = new ArrayBlockingQueue<>(2);
        ExecutorService es = new ThreadPoolExecutor(2, 4, 3000, TimeUnit.MILLISECONDS, workQueue);

        printState(es);

        es.execute(new RunnableTask("task1"));
        printState(es, "task1");

        es.execute(new RunnableTask("task2"));
        printState(es, "task2");

        es.execute(new RunnableTask("task3"));
        printState(es, "task3");

        es.execute(new RunnableTask("task4"));
        printState(es, "task4");

        es.execute(new RunnableTask("task5"));
        printState(es, "task5");

        es.execute(new RunnableTask("task6"));
        printState(es, "task6");

        try{
            es.execute(new RunnableTask("task7"));
        }catch (RejectedExecutionException e){
            log("task7 실행 거절 예외 발생 : " + e);
        }

        sleep(3000);
        log("== 작업 수행 완료 ==");
        printState(es);

        sleep(3000);
        log("== maximunPoolSize 대기 시간 초과 ==");
        printState(es);

        es.close();
        log("== shutdown 완료 ==");
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.03.22.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.03.38.png" alt="" width="563"><figcaption></figcaption></figure>

## Executor 스레드 풀 관리 - 분석&#x20;

### 실행 분석&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.04.12.png" alt="" width="563"><figcaption></figcaption></figure>

* `task1` 작업을 요청한다.&#x20;
* `Executor` 는 스레드 풀에 스레드가 `core` 사이즈 만큼 있는지 확인한다.&#x20;
  * `core` 사이즈 만큼 없다면 스레드를 하나 생성한다 .
  * 작업을 처리하기 위해 스레드를 하나 생성했기 때문에, 작업을 큐에 넣을 필요 없이, 해당 스레드가 바로 작업을 처리한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.05.31.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.05.38.png" alt="" width="563"><figcaption></figcaption></figure>

* `task2` 작업을 요청한다.&#x20;
* `Executor` 는 스레드 풀에 스레드가 `core` 사이즈만큼 있는지 확인한다.&#x20;
  * 아직 `core` 사이즈 만큼 없으므로 스레드를 하나 생성한다.&#x20;
* 새로 만들어진 스레드2가 `task2` 를 처리한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.07.11.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.08.37.png" alt="" width="563"><figcaption></figcaption></figure>

* `task3` 작업을 요청한다.&#x20;
* `Executor` 는 스레드 풀에 스레드가 `core` 사이즈 만큼 있는지 확인한다.&#x20;
* `core` 사이즈 만큼 스레드가 이미 만들어져 있고, 스레드 풀에 사용할 수 있는 스레드가 없으므로 이 경우 큐에 작업을 보관한다.&#x20;
* `task4` 작업 또한 큐에 작업을 보관한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.09.26.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.11.27.png" alt="" width="563"><figcaption></figcaption></figure>

* `task5` 작업을 요청한다.&#x20;
* `Executor` 는 스레드 풀의 스레드 core 사이즈 만큼 차있는지 확인한다. -> 차있다.&#x20;
* `Executor` 는 큐에 보관을 시도한다. -> 큐가 가득 찼다.&#x20;
* 큐가 가득 차면 긴급 상황으로 `maximumPoolSize` 까지 초과 스레드를 만들어서 작업한다.&#x20;
  * 초과 스레드인 스레드3 을 만든다.&#x20;
* 스레드3 이 작업을 처리한다.&#x20;
* `task6` 또한 초과 스레드가 작업을 처리한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.12.00.png" alt="" width="563"><figcaption></figcaption></figure>

* `task7` 작업을 요청한다.&#x20;
* 큐가 가득 찼다.&#x20;
* 스레드 풀의 스레드도 max 사이즈 만큰 가득 찼다.&#x20;
* `RejectedExecutionException` 이 발생한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.13.14.png" alt="" width="563"><figcaption></figcaption></figure>

* 모든 작업이 완료되고, 초과 스레드들은 지정된 시간동안 작업을 하지 않고 대기하면 제거된다.&#x20;
  * 긴급한 작업이 끝난 것으로 이해하면 된다.&#x20;
* **여기서 참고로 작업 요청이 계속해서 들어오면 지정된 시간은 계속 초기화된다.**&#x20;

## Executor 전략 - 고정 풀 전략&#x20;

### Executor 스레드 풀 관리 - 다양한 전략&#x20;

* `ThreadPoolExecutor` 를 사용하면 스레드 풀에 사용되는 숫자와 블로킹 큐등 다양한 속성을 조절할 수 있다.
* 자바는 Executors 클래스를 통해 3가지 기본 전략을 제공한다.&#x20;
  * **`newSingleThreadPool()`**: 단일 스레드 풀 전략&#x20;
  * **`newFixedThreadPool(nThreads)`**: 고정 스레드 풀 전략&#x20;
  * **`newCachedThreadPool()`**: 캐시 스레드 풀 전략

### Executor 스레드 풀 관리 - 고정 풀 전략&#x20;

**newFixedThreadPool(nThreads)**

```java
new ThreadPoolExecutor(1, 1,0L, TimeUnit.MILLISECONDS,
                             new LinkedBlockingQueue<Runnable>())
```

* 스레드 풀에 `nThreads` 만큼의 기본 스레드를 생성한다. 초과 스레드는 생성하지 않는다.&#x20;
* 큐 사이즈에 제한이 없다. ( `LinkedBlockingQueue` )&#x20;
* **스레드 수가 고정되어 있기 때문에 CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식이다.**

```java
package thread.executor.poolsize;

import thread.executor.RunnableTask;

import java.util.concurrent.*;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;

public class PoolSizeMainV2 {
    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(2);
//        ExecutorService es = new ThreadPoolExecutor(2, 2,
//                0L, TimeUnit.MILLISECONDS,
//                new LinkedBlockingQueue<Runnable>());
        log("pool 생성");
        printState(es);

        for (int i = 1; i <= 6; i++) {
            String taskName = "task" + i;
            es.execute(new RunnableTask(taskName));
            printState(es, taskName);
        }

        es.close();
        log("== shutdown 완료 ==");
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.17.49.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.17.57.png" alt="" width="563"><figcaption></figcaption></figure>

#### 특징&#x20;

* 스레드 수가 고정되어 있기 떄문에, CPU, 메모리 리소스가 어느정도 예측 가능한 안정적인 방식이다. 큐 사이즈도 제한이 없어서 작업을 많이 담아두어도 문제가 없다.&#x20;

#### 주의&#x20;

* 이 방식의 가장 큰 장점은 스레드 수가 고정되어서 CPU, 메모리 리소스가 어느 정도 예측이 가능하다는 점이다. 따라서 일반적인 상황에 가장 안정적으로 서비스를 운영할 수 있다. 하지만 상황에 따라 장점이 가장 큰 단점이 되기도 한다.&#x20;

#### 상황1 - 점진적인 사용자 확대

* 개발한 서비스가 잘 되어서 사용자가 점점 늘어난다.&#x20;
* 고정 스레드 정략을 사용해서 서비스를 안정적으로 잘 운영했는데, 언젠가부터 사용자들이 서비스 응답이 점점 느려진다고 항의한다. &#x20;

#### 상황2 - 갑작스런 요청 증가&#x20;

* 마케팅 팀의 이벤트가 성공하면서 갑자기 사용자가 폭증했다.&#x20;
* 고객은 응답을 받지 못한다고 항의한다.&#x20;

#### 확인&#x20;

* **개발자는 급하게 CPU, 메모리 사용량을 확인해보는데, 아무런 문제 없이 여유있고, 안정적으로 서비스가 운영되고 있다.**&#x20;
* **고정 스레드 정략은 스레드 수가 고정되어 있다. 따라서 사용자가 늘어나도 CPU, 메모리 사용량이 확 늘어나지 않는다.**&#x20;
* **큐의 사이즈를 확인해보니, 요청이 수 만건 쌓여있다. 요청이 처리되는 시간보다 쌓이는 시간이 더 빠른 것이다. 참고로  고정 풀 전략의 큐 사이즈는 무한이다.**&#x20;
* **서비스 초기에는 사용자가 적기 때문에, 문제가 되지 않지만, 사용자가 늘어나면서 문제가 될 수 있다.**&#x20;

## Executor 전략 - 캐시 풀 전략&#x20;

#### newCachedThreadPool()

```java
new ThreadPoolExecutor(0, Integer.MAX_VALUE, 60L, TimeUnit.SECONDS,
                               new SynchronousQueue<Runnable>());
```

* 기본 스레드를 사용하지 않고, 60초 생존 주기를 가진 초과 스레드만 사용한다.&#x20;
* 초과 스레드의 수는 제한이 없다.&#x20;
* 큐에 작업을 저장하지 않는다. (`SynchronousQueue`)
* 모든 요청을 대기하지 않고 스레드가 바로바로 처리한다. 따라서 빠른 처리가 가능하다.&#x20;

> #### SynchronousQueue
>
> * `BlockingQueue` 의 인터페이스 중 하나이다.&#x20;
> * 이 큐는 내부에 저장 공간이 없다. 대신에 생산자의 작업을 소비자 스레드에게 직접 전달한다.&#x20;
> * **쉽게 이야기해서 저장 공간의 크기가 0이고, 생산자 스레드가 큐에 작업을 전달하면 소비자 스레드가 큐에서 작업을 꺼낼 때까지 대기한다.**&#x20;
> * **소비자 작업을 요청하면 기다리던 생산자가 소비자에게 직접 작업을 전달하고 반환된다. 그 반대의 경우도 같다.**&#x20;
> * 이름 그대로 생산자와 소비자를 동기화하는 큐이다.&#x20;
>   * 쉽게 이야기해서 중간에 버퍼를 두지 않는 스레드간 직거래라고 생각하면 된다.

```java
package thread.executor.poolsize;

import thread.executor.RunnableTask;

import java.util.concurrent.*;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class PoolSizeMainV3 {
    public static void main(String[] args) {
        ExecutorService es = Executors.newCachedThreadPool();
//        ExecutorService es = new ThreadPoolExecutor(0, Integer.MAX_VALUE,
//                                      3, TimeUnit.SECONDS,
//                                      new SynchronousQueue<Runnable>());
        log("pool 생성");
        printState(es);

        for (int i = 1; i <= 4; i++) {
            String taskName = "task" + i;
            es.execute(new RunnableTask(taskName));
            printState(es, taskName);
        }

        sleep(3000);
        log("== 작업 수행 완료 ==");
        printState(es);

        sleep(3000);
        log("== maximumPoolSize 대기 시간 초과 ==");
        printState(es);

        es.close();
        log("== shutdown 완료 ==");
        printState(es);
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.28.31.png" alt="" width="563"><figcaption></figcaption></figure>

#### 특징&#x20;

* 캐시 스레드 풀 전략은 매우 빠르고, 유연한 전략이다.&#x20;
* 이 전략은 기본 스레드도 없고, 대기 큐에 작업을 쌓지도 않는다. 대신에 작업 요청이 오면 초과 스레드로 작업을 바로바로 처리한다. 따라서 빠른 처리가 가능하다. 초과 스레드의 수도 제한이 없기 때문에, CPU, 메모리 자원만 허용한다면 시스템의 자원을 최대로 사용할 수 있다.
  * **결과적으로 이 전략의 모든 작업은 초과 스레드가 담당한다.**&#x20;
* 추가로 초과 스레드는 60초간 생존하기 때문에, 작업 수에 맞추어 적절한 수의 스레드가 재사용된다. 이런 특징 때문에, 요청이 갑자기 증가하면 스레드도 값자기 증가하고, 요청이 줄어들면 스레드도 점점 줄어든다 .
* 이 전략은 작업의 요청 수에 따라서 스레드도 증가하고 감소하므로, 매우 유연한 전략이다.&#x20;

#### 주의&#x20;

* 이 방식은 작업 수에 맞추어 스레드 수가 변하기 때문에, 작업의 처리 속도가 빠르고, CPU, 메모리를 매우 유연하게 사용할 수 있다는 장점이 있다.&#x20;
* 하지만 이는 가장 큰 단점이 되기도 한다.&#x20;

#### 상황1 - 점진적인 사용자 확대&#x20;

* 개발한 서비스가 잘 되어서 사용자가 점점 늘어난다.&#x20;
* 캐시 스레드 전략을 사용하면 이런 경우 크게 문제가 되지 않는다.&#x20;
* 캐시 스레드 전략은 이런 경우에는 문제를 빠르게 찾을 수 있다. 사용자가 점점 증가하면서 스레드 사용량도 함께 늘어난다.&#x20;
* 따라서 CPU 메모리의 사용량도 자연스럽게 증가한다. 물론 CPU, 메모리 자원은 한계가 있기 때문에 적절한 시점에 시스템을 증설해야 한다. 그렇지 않으면 CPU, 메모 리 같은 시스템 자원을 너무 많이 사용하면서 시스템이 다운될 수 있다.

#### 상황2 - 갑작스런 요청 증가&#x20;

* 마케팅 팀의 이벤트가 대성공 하면서 갑자기 사용자가 폭증했다.&#x20;
* 고객은 응답을 받지 못한다고 항의한다.
* 개발자는 급하게 CPU, 메모리 사용량을 확인해보는데, CPU 사용량이 100%이고, 메모리 사용량도 지나치게 높아져있다.&#x20;
* 스레드 수를 확인해보니 스레드가 수 천개 실행되고 있다. 너무 많은 스레드가 작업을 처리하면서 시스템 전체가 느려지는 현상이 발생한다.&#x20;
* 캐시 스레드 풀 전략은 스레드가 무한으로 생성될 수 있다.&#x20;
* 수 천개의 스레드가 처리하는 속도 보다 더 많은 작업이 들어온다.&#x20;
* 시스템은 너무 많은 스레드에 잠식 당해서 거의 다운된다. 메모리도 거의 다 사용되어 버린다.&#x20;
* 시스템이 멈추는 장애가 발생한다

## Executor 전략 - 사용자 정의 풀 전략&#x20;

* 위에 고정 풀 전략, 캐시 풀 전략의 상황을 해결하기 위해 세분화된 전략을 사용해 볼 수 있다.&#x20;
* 그 전략은 다음과 같다.&#x20;
  * **일반**: 일반적인 상황에는 CPU, 메모리 자원을 예측할 수 있도록 고정 크기의 스레드로 서비스를 안정적으로 운영 한다.&#x20;
  * **긴급**: 사용자의 요청이 갑자기 증가하면 긴급하게 스레드를 추가로 투입해서 작업을 빠르게 처리한다.&#x20;
  * **거절**: 사용자의 요청이 폭증해서 긴급 대응도 어렵다면 사용자의 요청을 거절한다.

```java
ExecutorService es = new ThreadPoolExecutor(100, 200, 
                                60, TimeUnit.SECONDS, new
                                ArrayBlockingQueue<>(1000));
```

* 100개의 기본 스레드를 사용한다.&#x20;
* 추가로 긴급 대응 가능한 긴급 스레드 100개를 사용한다. 긴급 스레드는 60초의 생존 주기를 가진다.
* 1000개의 작업이 큐에 대기할 수 있다.

```java
package thread.executor.poolsize;

import thread.executor.RunnableTask;

import java.util.concurrent.*;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;

public class PoolSizeMainV4 {

//    static final int TASK_SIZE = 1100;  // 1. 일반
//    static final int TASK_SIZE = 1200;  // 2. 긴급
    static final int TASK_SIZE = 1201;  // 3. 거절

    public static void main(String[] args) {
        ExecutorService es = new ThreadPoolExecutor(100, 200,
                                60, TimeUnit.SECONDS,
                                            new ArrayBlockingQueue<>(1000));
        printState(es);

        long startMs = System.currentTimeMillis();
        for (int i = 0; i < TASK_SIZE; i++) {
            String taskName = "task" + i;
            try{
                es.execute(new RunnableTask(taskName));
                printState(es, taskName);
            } catch (RejectedExecutionException e){
                log(taskName + " -> " + e);
            }
        }

        es.close();
        long endMs = System.currentTimeMillis();
        log("time : " + (endMs - startMs));
    }
}
```

* `TASK_SIZE` 가 1200 개를 넘어가게 되면 다음과 같이 예외 로그를 확인할 수 있다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.37.18.png" alt="" width="563"><figcaption></figcaption></figure>

## Executor 예외 정책&#x20;

* `ThreadPoolExecutor` 에 작업을 요청할 때, 큐도 가득차고, 초과 스레드도 더는 할당할 수 없다면 작업을 거절한다.&#x20;
* `ThreadPoolExecutor` 는 작업을 거절하는 다양한 정책을 제공한다.&#x20;
  * **`AbortPolicy`** : 새로운 작업을 제출할 때 `RejectedExecutionException` 을 발생시킨다. 기본 정책이다.
  * **`DiscardPolicy`**: 새로운 작업을 조용히 버린다.&#x20;
  * **`CallerRunsPolicy`**: 새로운 작업을 제출한 스레드가 대신해서 직접 작업을 실행한다.&#x20;
  * **사용자 정의( `RejectedExecutionHandler`)** : 개발자가 직접 정의한 거절 정책을 사용할 수 있다.

### AbortPolicy

### DiscardPolicy&#x20;

### CallerRunsPolicy&#x20;

* 호출한 스레드가 직접 작업을 수행하게 한다. 이로 인해 새로운 작업을 제출하는 스레드의 속도가 느려질 수 있다.

#### CallerRunsPolicy 특징&#x20;

* 이 정책의 특징은 생산자 스레드가 소비자 대신 일을 수행하는 것도 있지만, 생산자 스레드가 대신 일을 수행하는 덕분에 작업의 생산 자체가 느려진다는 점이다.&#x20;
* 덕분의 작업의 생산 속도가 너무 빠르다면, 생산 속도를 조절할 수 있다.&#x20;

```java
package thread.executor.reject;

import thread.executor.RunnableTask;

import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class RejectMainV3 {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1,
                0, TimeUnit.SECONDS, new SynchronousQueue<>(), new ThreadPoolExecutor.CallerRunsPolicy());

        executor.submit(new RunnableTask("task1"));
        executor.submit(new RunnableTask("task2"));
        executor.submit(new RunnableTask("task3"));
        executor.submit(new RunnableTask("task4"));

        executor.close();
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.41.08.png" alt="" width="563"><figcaption></figcaption></figure>

### 사용자 정의 (**RejectedExecutionHandler**)

```java
package thread.executor.reject;

import thread.executor.RunnableTask;

import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.SynchronousQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

import static util.MyLogger.log;

public class RejectMainV4 {
    public static void main(String[] args) {
        ThreadPoolExecutor executor = new ThreadPoolExecutor(1, 1,
                0, TimeUnit.SECONDS, new SynchronousQueue<>(), new MyRejectedExecutionHandler());

        executor.submit(new RunnableTask("task1"));
        executor.submit(new RunnableTask("task2"));
        executor.submit(new RunnableTask("task3"));
        executor.submit(new RunnableTask("task4"));

        executor.close();
    }

    static class MyRejectedExecutionHandler implements RejectedExecutionHandler {

        static AtomicInteger count = new AtomicInteger(0);

        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            int i = count.incrementAndGet();
            log("[경고] 누적된 거절 수 : " + i);
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 19.43.37.png" alt=""><figcaption></figcaption></figure>
