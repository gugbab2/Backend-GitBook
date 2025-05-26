# 스레드 풀과 Executor 프레임워크1

## 스레드를 직접 사용할 때 문제점

### 문제점

실무에서 스레드를 직접 생성해서 사용하면 다음과 같은 3가지 문제가 있다.

* **스레드 생성 시간으로 인한 성능 문제**
* **스레드 관리 문제**
* `Runnable` **인터페이스의 불편함**

#### 1. 스레드 생성 시간으로 인한 성능 문제

스레드를 사용하려면 먼저 스레드를 생성해야 한다. 그런데 스레드는 다음과 같은 이유로 매우 무겁다.

* **메모리 할당** : 각 스레드는 자신만의 호출 스택(call stack)을 가지고 있어야 한다. 이 호출 스택은 스레드가 실행되는 동안 사용하는 메모리 공간이다. 따라서 스레드를 생성할 때는 이 호출 스택을 위한 메모리를 할당해야 한다.&#x20;
* **운영체제 자원 사용** : 스레드를 생성하는 작업은 운영체제 커널 수준에서 이루어지며, 시스템 콜(system call)을 통해 처리된다. 이는 CPU와 메모리 리소스를 소모하는 작업이다.
* **운영체제 스케줄러 설정** : 새로운 스레드가 생성되면 운영체제의 스케줄러는 이 스레드를 관리하고 실행 순서를 조정해야 한다. 이는 운영체제의 스케줄링 알고리즘에 따라 추가적인 오버헤드가 발생할 수 있다.

> **참고로 스레드 하나는 보통 1MB 이상의 메모리를 사용한다.**

**스레드를 생성하는 작업은 상대적으로 무겁다. 단순히 자바 객체를 하나 생성하는 것과 비교할 수 없을 정도로 큰 작업이다.**

* 예를 들어, 어떤 작업 하나를 수행할 때 마다 스레드를 각각 생성하고 실행한다면, 스레드의 생성 비용 때문에, 이미 많은 시간이 소모된다.
* 아주 가벼운 작업이라면, 작업의 실행 시간보다 스레드의 생성 시간이 더 오래 걸릴 수도 있다.

**이런 문제를 해결하기 위해서 스레드를 재사용하면 처음 생성할 때를 제외하고는 생성을 위한 시간이 들지 않는다.** \
**(따라서 스레드가 아주 빠르게 작업을 수행할 수 있다)**

#### 2. 스레드 관리 문제

서버의 CPU, 메모리 자원은 한정되어 있기 때문에, 스레드는 무한하게 만들 수 없다..

* 예를 들어, 사용자의 주문을 처리하는 서비스라고 가정하자. 그리고 사용자의 주문이 들어올 때 마다 스레드를 만들어서 요청을 처리한다고 가정하겠다.
* 서비스 마케팅을 위해 선착순 할인 이벤트를 진행한다고 가정해보자. 그러면 사용자 가 갑자기 몰려들 수 있다. 평소 \
  동시에 100개 정도의 스레드면 충분했는데, 갑자기 10000개의 스레드가 필요한 상황 이 된다면 CPU, 메모리 자원이 버티지 못할 것이다.
* **이런 문제를 해결하려면 우리 시스템이 버틸 수 있는, 최대 스레드의 수까지만 스레드를 생성할 수 있게 관리해야 한다.**

또한 이런 문제도 있다. 예를 들어 애플리케이션을 종료한다고 가정해보자.

* 이때 안전한 종료를 위해 실행 중인 스레드가 남은 작업은 모두 수행한 다음에 프로그램을 종료하고 싶다거나,
* 또는 급하게 종료해야 해서 인터럽트 등의 신호를 주고 스레드를 종료하고 싶다고 가정해보자.
* 이런 경우에도 스레드가 어딘가에 관리가 되어 있어야한다.

#### 3. `Runnable` 인터페이스의 불편함

```java
public interface Runnable {
     void run();
}
```

**반환 값이 없다 : `run()` 메서드는 반환 값을 가지지 않는다.**

* 따라서 실행 결과를 얻기 위해서는 별도의 메커니즘을 사용해야 한다. 쉽게 이야기해서 스레드의 실행 결과를 직접 받을 수 없다.&#x20;
* 앞에서 공부한 `SumTask` 의 예를 생각해보자. 스레드가 실행한 결과를 멤버 변수에 넣어두고, `join()` 등을 사용해서 스레드가 종료되길 기다린 다음에 멤버 변수에 보관한 값을 받아야 한다.

**예외 처리 : `run()` 메서드는 체크 예외(checked exception)를 던질 수 없다.**&#x20;

* 체크 예외의 처리는 메서드 내부에서 `try-catch` 를 통해 처리해야 한다.

이런 문제를 해결하려면 반환 값도 받을 수 있고, 예외도 좀 더 쉽게 처리할 수 있는 방법이 필요하다. \
추가로 반환 값 뿐만 아니라 해당 스레드에서 발생한 예외도 받을 수 있다면 더 좋을 것이다.

### 해결방안&#x20;

위에서 말한 1,2 번의 문제를 해결하기 위해서는 **스레드 풀(Thread Pool)** 이 필요하다.

**이렇게 스레드 풀이라는 개념을 사용하면 스레드를 재사용할 수 있어서, 재사용시 스레드의 생성 시간을 절약할 수 있다.**\
**그리고 스레드 풀에서 스레드가 관리되기 때문에 필요한 만큼만 스레드를 만들 수 있고, 또 관리할 수 있다.**

* 하지만 스레드 풀을 사용해도 스택을 사용하기에 시스템에 따라 자원이 한정된다는 점과 `Runnable` 인터페이스 사용에 대한 문제점은 가지고 있다.

사실 스레드 풀이라는 것이 별것이 아니다. 그냥 컬렉션에 스레드를 보관하고 재사용할 수 있게 하면 된다. 하지만 스레드 풀에 있는 스레드는 처리할 작업이 없다면, 대기( `WAITING`) 상태로 관리해야 하고, 작업 요청이 오면 `RUNNABLE` 상태로 변경해야 한다.

* 막상 구현하려고 하면 생각보다 매우 복잡하다는 사실을 알게될 것이다. 여기에 생산자 소비자 문제까지 겹친다.&#x20;
  * 생산자 - 프론트엔드 서버&#x20;
  * 소비자 - 백엔드 서버 (스레드 풀에 있는 스레드)&#x20;
* **이런 문제를 한방에 해결해주는 것이 바로 자바가 제공하는 `Executor` 프레임워크다.**
  * `Executor` 프레임워크는 스레드 풀, 스레드 관리, `Runnable` 의 문제점은 물론이고, 생산자 소비자 문제까지 한방에 해결해주는 자바 멀티스레드 최고의 도구이다.
  * 지금까지 우리가 배운 멀티스레드 기술의 총 집합이 여기에 들어있다.

