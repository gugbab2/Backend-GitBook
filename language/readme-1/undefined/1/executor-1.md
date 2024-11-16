# 스레드 풀과 Executor 프레임워크1

## 스레드를 직접 사용할 때 문제점&#x20;

* 실무에서 스레드를 직접 생성해서 사용하면 다음과 같은 3가지 문제가 있다.
  * 스레드 생성 시간으로 인한 성능 문제
  * 스레드 관리 문제
  * `Runnable` 인터페이스의 불편함

#### 1. 스레드 생성 시간으로 인한 성능 문제

* 스레드를 사용하려면 먼저 스레드를 생성해야 한다. 그런데 스레드는 다음과 같은 이유로 매우 무겁다.
  * **메모리 할당** : 각 스레드는 자신만의 호출 스택(call stack)을 가지고 있어야 한다. 이 호출 스택은 스레드가 실행되 는 동안 사용하는 메모리 공간이다. 따라서 스레드를 생성할 때는 이 호출 스택을 위한 메모리를 할당해야 한다.&#x20;
  * **운영체제 자원 사용** : 스레드를 생성하는 작업은 운영체제 커널 수준에서 이루어지며, 시스템 콜(system call)을 통해 처리된다. 이는 CPU와 메모리 리소스를 소모하는 작업이다.
  * **운영체제 스케줄러 설정** : 새로운 스레드가 생성되면 운영체제의 스케줄러는 이 스레드를 관리하고 실행 순서를 조 정해야 한다. 이는 운영체제의 스케줄링 알고리즘에 따라 추가적인 오버헤드가 발생할 수 있다.
  * **참고로 스레드 하나는 보통 1MB 이상의 메모리를 사용한다.**
* 스레드를 생성하는 작업은 상대적으로 무겁다. 단순히 자바 객체를 하나 생성하는 것과는 비교할 수 없을 정도로 큰 작 업이다.
  * 예를 들어서 어떤 작업 하나를 수행할 때 마다 스레드를 각각 생성하고 실행한다면, 스레드의 생성 비용 때문에, 이미 많은 시간이 소모된다. 아주 가벼운 작업이라면, 작업의 실행 시간보다 스레드의 생성 시간이 더 오래 걸릴 수도 있다.
* 이런 문제를 해결하려면 생성한 스레드를 재사용하는 방법을 고려할 수 있다. 스레드를 재사용하면 처음 생성할 때를 제외하고는 생성을 위한 시간이 들지 않는다. 따라서 스레드가 아주 빠르게 작업을 수행할 수 있다.

#### 2. 스레드 관리 문제

* 서버의 CPU, 메모리 자원은 한정되어 있기 때문에, 스레드는 무한하게 만들 수 없다.
  * 예를 들어서, 사용자의 주문을 처리하는 서비스라고 가정하자. 그리고 사용자의 주문이 들어올 때 마다 스레드를 만들어서 요청을 처리한다고 가정하겠다.
  * 서비스 마케팅을 위해 선착순 할인 이벤트를 진행한다고 가정해보자. 그러면 사용자 가 갑자기 몰려들 수 있다. 평소 동시에 100개 정도의 스레드면 충분했는데, 갑자기 10000개의 스레드가 필요한 상황 이 된다면 CPU, 메모리 자원이 버티지 못할 것이다.
  * **이런 문제를 해결하려면 우리 시스템이 버틸 수 있는, 최대 스레드의 수 까지만 스레드를 생성할 수 있게 관리해야 한다.**
* 또한 이런 문제도 있다. 예를 들어 애플리케이션을 종료한다고 가정해보자.
  * 이때 안전한 종료를 위해 실행 중인 스레드가 남은 작업은 모두 수행한 다음에 프로그램을 종료하고 싶다거나,&#x20;
  * 또는 급하게 종료해야 해서 인터럽트 등의 신호를 주고 스레드를 종료하고 싶다고 가정해보자.
  * 이런 경우에도 스레드가 어딘가에 관리가 되어 있어야한다.