> 참고로 앞서 설명한 이유와 같이 스레드를 사용할 때는 생각보다 고려해야 할 일이 많다.&#x20;
>
> 그래서 실무에서는 스레드를 직접 하나하나 생성해서 사용하는 일이 드물다.&#x20;
>
> 대신에 지금부터 설명할 `Executor` 프레임워크를 주로 사용하는데, 이 기술을 사용하면 매우 편리하게 \
> 멀티스레드 프로그래밍을 할 수 있다.

## Executor 프레임워크 소개&#x20;

### Executor 프레임워크의 주요 구성 요소

#### Executor 인터페이스

가장 간단한 인터페이스로 **작업을 실행**하는 책임을 가지고 있다.

```java
package java.util.concurrent;
public interface Executor {
     void execute(Runnable command);
}
```

#### ExecutorService 인터페이스 - 주요 메서드

`Executor` 인터페이스를 확장해서 **작업 제출,** **제어 기능**을 추가로 제공한다.

주요 메서드로는 `submit()` , `close()` 가 있다.\
(더 많은 기능이 있지만 나머지 기능들은 뒤에서 알아보자)

`Executor` 프레임워크를 사용할 때는 대부분 이 인터페이스를 사용한다.

```java
public interface ExecutorService extends Executor, AutoCloseable {
     <T> Future<T> submit(Callable<T> task);
     @Override
     default void close(){...}
     
     ...
}
```

### 로그 출력 유틸리티 만들기

**`ExecutorService` 의 기본 구현체는** `ThreadPoolExecutor` **이다.**

* `pool` : 스레드 풀에서 관리되는 스레드의 숫자
* `active` : 작업을 수행하는 스레드의 숫자
* `queuedTasks` : 큐에 대기중인 작업의 숫자
* `completedTask` : 완료된 작업의 숫자

`ExecutorService` 인터페이스는 `getPoolSize()`, `getActiveCount()` 와 같은 메서드를 제공하지 않는다. \
이 기능은 `ExecutorService` 의 대표 구현체인 `ThreadPoolExecutor` 를 사용해야 한다.

```java
package thread.executor;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.ThreadPoolExecutor;

import static util.MyLogger.log;

public abstract class ExecutorUtils {

    public static void printState(ExecutorService executorService) {
        if (executorService instanceof ThreadPoolExecutor poolExecutor) {
            int pool = poolExecutor.getPoolSize();
            int active = poolExecutor.getActiveCount();
            int queuedTasks = poolExecutor.getQueue().size();
            long completedTask = poolExecutor.getCompletedTaskCount();
            log("[pool=" + pool + ", active=" + active + ", queuedTasks=" + queuedTasks + ", completedTask=" + completedTask + "]");
        } else {
            log(executorService);
        }
    }
}
```

## ExecutorService 코드로 시작하기

`ExecutorService` 가장 대표적인 구현체는 `ThreadPoolExecutor` 이다.

```java
package thread.executor;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class RunnableTask implements Runnable {

    private final String name;
    private int sleepMs = 1000;

    public RunnableTask(String name) {
        this.name = name;
    }

    public RunnableTask(String name, int sleepMs) {
        this.name = name;
        this.sleepMs = sleepMs;
    }

    @Override
    public void run() {
        log(name + "시작");
        sleep(sleepMs); // 작업 시간 시뮬레이션
        log(name + "완료");
    }
}
```

```java
package thread.executor;

import java.util.concurrent.*;

import static thread.executor.ExecutorUtils.printState;
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class ExecutorBasicMain {
    public static void main(String[] args) {
        ExecutorService es = new ThreadPoolExecutor(2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());
        log("== 초기 상태 ==");
        printState(es);
        es.execute(new RunnableTask("taskA"));
        es.execute(new RunnableTask("taskB"));
        es.execute(new RunnableTask("taskC"));
        es.execute(new RunnableTask("taskD"));
        log("== 작업 수행 중 ==");
        printState(es);

        sleep(3000);
        log("==작업 수행 완료==");
        printState(es);

        es.close();
        log("==showdown 완료==");
        printState(es);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-29 15.15.45.png" alt="" width="436"><figcaption></figcaption></figure>

`ThreadPoolExecutor(ExecutorService)` 는 두가지 요소로 구성되어 있다.&#x20;

* 스레드 풀 : 스레드를 관리한다.
* `BlockingQueue` : 작업을 보관한다.
  * **생산자 소비자 문제를 실행하기 위해서 단순한 큐가 아니라 `BlockingQueue` 를 사용한다.**

생산자가 `es.execute(new RunnableTask("taskA"))` 를 호출하면, `RunnableTask("taskA")` 인스턴스가 `BlockingQueue` 에 보관된다.

* **생산자** : `es.execute(작업)`를 호출하면 내부에서 `BlockingQueue` 에 작업을 보관한다.
  * `main` 스레드가 생산자이다.
* **소비자** : 소비자 중에 하나가 `BlockingQueue` 에 들어있는 작업을 받아서 처리한다.
  * &#x20;스레드 풀에 있는 스레드가 소비자이다.

#### **ThreadPoolExecutor 생성자**

`ThreadPoolExecutor` 의 생성자는 다음 속성을 사용한다.

* `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
* `maximumPoolSize` : 스레드 풀에서 관리되는 최대 스레드 수
* `keepAliveTime`, `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간이다. 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.
* `BlockingQueue workQueue` : 작업을 보관할 블로킹 큐

`new ThreadPoolExecutor(2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());`

* 최대 스레드 수와 `keepAliveTime` , `TimeUnit` unit 에 대한 부분은 뒤에서 따로 설명하겠다.
* 여기서는 `corePoolSize=2`, `maximumPoolSize=2` 를 사용해서 기본 스레드와 최대 스레드 수를 맞추었다. 따라서 풀에서 관리되는 스레드는 2개로 고정된다.&#x20;
* `keepAliveTime`, `TimeUnit unit` 는 0으로 설정했는 데, 이 부분은 뒤에서 설명한다.
* 작업을 보관할 블로킹 큐의 구현체로 `LinkedBlockingQueue` 를 사용했다. 참고로 이 블로킹 큐는 작업을 무한대로 저장할 수 있다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 15.46.04.png" alt="" width="563"><figcaption></figcaption></figure>

### ExecutorService 분석

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.12.png" alt="" width="563"><figcaption></figcaption></figure>

`ThreadPoolExecutor` 를 생성한 시점에는 스레드 풀에 스레드를 미리 만들어두지 않는다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.25.png" alt="" width="563"><figcaption></figcaption></figure>

`main` 스레드가 `es.execute("taskA ~ taskD")` 를 호출한다.

*   참고로 당연한 이야기지만 `main` 스레드는 작업을 전달하고 기다리지 않는다. 전달한 작업은 다른 스레드

    가 실행할 것이다. `main` 스레드는 작업을 큐에 보관까지만 하고 바로 다음 코드를 수행한다.

`taskA~D` 요청이 블로킹 큐에 들어온다.

**최초의 작업이 들어오면 이때 작업을 처리하기 위해 스레드를 만든다.**&#x20;

* **참고로 스레드 풀에 스레드를 미리 만들어두지는 않는다.**

**작업이 들어올 때 마다 `corePoolSize`의 크기까지 스레드를 만든다.**

* 예를 들어, 최초 작업인 `taskA` 가 들어오는 시점에 스레드1 을 생성하고, 다음 작업인 `taskB` 가 들어오는 시점에 스레드2를 생성한다.
* 이런 방식으로 `corePoolSize` 에 지정한 수 만큼 스레드를 스레드 풀에 만든다. 여기서는 2를 설정했으므로 2개까지 만든다.
* `corePoolSize` 까지 스레드가 생성되고 나면, 이후에는 스레드를 생성하지 않고 앞서 만든 스레드를 재사용한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.43.png" alt="" width="563"><figcaption></figcaption></figure>

현재 상황

* 스레드 풀에 관리되는 스레드가 2개이므로 `pool=2`
* 작업을 수행중인 스레드가 2개이므로 `active=2`
* 큐에 대기중인 작업이 2개이므로 `queuedTasks=2`
* 완료된 작업은 없으므로 `completedTasks=0`

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.11.45.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.12.46.png" alt="" width="563"><figcaption></figcaption></figure>

작업이 완료되면 스레드 풀에 스레드를 반납한다.

* **스레드를 반납하면 스레드는 대기( `WAITING` ) 상태로 스레드 풀에 대기한다.**
* 참고로 실제 반납 되는게 아니라, 스레드의 상태가 변경된다고 이해하면 된다.
* 반납된 스레드는 재사용된다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.15.38.png" alt="" width="563"><figcaption></figcaption></figure>

* `taskC` , `taskD` 의 작업을 처리하기 위해 스레드 풀에서 스레드를 꺼내 재사용한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.15.55.png" alt="" width="563"><figcaption></figcaption></figure>

* 작업이 완료되면 스레드는 다시 스레드 풀에서 대기한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.16.24.png" alt="" width="563"><figcaption></figcaption></figure>

* `close()` 를 호출하면 `ThreadPoolExecutor` 가 종료된다. 이때 스레드 풀에 대기하는 스레드도 함께 제거된다.

> `close()` 는 자바 19부터 지원되는 메서드이다. 만약 19 미만 버전을 사용한다면 `shutdown()` 을 호출
>
> 하자. 둘의 차이는 뒤에서 설명한다.

## Runnable 의 불편함&#x20;

위에서 Runnable 인터페이스는 다음과 같은 불편함이 있다고 했다.&#x20;

* **반환값이 없다** : `run()` 메서드는 반환 값을 가지지 않는다. 따라서 실행 결과를 얻기 위해서는 별도의 메커니즘을 \
  사용해야 한다.&#x20;
  * 쉽게 이야기해서 스레드의 실행 결과를 직접 받을 수 없다.&#x20;
  * 앞에서 공부한 `SumTask`의 예를 생각해보자. 스레드가 실행한 결과를 멤버 변수에 넣어두고, `join()` 등을 \
    사용해서 스레드가 종료되길 기다린 다음에 멤버 변수를 통해 값을 받아야 한다.
* **예외 처리** : `run()` 메서드는 체크 예외(checked exception)를 던질 수 없다. 체크 예외의 처리는 메서드 내부에서 처리해야 한다.
  * 물론 최근 예외처리 기조가 언체크 예외를 선호하기 때문에, 무조건적인 단점이라고는 볼 수 없다.

### Runnable 사용

이해를 돕기 위해서 `Runnable` 을 통해 별도의 스레드에서 무작위 값을 하나 구하는 간단한 코드를 만들어보자.&#x20;

```java
package thread.executor.future;