#### 3. `Runnable` 인터페이스의 불편함

```java
public interface Runnable {
     void run();
}
```

* `Runnable` 인터페이스는 다음과 같은 이유로 불편하다.
  * **반환 값이 없다** : `run()` 메서드는 반환 값을 가지지 않는다. 따라서 실행 결과를 얻기 위해서는 별도의 메커니즘 을 사용해야 한다. 쉽게 이야기해서 스레드의 실행 결과를 직접 받을 수 없다.&#x20;
    * 앞에서 공부한 `SumTask` 의 예를 생각해보자. 스레드가 실행한 결과를 멤버 변수에 넣어두고, `join()` 등을 사용해서 스레드가 종료되길 기다린 다음에 멤버 변수에 보관한 값을 받아야 한다.
  * **예외 처리** : `run()` 메서드는 체크 예외(checked exception)를 던질 수 없다. 체크 예외의 처리는 메서드 내부 에서 처리해야 한다.
* 이런 문제를 해결하려면 반환 값도 받을 수 있고, 예외도 좀 더 쉽게 처리할 수 있는 방법이 필요하다. 추가로 반환 값 뿐만 아니라 해당 스레드에서 발생한 예외도 받을 수 있다면 더 좋을 것이다.

#### 해결&#x20;

* 위에서 말한 1,2 번의 문제를 해결하기 위해서는 스레드 풀(Thread Pool) 이 필요하다.&#x20;
* 이렇게 스레드 풀이라는 개념을 사용하면 스레드를 재사용할 수 있어서, 재사용시 스레드의 생성 시간을 절약할 수 있 다. 그리고 스레드 풀에서 스레드가 관리되기 때문에 필요한 만큼만 스레드를 만들 수 있고, 또 관리할 수 있다.
* 사실 스레드 풀이라는 것이 별것이 아니다. 그냥 컬렉션에 스레드를 보관하고 재사용할 수 있게 하면 된다. 하지만 스레 드 풀에 있는 스레드는 처리할 작업이 없다면, 대기( `WAITING`) 상태로 관리해야 하고, 작업 요청이 오면 `RUNNABLE` 상태로 변경해야 한다.&#x20;
* 막상 구현하려고 하면 생각보다 매우 복잡하다는 사실을 알게될 것이다. 여기에 생산자 소비자 문제까지 겹친다.&#x20;
  * 잘 생각해보면 어떤 생산자가 작업(task)를 만들 것이고, 우리의 스레드 풀에 있는 스레드가 소비자가 되는 것이다.
* 이런 문제를 한방에 해결해주는 것이 바로 자바가 제공하는 `Executor` 프레임워크다.
  * `Executor` 프레임워크는 스레드 풀, 스레드 관리, `Runnable` 의 문제점은 물론이고, 생산자 소비자 문제까지 한방에 해결해주는 자바 멀티스레드 최고의 도구이다.&#x20;
  * 지금까지 우리가 배운 멀티스레드 기술의 총 집합이 여기에 들어있다.
* 참고로 앞서 설명한 이유와 같이 스레드를 사용할 때는 생각보다 고려해야 할 일이 많다. 그래서 실무에서는 스레드를 직접 하나하나 생성해서 사용하는 일이 드물다. 대신에 지금부터 설명할 `Executor` 프레임워크를 주로 사용하는데, 이 기술을 사용하면 매우 편리하게 멀티스레드 프로그래밍을 할 수 있다.

## Executor 프레임워크 소개&#x20;

### Executor 프레임워크의 주요 구성 요소

#### Executor 인터페이스

* 가장 간단한 인터페이스로 작업을 실행하는 책임을 가지고 있다.&#x20;

```java
package java.util.concurrent;
public interface Executor {
     void execute(Runnable command);
}
```

#### ExecutorService 인터페이스 - 주요 메서드