import java.util.Random;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class RunnableMain {
    public static void main(String[] args) throws InterruptedException {
        MyRunnable task = new MyRunnable();
        Thread thread = new Thread(task, "Thread-1");
        thread.start();
        thread.join();
        int result = task.value;
        log("result value = " + result);
    }

    static class MyRunnable implements Runnable {

        int value;

        @Override
        public void run() {
            log("Runnable 시작");
            sleep(2000);
            value = new Random().nextInt(10);
            log("create value = " + value);
            log("Runnable 완료");
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.12.47.png" alt="" width="563"><figcaption></figcaption></figure>

#### 실행 결과

* 무작위 값이므로 결과는 다를 수 있다.
* 프로그램이 시작되면 `Thread-1` 이라는 별도의 스레드를 만든다.
* `Thread-1` 이 수행하는 `MyRunnable` 은 무작위 값을 하나 구한 다음에 `value` 필드에 보관한다.
* 클라이언트인 `main` 스레드가 이 별도의 스레드에서 만든 무작위 값을 얻어오려면 `Thread-1` 스레드가 종료될 \
  때까지 기다려야 한다. 그래서 `main` 스레드는 `join()` 을 호출해서 대기한다.
* 이후에 `main` 스레드에서 `MyRunnable` 인스턴스의 `value` 필드를 통해 최종 무작위 값을 획득한다.

~~**별도의 스레드에서 만든 무작위 값 하나 받아오는 코드가 위처럼 복잡하다 ;;**~~

작업 스레드는 간단히 값을 `return` 해서 반환하고, 요청 스레드는 그 반환 값을 받을 수 있다면 정말 간단할 것이다.

**이런 문제를 해결하기 위해서 `Executor` 프레임워크는 `Callable` 과 `Future` 이라는 인터페이스를 도입했다.**

## Future1 - 시작

### Runnable 과 Callable 비교

#### Runnable

```java
package java.lang;

public interface Runnable {
   void run();
}
```

* `Runnable` 의 `run()` 은 반환 타입이 `void` 이다.&#x20;
  * **따라서 값을 반환받을 수 없다.**&#x20;
* 예외를 던지지 않는다.
  * **따라서 해당 인터페이스를 구현하는 모든 메서드는 체크 예외를 던질 수 없다.**
  * 물론 런타임 예외는 제외한다.

#### Callable&#x20;

```java
package java.util.concurrent;

public interface Callable<V> {
   V call() throws Exception;
}
```

* `java.util.concurrent` 에서 제공되는 기능이다.
* `Callable` 의 `call()` 은 반환 타입이 제네릭이다.
  * **따라서 값을 반환할 수 있다.**
* `throw Excetpion` 예외가 선언되어 있다.
  * **따라서 해당 인터페이스를 구현하는 모든 메서드는 체크 예외인 `Exception` 과 그 하위 예외를 던질 수 있다.**

### Callable 과 Future 사용&#x20;

`java.util.concurrent.Executors` 가 제공하는 `newFixedThreadPool(size)` 을 사용하면 편리하게 `ExecutorService` 를 생성할 수 있다.

```java
package thread.executor.future;

import java.util.Random;
import java.util.concurrent.*;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class CallableMainV1 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(1);
        Future<Integer> future = es.submit(new MyCallable());
        Integer result = future.get();
        log("result value = " + result);
        es.close();
    }

    static class MyCallable implements Callable<Integer> {

        @Override
        public Integer call() /*throws Exception*/ {
            log("Callable 시작");
            sleep(2000);
            int value = new Random().nextInt(10);
            log("create value = " + value);
            log("Callable 완료");
            return value;
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.21.19.png" alt="" width="563"><figcaption></figcaption></figure>

#### 실행 결과&#x20;

먼저 `MyCallable` 을 구현하는 부분을 보자.

* 숫자를 반환하므로 반활할 제네릭 타입을 `<Integer>` 로 선언했다.
* 구현은 `Runnable` 코드와 비슷한데, 유일한 차이는 결과를 필드에 담아두는 것이 아니라, **결과를 반환한다는 점이다!**

#### submit()

* `MyCallable` 인스턴스가 블로킹 큐에 전달되고, 스레드 풀의 스레드 중 하나가 이 작업을 실행할 것이다.
* 이 때, 작업의 처리 결과는 직접 반환되는 것이 아니라, `Future` 라는 특별한 인터페이스를 통해서 반환된다.
* `future.get()` 을 호출하면 `MyCallable` 의 `call()` 이 반환한 결과를 받을 수 있다.
* 참고로 `Future.get()` 은 `InterruptedException` , `ExecutionException` 체크 예외를 던진다. \
  여기서는 잡지말고 간단하게 밖으로 던지자. 예외에 대한 부분은 뒤에서 설명한다.

#### Executor 프레임워크의 장점

메인 스레드가 결과를 받는 상황으로, `Callable` 을 사용한 방식은 `Runnable` 을 사용하는 방식보다 훨씬 편리하다.

* 코드만 보면 복잡한 멀티스레드를 사용하는 느낌보다는, **단순한 싱글 스레드 방식으로 개발한다는 느낌이 들 것이다.**
* 단순하게 `ExecutorService` 에 필요한 작업을 요청하고 결과를 받아서 사용하면 된다.

~~하지만 편리한 것과 별개로 기반 원리를 잘 이해해야 문제가 없다..~~ \
여기서 잘생각해보면 애매한 부분이 있다.&#x20;

`future.get()` 을 호출하는 요청 스레드(`main`) 는 `future.get()` 을 호출 했을 때 2가지 상황으로 나뉘게 된다.&#x20;

* `MyCallable` 작업을 처리하는 스레드 풀의 스레드가 작업을 완료했다.
* `MyCallable` 작업을 처리하는 스레드 풀의 스레드가 아직 작업을 완료하지 못했다.

`future.get()` 을 호출했을 때 스레드 풀의 스레드가 작업을 완료했다면 반환 받을 결과가 있을 것이다.\
하지만 작업을 처리중이라면 받을 수 있을까?&#x20;

## Future2 - 분석&#x20;

`Future` 는 번역하면 미래라는 뜻이고, 여기서는 미래의 결과를 받을 수 있는 객체라는 뜻이다.\
그렇다면 누구의 미래의 결과를 말하는 것일까?

```java
Future<Integer> future = es.submit(new MyCallable());
```

* `submit()` 의 호출로 `MyCallable` 의 인스턴스를 전달한다.
* 이때, `submit()` 은 `MyCallable.call()` 이 반환하는 무작위 숫자 대신에 `Future` 을 반환한다.
* 생각해보면 `MyCallable` 이 즉시 실행되어 결과를 즉시 반환하는 것은 불가능하다. 왜냐면 `MyCallable` 은 즉시 실행되는 것이 아니다. 스레드 풀의 스레드가 미래의 어떤 시점에 이 코드를 대신 실행해야 한다.
* `MyCallable.call()` 메서드는 호출 스레드(`main` 스레드)가 실행하는 것도 아니고, 스레드 풀의 다른 스레드가 실행하기 때문에, 언제 실행이 완료 되어서 결과를 반환할 지는 알 수 없다.
* 따라서, 결과를 즉시 받는 것은 불가능하다. 이런 이유로, `es.submit()` 은 `MyCallable` 의 결과를 반환하는 대신에 `MyCallable` 의 결과를 나중에 받을 수 있는 `Future` 라는 객체를 대신 제공한다.
* 정리하면, `Future` 에는 전달한 작업(`MyCallable`)의 미래 결과를 담고 있다고 생각하면 된다.

### Future 분석

```java
package thread.executor.future;

import java.util.Random;
import java.util.concurrent.*;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class CallableMainV2 {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ExecutorService es = Executors.newFixedThreadPool(1);
        log("submit() 호출");
        Future<Integer> future = es.submit(new MyCallable());
        log("future 즉시 반환, future = " + future);

        log("future.get() [블로킹] 메서드 호출 시작 -> main 스레드 WAITING");
        Integer result = future.get();
        log("future.get() [블로킹] 메서드 호출 완료 -> main 스레드 RUNNABLE");

        log("result value = " + result);
        log("future 완료, future = " + future);
        es.close();
    }

    static class MyCallable implements Callable<Integer> {

        @Override
        public Integer call() /*throws Exception*/ {
            log("Callable 시작");
            sleep(2000);
            int value = new Random().nextInt(10);
            log("create value = " + value);
            log("Callable 완료");
            return value;
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.34.12.png" alt="" width="563"><figcaption></figcaption></figure>

#### 실행 결과

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.36.19.png" alt="" width="563"><figcaption></figcaption></figure>

* `submit()` 을 호출해서 `ExecutorService` 에 `taskA`(MyCallable 인스턴스)를 전달한다.&#x20;

Feture 생성&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.36.36.png" alt="" width="563"><figcaption></figcaption></figure>

* 요청 스레드(`main` 스레드)는 `es.submit(taskA)` 를 호출하고 있는 중이다.
* `ExecutorService` 는 전달한 `taskA` 의 미래 결과를 알 수 있는 `Future` 객체를 생성한다.
  * `Future` 은 인터페이스다. 이 때 생성되는 실제 구현체는 `FutureTask` 이다.
* 그리고 생성한 `Future` 객체 안에 `taskA` 의 인스턴스를 보관한다.
* `Future` 내부에 `taskA` 작업의 완료 여부와, 작업의 결과 값을 가진다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.39.45.png" alt="" width="563"><figcaption></figcaption></figure>

* `submit()` 을 호출한 경우 `Future` 가 만들어지고, 전달한 작업인 `taskA` 가 바로 블로킹 큐에 담기는 것이 아니라, 그림처럼 `taskA` 를 감싸고 있는 `Future` 가 대신 블로킹 큐에 담긴다.&#x20;
* `Future` 는 내부에 작업의 완료 여부와, 작업의 결과 값을 가진다. 작업이 완료되지 않았기 때문에, 아직은 결과값이 없다.&#x20;
  * 로그를 보면 `Future`의 구현체는 `FutureTask` 이다.&#x20;
  * `Future` 의 상태는 **Not completed** 이고, 연관된 작업은 전달한 `taskA(MyCallable)` 인스턴스 이다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-29 16.24.58.png" alt="" width="563"><figcaption></figcaption></figure>

* **여기서 중요한 핵심이 있는데, 바로 작업을 전달할 때 생성된 `Future` 은 즉시 반환된다는 점이다.**

다음 로그를 보자.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-29 16.26.57.png" alt="" width="563"><figcaption></figcaption></figure>

* 생성한 `Future` 를 즉시 반환하기 때문에, 요청 스레드(`main` 스레드)는 대기하지 않고,\
  자유롭게 다음 코드를 호출할 수 있다.
  * 이것은 마치 `Thread.start()` 를 호출한 것과 비슷하다.
  * `Thread.start()` 를 호출하면 스레드의 작업 코드가 별도의 스레드에서 실행된다.\
    요청 스레드는 대기하지 않고 즉시 다음 코드를 호출할 수 있다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.44.02.png" alt="" width="563"><figcaption></figcaption></figure>

* 큐에 들어있는 `Future[taskA]` 를 꺼내서 스레드 풀의 스레드1이 작업을 시작한다.
  * 참고로 `Future` 의 구현체인 `FutureTask` 는 `Runnable` 인터페이스도 함께 구현하고 있다.
* 스레드1은 `FutureTask` 의 `run()` 메서드를 실행한다.
* 그리고 `run()` 메서드가 `taskA` 의 `call()` 메서드를 호출하고 그 결과를 받아서 처리한다.
  * `FutureTask.run()` -> `MyCallable.call()`

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.46.08.png" alt="" width="563"><figcaption></figcaption></figure>

**스레드1**

* 스레드 1은 `taskA` 의 작업을 아직 처리중이다. 아직 완료하지는 않았다.&#x20;

**요청 스레드 (**`main` **스레드)**

* 요청 스레드는 `Future` 인터스턴스의 참조를 가지고 있다.
* 그리고 언제든지 본인이 필요할 때 `Future.get()` 을 호출해서 `taskA` 작업의 미래 결과를 받을 수 있다.
* 요청 스레드는 작업의 결과가 필요해서 `future.get()` 을 호출한다.
  * `Future` 에는 완료 상태가 있다.
  * `taskA` 의 작업이 완료되면 `Future` 의 상태도 완료로 변경된다.
  * 하지만 `taskA`의 작업이 아직 완료되지 않았다. 그래서 `Future`도 완료 상태가 아니다.
* 때문에, 요청 스레드가 `future.get()` 을 호출하면 `Future`가 완료 상태가 될 때까지 대기한다. (동기)&#x20;
  * 이때, 요청 스레드의 상태는 `RUNNABLE` -> `WAITING` 이다.

**`future.get()` 을 호출했을 때,**

* **`Future` 가 완료 상태**&#x20;
  * `Future`가 완료 상태면 `Future` 에 결과도 포함되어 있다.&#x20;
  * **이 경우 요청 스레드는 대기하지 않고 바로 리턴값을 확인할 수 있다.**&#x20;
* **`Future` 가 완료 상태가 아님**
  * `taskA` 가 아직 수행되지 않았거나 또는 수행 중이라는 뜻이다.
  * 이때는 어쩔 수 없이 요청 스레드가 결과를 받기 위해 대기해야 한다. 요청 스레드가 마치 락을 얻을 때처럼 **결과를 얻기 위해 대기한다. 이처럼 스레드가 어떤 결과를 얻기 위해 대기하는 것을 블로킹(Blocking) 이라 한다.**

> #### 블로킹 메서드&#x20;
>
> `Thread.join()`, `Future.get()` 처럼 메서드가 작업을 바로 수행하지 않고, 다른 작업이 완료될 때까지 \
> 기다리게 하는 메서드이다.&#x20;
>
> 이러한 메서드를 호출하면 호출한 스레드는 지정된 작업이 완료될 때까지 블록(대기) 되어 다른 작업을 진행할 수 \
> 없다.
>
> 대표적으로 `Object.join()`, `Future.get()` 이 있다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.53.55.png" alt="" width="563"><figcaption></figcaption></figure>

요청 스레드 (`main` 스레드)&#x20;

* 대기(`WAITING`) 상태로 `future.get()` 을 호출하고 대기중이다. &#x20;

스레드1

1. `taskA` 작업을 완료한다.
2. `Future` 의 `taskA` 리턴값을 담는다.
3. `Future` 의 상태를 완료로 변경한다.
4. 스레드1이 요청 스레드를 깨운다. 요청 스레드는 `WAITING` -> `RUNNABLE` 상태로 변한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.54.01.png" alt="" width="563"><figcaption></figcaption></figure>

요청 스레드 **(**`main` **스레드)**

* 요청 스레드는 `RUNNABLE` 상태가 되었다. 그리고 완료 상태의 `Future` 를 결과로 받는다.

스레드1

* 작업을 마친 스레드1은 스레드 풀로 반환된다. `RUNNABLE` -> `WAITING`

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 16.56.50.png" alt="" width="563"><figcaption></figcaption></figure>

## Future3 - 활용

### SunTask - Runnable

이번에는 숫자를 더하는 기능을 멀티스레드로 수행해보자.&#x20;

```java
package thread.executor.future;

import static util.MyLogger.log;

public class SumTaskMainV1 {

    public static void main(String[] args) throws InterruptedException {
        SumTask task1 = new SumTask(1, 50);
        SumTask task2 = new SumTask(51, 100);
        Thread thread1 = new Thread(task1, "thread-1");
        Thread thread2 = new Thread(task2, "thread-2");

        thread1.start();
        thread2.start();

        // 스레드가 종료될 때까지 대기
        log("join() - main 스레드가 thread1, thread2 종료까지 대기");
        thread1.join();
        thread2.join();
        log("main 스레드 대기 완료");

        log("task1.result = " + task1.result);
        log("task2.result = " + task2.result);

        int sumAll = task1.result + task2.result;
        log("task1 + task2 = " + sumAll);

        log("end");
    }

    static class SumTask implements Runnable {

        int startValue;
        int endValue;
        int result = 0;

        public SumTask(int startValue, int endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        public void run() {
            log("작업 시작");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            int sum = 0;
            for(int i=startValue; i<=endValue; i++) {
                sum += i;
            }

            result = sum;
            log("작업 완료 result = " + result);
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 17.00.54.png" alt=""><figcaption></figcaption></figure>

### SumTask - Callable&#x20;

위 코드와 비교해서 `ExecutorService` 와 `Callable` 을 사용한 덕분에, 이전 코드보다 훨씬 더 직관적이고 깔끔하게 코드를 작성할 수 있다.&#x20;

**특히 작업의 결과를 반환하고, 요청 스레드에서 그 결과를 바로 받아서 처리하는 부분이 매우 직관적이다.**

* 코드만 보면 싱글 스레드 상황에서 일반적인 메서드를 호출하고 결과를 받는 것 처럼 보여진다.
* 그리고 스레드를 생성하고, `Thread.join()` 과 같은 스레드를 관리하는 코드도 모두 제거할 수 있었다.

```java
package thread.executor.future;

import java.util.concurrent.*;

import static util.MyLogger.log;

public class SumTaskMainV2 {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        SumTask task1 = new SumTask(1, 50);
        SumTask task2 = new SumTask(51, 100);

        ExecutorService es = Executors.newFixedThreadPool(2);

        Future<Integer> future1 = es.submit(task1);
        Future<Integer> future2 = es.submit(task2);

        Integer sum1 = future1.get();
        Integer sum2 = future2.get();

        log("task1.result = " + sum1);
        log("task2.result = " + sum2);

        int sumAll = sum1 + sum2;
        log("task1 + task2 = " + sumAll);
        log("End");

        es.close();
    }

    static class SumTask implements Callable<Integer> {

        int startValue;
        int endValue;

        public SumTask(int startValue, int endValue) {
            this.startValue = startValue;
            this.endValue = endValue;
        }

        @Override
        public Integer call() throws Exception {
            log("작업 시작");
            Thread.sleep(2000);
            int sum = 0;
            for (int i = startValue; i <= endValue; i++) {
                sum += i;
            }

            log("작업 완료 result = " + sum);
            return sum;
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 17.01.48.png" alt=""><figcaption></figcaption></figure>

## Future4 - 이유&#x20;

### Future 가 필요한 이유

#### Future 를 반환하는 코드

```java
Future<Integer> future1 = es.submit(task1);    // 여기는 블로킹 아님 
Future<Integer> future2 = es.submit(task2);    // 여기는 블로킹 아님

Integer sum1 = future1.get();    // 여기서 블로킹 
Integer sum2 = future2.get();    // 여기서 블로킹
```

#### Future 없이 결과를 직접 반환 하는 코드 (가정)

* 참고로 이런 코드는 없다.

```java
Integer sum1 = es.submit(task1); // 여기서 블로킹 
Integer sum2 = es.submit(task2); // 여기서 블로킹
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 17.05.32.png" alt="" width="563"><figcaption></figcaption></figure>

먼저 `ExecutorService` 가 `Future` 없이 결과를 직접 반환한다고 가정해보자.

* 요청 스레드는 `task1` 을 `ExecutorService` 에 요청하고 결과를 기다린다.
  * 작업 스레드가 작업을 수행하는데 2초가 걸린다.&#x20;
  * 요청 스레드는 결과를 받을 때 까지 2초간 대기한다.
  * 요청 스레드는 2초 후에 결과를 받고 라인을 수행한다.
* 요청 스레드는 `task2` 을 `ExecutorService` 에 요청하고 결과를 기다린다.&#x20;
  * 작업 스레드가 작업을 수행하는데 2초가 걸린다.&#x20;
  * 요청 스레드는 결과를 받을 때 까지 2초간 대기한다.&#x20;
  * 결과를 받고 요청 스레드가 다음 라인을 수행한다.&#x20;

**`Future` 를 사용하지 않는 경우 결과적으로 `task1` 의 결과를 기다린 다음에 `task2` 를 요청한다.** \
**따라서 총 4초의 시간이 걸렸다.**&#x20;

**이것은 마치 단일 스레드가 작업을 한 것과 비슷한 결과이다.**&#x20;

#### Future 를 반환 하는 코드

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 17.20.27.png" alt="" width="563"><figcaption></figcaption></figure>

이번에는 `Future` 를 반환한다고 가정해보자.&#x20;

* 요청 스레드는 `task1` 을 `ExecutorService` 에 요청한다.
  * 요청 스레드는 즉시 `Future` 를 반환 받는다.
  * 작업 스레드1은 `task1` 을 수행한다.&#x20;
* 요청 스레드는 `task2` 를 `ExecutorService` 에 요청한다.
  * 요청 스레드는 즉시 `Future` 를 반환 받는다.&#x20;
  * 작업 스레드2 는 `task2` 을 수행한다.&#x20;

**요청 스레드는 `task1`, `task2` 의 작업을 동시에 요청할 수 있다. 따라서 두 작업은 동시에 수행된다.**&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 17.20.38.png" alt="" width="563"><figcaption></figcaption></figure>

**약 2초 뒤 `task1`, `task2` 의 작업이 모두 완료되고 결과를 받는다.**&#x20;

### Future 를 잘못 사용하는 예

#### Future 를 잘못 활용한 예1&#x20;

* 요청 스레드가 작업 하나 요청하고 그 결과를 기다린다. 그리고 그 다음에 다시  다음 요청을 전달하고 결과를 기다린다.&#x20;
* 총 4초의 시간이 걸린다..&#x20;

```java
Future<Integer> future1 = es.submit(task1); // non-blocking 
Integer sum1 = future1.get(); // blocking, 2초 대기

Future<Integer> future2 = es.submit(task2); // non-blocking 
Integer sum2 = future2.get(); // blocking, 2초 대기
```

#### Future 를 잘못 활용한 예2

* 요청 스레드가 작업 하나 요청하고 그 결과를 기다린다. 그리고 그 다음에 다시  다음 요청을 전달하고 결과를 기다린다.&#x20;
* 총 4초의 시간이 걸린다..&#x20;

```java
Integer sum1 = es.submit(task1).get(); // get()에서 블로킹 
Integer sum2 = es.submit(task2).get(); // get()에서 블로킹
```

### 정리&#x20;

* `Future` 라는 개념이 없다면 결과를 받을 때까지 요청 스레드는 아무일도 못하고 대기해야 한다. 따라서 다른 작업을 동시에 수행할 수 없다.&#x20;
* `Future` 라는 개념 덕분에 요청 스레드는 대기하지 않고 다른 작업을 수행할 수 있다. 예를 들어서 다른 작업을 더 요청할 수 있다. 그리고 모든 작업 요청이 끝난 다음에, 본인이 필요할 때 `Future.get()` 을 호출해서 최종 결과를 받을 수 있다.&#x20;
* `Future` 를 사용하는 경우 결과적으로 `task1`, `task2` 를 동시에 요청할 수 있다. 두 작업을 바로 요청했기 때문에 작업을 동시에 제대로 수행할 수 없다.&#x20;

`Future` 는 요청 스레드를 블로킹(대기) 상태로 만들지 않고, 필요한 요청을 모두 수행할 수 있게 해준다. 필요한 모든 요청을 한 다음에 `Future.get()` 을 통해 블로킹 상태로 대기하며 결과를 받으면 된다.

이런 이유로 `ExecutorService` 는 결과를 직접 반환하지 않고, `Future` 를 반환한다.

## Future5 - 정리&#x20;

`Future` 는 작업의 미래 계산의 결과를 나타내며, 계산이 완료되었는지 확인하고, 완료될 때까지 기다릴 수 있는 기능들을 제공한다.

### Future 인터페이스

<pre class="language-java"><code class="lang-java">package java.util.concurrent;
<strong>
</strong><strong>public interface Future&#x3C;V> {
</strong>     boolean cancel(boolean mayInterruptIfRunning);
     boolean isCancelled();
     boolean isDone();
     V get() throws InterruptedException, ExecutionException;
     V get(long timeout, TimeUnit unit)
         throws InterruptedException, ExecutionException, TimeoutException;
     
     enum State {
         RUNNING,
         SUCCESS,
<strong>         FAILED,
</strong>         CANCELLED
<strong>    }
</strong>     default State state() {}
}
</code></pre>

### 주요 메서드&#x20;

**`boolean cancel(boolean mayInterruptIfRunning)`**

* **기능** : 아직 완료되지 않은 작업을 취소한다.
* **매개변수** : **`mayInterruptIfRunning`**
  * `cancel(true)` : `Future` 를 취소 상태로 변경한다. 이때 작업이 실행중이라면 `Thread.interrupt()` 를 호출해서 작업을 중단한다.
  * `cancel(false)` : `Future` 를 취소 상태로 변경한다. 단 이미 실행 중인 작업을 중단하지는 않는다.
* **반환값** : 작업이 성공적으로 취소된 경우 `true` , 이미 완료되었거나 취소할 수 없는 경우 `false`&#x20;
* **설명** : 작업이 실행 중이 아니거나 아직 시작되지 않았으면 취소하고, 실행 중인 작업의 경우`mayInterruptIfRunning` 이 `true` 이면 중단을 시도한다.
* **참고** : 취소 상태의 `Future` 에 `Future.get()` 을 호출하면 `CancellationException` 런타임 예외가 발생 한다.

**`boolean isCancelled()`**

* **기능** : 작업이 취소되었는지 여부를 확인한다.&#x20;
* **반환값**: 작업이 취소된 경우 `true` , 그렇지 않은 경우 `false`&#x20;
* **설명**: 이 메서드는 작업이 `cancel()` 메서드에 의해 취소된 경우에 `true` 를 반환한다.

#### `boolean isDone()`

* **기능** : 작업이 완료되었는지 여부를 확인한다.
* **반환값** : 작업이 완료된 경우 `true` , 그렇지 않은 경우 `false`
* **설명** : 작업이 정상적으로 완료되었거나, 취소되었거나, 예외가 발생하여 종료된 경우에 `true` 를 반환한다.

#### `State state()`

* **기능** : `Future` 의 상태를 반환한다. 자바 19부터 지원한다.
  * `RUNNING` : 작업 실행 중&#x20;
  * `SUCCESS` : 성공 완료&#x20;
  * `FAILED` : 실패 완료&#x20;
  * `CANCELLED` : 취소 완료

#### `V get()`

* **기능** : 작업이 완료될 때까지 대기하고, 완료되면 결과를 반환한다.&#x20;
* **반환값** : 작업의 결과
* **예외**
  * `InterruptedException` : 대기 중에 현재 스레드가 인터럽트된 경우 발생
  * `ExecutionException` : 작업 계산 중에 예외가 발생한 경우 발생
* **설명** : 작업이 완료될 때까지 `get()` 을 호출한 현재 스레드를 대기(블록킹)한다. 작업이 완료되면 결과를 반환한다.

#### `V get(long timeout, TimeUnit unit)`

* 기능 : `get()` 과 같은데, 시간 초과되면 예외를 발생시킨다.
* 매개변수 :
  * `timeout` : 대기할 최대 시간
  * `unit` : `timeout` 매개변수의 시간 단위 지정
* 반환값 : 작업의 결과
* 예외 :&#x20;
  * `InterruptedException` : 대기 중에 현재 스레드가 인터럽트된 경우 발생
  * `ExecutionException` : 계산 중에 예외가 발생한 경우 발생
  * `TimeoutException` : 주어진 시간 내에 작업이 완료되지 않은 경우 발생
* 설명 : 지정된 시간 동안 결과를 기다린다. 시간이 초과되면 `TimeoutException` 을 발생시킨다.

## Future6 - 취소&#x20;

`cancel()` 이 어떻게 동작하는지 알아보자.

```java
package thread.executor.future;

import java.util.concurrent.*;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class FutureCancelMain {

//    private static boolean mayInterruptIfRunning = true;
    private static boolean mayInterruptIfRunning = false;

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(1);
        Future<String> future = es.submit(new MyTask());
        log("Future.state : " + future.state());

        sleep(3000);

        log("future.cancel(" + mayInterruptIfRunning + ") 호출");
        boolean cancelResult = future.cancel(mayInterruptIfRunning);
        log("cancel(" + mayInterruptIfRunning + ") result : " + cancelResult);

        try {
            log("Future result : " + future.get());
        } catch (CancellationException e) {
            log("Future 는 이미 취소 되었습니다.");
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }

        es.close();
    }

    static class MyTask implements Callable<String> {

        @Override
        public String call() {
            try {
                for (int i = 0; i < 10; i++) {
                    log("작업 중 : " + i);
                    Thread.sleep(1000);
                }
            } catch (InterruptedException e) {
                log("인터럽트 발생");
                return "Interrupted";
            }

            return "Completed";
        }
    }
}
```

#### **실행 결과 - mayInterruptIfRunning=true**

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-29 17.26.48.png" alt="" width="375"><figcaption></figcaption></figure>

* `cancel(true)` 를 호출했다.
* `mayInterruptIfRunning=true` 를 사용하면 실행중인 작업에 인터럽트가 발생, 실행중인 작업을 중지 시도한다.
* 이후 `Future.get()`을 호출하면 `CancellationException` 런타임 예외가 발생한다.

#### 실행 결과 - mayInterruptIfRunning=false

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-29 17.25.38.png" alt="" width="375"><figcaption></figcaption></figure>

* `cancel(false)`를 호출했다.
* `mayInterruptIfRunning=false`를 사용하면 실행중인 작업은 그냥 둔다. (인터럽트를 걸지 않는다.)&#x20;
* 실행중인 작업은 그냥 두더라도 `cancel()` 을 호출했기 때문에 `Future` 는 `CANCEL` 상태가 된다.
* 이후 `Future.get()` 을 호출하면 `CancellationException` 런타임 예외가 발생한다.

## Future7 - 예외

`Future.get()` 을 호출하면 작업의 결과 뿐 아니라, 작업 중 발생한 예외도 받을 수 있다.&#x20;

아래 코드를 살펴보자.&#x20;

* 요청 스레드 : `es.submit(new ExCallable())` 을 호출해서 작업을 전달한다.
* 작업 스레드 : `ExCallable` 을 실행하는데, `IllegalStatementException` 예외가 발생한다.&#x20;
  * 작업 스레드는 `Future` 에 발생한 예외를 담아둔다. 참고로 예외도 객체이다. 잡아서 필드에 보관할 수 있다.&#x20;
  * 예외가 발생했으므로, `Future` 의 상태는 `FAILED` 가 된다.&#x20;
* 요청 스레드 : 결과를 얻기 위해 `future.get()` 을 호출한다.
  * `Future` 의 상태가 `FAILED` 면 `ExecutionException` 예외를 던진다.&#x20;
  * 이 예외는 내부에 앞서 `Future` 에 저장해둔 `IllegalStatementException` 을 포함하고 있다.&#x20;
  * `e.getCause()` 을 호출하면 작업에서 발생한 원본 예외를 받을 수 있다.

```java
 package thread.executor.future;

import java.util.concurrent.*;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class FutureExceptionMain {

    public static void main(String[] args) {
        ExecutorService es = Executors.newFixedThreadPool(1);
        log("작업 전달");
        Future<Integer> future = es.submit(new ExCallable());
        sleep(1000);

        try {
            log("future.get() 호출 시도, future.state() : " + future.state());
            Integer result = future.get();
            log("result value = " + result);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        } catch (ExecutionException e) {
            log("e = " + e);
            Throwable cause = e.getCause();
            log("cause = " + cause);
        }

        es.close();
    }

    static class ExCallable implements Callable<Integer> {

        @Override
        public Integer call() throws Exception {
            log("Callable 실행, 예외 발생");
            throw new IllegalStateException("ex!");
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-17 18.29.28.png" alt="" width="563"><figcaption></figcaption></figure>

`Future.get()` 은 작업의 결과 값을 받을 수도 있고, 예외를 받을 수도 있다. 마치 싱글 스레드 상황에서 일반적인 메서드를 호출하는 것과 같다. `Executor` 프레임워크가 얼마나 잘 설계되어 있는지 알 수 있는 부분이다.

## ExecutorService - 작업 컬렉션 처리

`ExecutorService` 는 여러 작업을 한 번에 편리하게 처리하는 `invokeAll()` , `invokeAny()` 기능을 제공한다.

#### 작업 컬렉션 처리&#x20;

#### **invokeAll()**

* `<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks) throws InterruptedException`
  * 모든 `Callable` 작업을 제출하고, 모든 작업이 완료될 때까지 기다린다.
* `<T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException`&#x20;
  * 지정된 시간 내에 모든 `Callable` 작업을 제출하고 완료될 때까지 기다린다.

```java
package thread.executor;

import java.util.concurrent.Callable;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class CallableTask implements Callable<Integer> {

    private final String name;
    private int sleepMs = 1000;

    public CallableTask(String name) {
        this.name = name;
    }

    public CallableTask(String name, int sleepMs) {
        this.name = name;
        this.sleepMs = sleepMs;
    }

    @Override
    public Integer call() throws Exception {
        log(name + "실행");
        sleep(sleepMs);
        log(name + "완료");
        return sleepMs;
    }
}
```

```java
package thread.executor.future;

import thread.executor.CallableTask;

import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

import static util.MyLogger.log;

public class InvokeAllMain {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService es = Executors.newFixedThreadPool(10);

        CallableTask task1 = new CallableTask("task1", 1000);
        CallableTask task2 = new CallableTask("task2", 2000);
        CallableTask task3 = new CallableTask("task3", 3000);

        List<CallableTask> tasks = List.of(task1, task2, task3);

        List<Future<Integer>> futures = es.invokeAll(tasks);
        for (Future<Integer> future : futures) {
            Integer value = future.get();
            log("value = " + value);
        }
        es.close();
    }
}
```

**invokeAny()**

* `<T> T invokeAny(Collection<? extends Callable<T>> tasks) throws InterruptedException, ExecutionException`
  * 하나의 `Callable` 작업이 완료될 때까지 기다리고, 가장 먼저 완료된 작업의 결과를 반환한다.
  * 완료되지 않은 나머지 작업은 취소한다.
* `<T> T invokeAny(Collection<? extends Callable<T>> tasks, long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException`&#x20;
  * 지정된 시간 내에 하나의 `Callable` 작업이 완료될 때까지 기다리고, 가장 먼저 완료된 작업의 결과를 반환한다.
  * 완료되지 않은 나머지 작업은 취소한다.

```java
package thread.executor.future;

import thread.executor.CallableTask;

import java.util.List;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

import static util.MyLogger.log;

public class InvokeAnyMain {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService es = Executors.newFixedThreadPool(10);

        CallableTask task1 = new CallableTask("task1", 1000);
        CallableTask task2 = new CallableTask("task2", 2000);
        CallableTask task3 = new CallableTask("task3", 3000);

        List<CallableTask> tasks = List.of(task1, task2, task3);

        Integer value = es.invokeAny(tasks);
        log("value = " + value);
        es.close();
    }
}
```