* `Executor` 인터페이스를 확장해서 작업 제출과 제어 기능을 추가로 제공한다.&#x20;
* 주요 메서드로는 `submit()` , `close()` 가 있다.&#x20;
* 더 많은 기능이 있지만 나머지 기능들은 뒤에서 알아보자.&#x20;
* `Executor` 프레임워크를 사용할 때는 대부분 이 인터페이스를 사용한다.

```java
public interface ExecutorService extends Executor, AutoCloseable {
     <T> Future<T> submit(Callable<T> task);
     @Override
     default void close(){...}
     
     ...
}
```

### 로그 출력 유틸리티 만들기&#x20;

* `ExecutorService` 의 기본 구현체는 `ThreadPoolExecutor` 이다.&#x20;
  * `pool` : 스레드 풀에서 관리되는 스레드의 숫자
  * `active` : 작업을 수행하는 스레드의 숫자
  * `queuedTasks` : 큐에 대기중인 작업의 숫자
  * `completedTask` : 완료된 작업의 숫자
* `ExecutorService` 인터페이스는 `getPoolSize()`, `getActiveCount()` 와 같은 메서드를 제공하지 않는다. 이 기능은 `ExecutorService` 의 대표 구현체인 `ThreadPoolExecutor` 를 사용해야 한다.

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

* `ExecutorService` 가장 대표적인 구현체는 `ThreadPoolExecutor` 이다.&#x20;
* `ThreadPoolExecutor(ExecutorService)` 는 두가지 요소로 구성되어 있다.&#x20;
  * 스레드 풀 : 스레드를 관리한다.&#x20;
  * `BlockingQueue` : 작업을 보관한다. 생산자 소비자 문제를 실행하기 위해서 단순한 큐가 아니라 `BlockingQueue` 를 사용한다.&#x20;
* 생산자가 `es.execute(new RunnableTask("taskA"))` 를 호출하면, `RunnableTask("taskA")` 인스턴스가 `BlockingQueue` 에 보관된다.
  * **생산자** : `es.execute(작업)` 를 호출하면 내부에서 `BlockingQueue` 에 작업을 보관한다. `main` 스레드가 생산된다.&#x20;
  * **소비자** : 스레드 풀에 있는 스레드가 소비자이다. 이후에 소비자 중에 하나가 `BlockingQueue` 에 들어있는 작업 을 받아서 처리한다.

#### **ThreadPoolExecutor 생성자**

* `ThreadPoolExecutor` 의 생성자는 다음 속성을 사용한다.
  * `corePoolSize` : 스레드 풀에서 관리되는 기본 스레드의 수
  * `maximumPoolSize` : 스레드 풀에서 관리되는 최대 스레드 수
  * `keepAliveTime`, `TimeUnit unit` : 기본 스레드 수를 초과해서 만들어진 스레드가 생존할 수 있는 대기 시간이다. 이 시간 동안 처리할 작업이 없다면 초과 스레드는 제거된다.&#x20;
  * `BlockingQueue workQueue` : 작업을 보관할 블로킹 큐
* `new ThreadPoolExecutor(2,2,0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>());`
  * 최대 스레드 수와 `keepAliveTime` , `TimeUnit` unit 에 대한 부분은 뒤에서 따로 설명하겠다.
  * 여기서는 `corePoolSize=2`, `maximumPoolSize=2` 를 사용해서 기본 스레드와 최대 스레드 수를 맞추었다. 따라서 풀에서 관리되는 스레드는 2개로 고정된다.&#x20;
  * `keepAliveTime`, `TimeUnit unit` 는 0으로 설정했는 데, 이 부분은 뒤에서 설명한다.
  * 작업을 보관할 블로킹 큐의 구현체로 `LinkedBlockingQueue` 를 사용했다. 참고로 이 블로킹 큐는 작업을 무한대로 저장할 수 있다.

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

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 15.46.04.png" alt=""><figcaption></figcaption></figure>

### ExecutorService 분석

* `ThreadPoolExecutor` 를 생성한 시점에는 스레드 풀에 스레드를 미리 만들어두지 않는다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.12.png" alt="" width="563"><figcaption></figcaption></figure>

* `main` 스레드가 `es.execute("taskA ~ taskD")` 를 호출한다.
  *   참고로 당연한 이야기지만 `main` 스레드는 작업을 전달하고 기다리지 않는다. 전달한 작업은 다른 스레드

      가 실행할 것이다. `main` 스레드는 작업을 큐에 보관까지만 하고 바로 다음 코드를 수행한다.&#x20;
* `taskA~D` 요청이 블로킹 큐에 들어온다.
* 최초의 작업이 들어오면 이때 작업을 처리하기 위해 스레드를 만든다.&#x20;
  * 참고로 스레드 풀에 스레드를 미리 만들어두지는 않는다.
* 작업이 들어올 때 마다 `corePoolSize`의 크기 까지 스레드를 만든다.
  * 예를 들어서 최초 작업인 `taskA` 가 들어오는 시점에 스레드1을 생성하고, 다음 작업인 `taskB` 가 들어오 는 시점에 스레드2를 생성한다.
  * 이런 방식으로 `corePoolSize` 에 지정한 수 만큼 스레드를 스레드 풀에 만든다. 여기서는 2를 설정했으므 로 2개까지 만든다.
  * `corePoolSize` 까지 스레드가 생성되고 나면, 이후에는 스레드를 생성하지 않고 앞서 만든 스레드를 재 사용한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.25.png" alt="" width="563"><figcaption></figcaption></figure>

* 스레드 풀에 관리되는 스레드가 2개이므로 `pool=2`&#x20;
* 작업을 수행중인 스레드가 2개이므로 `active=2`&#x20;
* 큐에 대기중인 작업이 2개이므로 `queuedTasks=2`&#x20;
* 완료된 작업은 없으므로 `completedTasks=0`

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.01.43.png" alt="" width="563"><figcaption></figcaption></figure>

* 작업이 완료되면 스레드 풀에 스레드를 반납한다. **스레드를 반납하면 스레드는 대기( `WAITING` ) 상태로 스레드 풀에 대기한다.**
  * 참고로 실제 반납 되는게 아니라, 스레드의 상태가 변경된다고 이해하면 된다.
* 반납된 스레드는 재사용된다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.11.45.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.12.46.png" alt="" width="563"><figcaption></figcaption></figure>

* `taskC` , `taskD` 의 작업을 처리하기 위해 스레드 풀에서 스레드를 꺼내 재사용한다.
* 작업이 완료되면 스레드는 다시 스레드 풀에서 대기한다.
* `close()` 를 호출하면 `ThreadPoolExecutor` 가 종료된다. 이때 스레드 풀에 대기하는 스레드도 함께 제거된다.

> `close()` 는 자바 19부터 지원되는 메서드이다. 만약 19 미만 버전을 사용한다면 `shutdown()` 을 호출
>
> 하자. 둘의 차이는 뒤에서 설명한다.



<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.15.38.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.15.55.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-16 16.16.24.png" alt="" width="563"><figcaption></figcaption></figure>

## Runnable 의 불편함&#x20;

* 위에서 Runnable 인터페이스는 다음과 같은 불편함이 있다고 했다.&#x20;
  * **반환값이 없다** : `run()` 메서드는 반환 값을 가지지 않는다. 따라서 실행 결과를 얻기 위해서는 별도의 메커니즘 을 사용해야 한다.&#x20;
    * 쉽게 이야기해서 스레드의 실행 결과를 직접 받을 수 없다.&#x20;
    * 앞에서 공부한 `SumTask`의 예를 생각해보자. 스레드가 실행한 결과를 멤버 변수에 넣어두고, `join()` 등을 사용해서 스레드가 종료되길 기다린 다음에 멤버 변수를 통해 값을 받아야 한다.
  * **예외 처리** : `run()` 메서드는 체크 예외(checked exception)를 던질 수 없다. 체크 예외의 처리는 메서드 내부 에서 처리해야 한다.
