# 병렬 스트림

## 단일 스트림&#x20;

자바 병렬 스트림을 제대로 이해하려면, 우리가 앞서 학습한 스트림은 물론이고, 멀티스레드, Fork/Join 프레임워크에 대한 기본 지식이 필요하다.&#x20;

### 병렬 스트림 준비 예제&#x20;

병렬 스트림을 학습하기 전에, 로깅 유틸리티와 무거운 작업을 시뮬레이션하는 클래스를 준비하자.&#x20;

```java
package util;

import java.time.LocalTime;
import java.time.format.DateTimeFormatter;


public abstract class MyLogger {

    private static final DateTimeFormatter formatter = DateTimeFormatter.ofPattern("HH:mm:ss.SSS");

    public static void log(Object obj) {
        String time = LocalTime.now().format(formatter);
        System.out.printf("%s [%9s] %s\n", time, Thread.currentThread().getName(), obj);
    }
}
```

```java
package parallel;

import util.MyLogger;

public class HeavyJob {

    public static int heavyTask(int i) {
        MyLogger.log("calculate " + i + " -> " + i * 10);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return i * 10;
    }

    public static int heavyTask(int i, String name) {
        MyLogger.log("[" + name + "] " + i + " -> " + i * 10);
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        return i * 10;
    }
}
```

* `HeavyJob` 클래스는 오래 걸리는 작업을 시뮬레이션하는데, 각 작업은 1초 정도 소요된다고 가정하겠다. \
  입력값에 10을 곱한 결과를 반환하며, 작업이 실행될 때마다 로그를 출력한다.

### 예제1 - 단일 스트림

먼저 단일 스트림으로 `IntStream.rangeClosed(1, 8)` 에서 나온 1\~8까지의 숫자 각각에 대해 `heavyTask()` 를 순서대로 수행해보자.&#x20;

```java
package parallel;

import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ParallelMain1 {

    public static void main(String[] args) {
        long startTime = System.currentTimeMillis();

        int sum = IntStream.rangeClosed(1, 8)
                .map(HeavyJob::heavyTask)
                .reduce(0, (a, b) -> a + b);

        long endTime = System.currentTimeMillis();
        log("time: " + (endTime - startTime) + "ms, sum: " + sum);
    }
}
```

* `map(HeavyJob::heavyTask)` 로 1초씩 걸리는 작업을 8번 순차로 호출하므로, 약 8초가 소요된다.
* 마지막에 `reduce(0, (a, b) -> a + b)`, 또는 `sum()` 으로 최종 결과를 합산한다.
*   결과적으로 **단일 스레드**(`main` 스레드)에서 작업을 순차적으로 수행하기 때문에 로그에도 `[main]` 스레드만

    표시된다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.09.03.png" alt="" width="563"><figcaption></figcaption></figure>

* 전체 시간이 8초 정도 걸리는 것을 확인할 수 있다.&#x20;

8초는 너무 오래 걸린다.. 스레드를 사용해서 실행 시간을 단축해보자.&#x20;

## 스레드 직접 사용&#x20;

### 예제2 - 스레드 직접 사용&#x20;

```java
package parallel;

import util.MyLogger;

import java.util.stream.IntStream;

import static util.MyLogger.*;
import static util.MyLogger.log;

public class ParallelMain2 {

    public static void main(String[] args) throws InterruptedException {
        long startTime = System.currentTimeMillis();

        // 1. Fork 작업을 분할한다.
        SumTask task1 = new SumTask(1, 4);
        SumTask task2 = new SumTask(5, 8);

        Thread thread1 = new Thread(task1, "thread-1");
        Thread thread2 = new Thread(task2, "thread-2");

        // 2. 분할한 작업을 처리한다.
        thread1.start();
        thread2.start();

        // 3. join - 처리한 결과를 합친다.
        thread1.join();
        thread2.join();
        log("main 스레드 대기 완료");

        int sum = task1.result + task2.result;

        long endTime = System.currentTimeMillis();
        log("time: " + (endTime - startTime) + "ms, sum: " + sum);
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
            int sum = 0;
            for (int i = startValue; i <= endValue ; i++) {
                int calculated = HeavyJob.heavyTask(i);
                sum += calculated;
            }
            result = sum;
            log("작업 완료 result = " + result);
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.10.36.png" alt="" width="563"><figcaption></figcaption></figure>

* 2개의 스레드가 작업을 분할해서 처리했기 때문에, 8초의 작업 시작을 4초로 줄일 수 있었다.&#x20;
* 하지만, 이렇게 스레드를 직접 관리하면 스레드 수가 늘어날 때 코드가 복잡해지고, 예외 처리, 스레드 풀 관리 등 추가 관리 포인트가 생기는 문제가 있다.&#x20;

## 스레드 풀 사용&#x20;

이번에는 자바가 제공하는 `ExecutorService` 를 사용해서 더 편리하게 병렬 처리를 해보자.&#x20;

### 예제3 - 스레드 풀&#x20;

```java
package parallel;

import java.util.concurrent.*;

import static util.MyLogger.log;

public class ParallelMain3 {

    public static void main(String[] args) throws InterruptedException, ExecutionException {
        long startTime = System.currentTimeMillis();

        // 스레드 풀을 준비한다
        ExecutorService es = Executors.newFixedThreadPool(2);

        // 1. fork - 작업을 분할한다.
        SumTask task1 = new SumTask(1, 4);
        SumTask task2 = new SumTask(5, 8);

        // 2. execute - 분할한 작업을 처리한다.
        Future<?> future1 = es.submit(task1);
        Future<?> future2 = es.submit(task2);

        // 3. join - 처리한 결과를 합친다. get: 결과가 나올 때 까지 대기한다.(블로킹 메서드)
        Integer result1 = (Integer) future1.get();
        Integer result2 = (Integer) future2.get();
        log("main 스레드 대기 완료");

        int sum = result1 + result2;
        long endTime = System.currentTimeMillis();
        log("time: " + (endTime - startTime) + "ms, sum: " + sum);

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
            int sum = 0;
            for (int i = startValue; i <= endValue ; i++) {
                int calculated = HeavyJob.heavyTask(i);
                sum += calculated;
            }
            log("작업 완료 result = " + sum);
            return sum;
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.13.17.png" alt="" width="563"><figcaption></figcaption></figure>

* 예제2 처럼 스레드가 2개이므로 각각 4개씩 나눠 처리한다.&#x20;
* `Future` 로 반환값을 쉽게 받아올 수 있기 때문에(`Executor` 프레임워크 덕분에 싱글 스레드를 사용하는 듯하다), \
  결과값을 합산하는 과정에서 더 편리해졌다.&#x20;
* 하지만 여전히 코드 레벨에서 분할/병합 로직을 직접 처리해야 하고, 스레드 풀 생성과 관리도 개발자가 직접 해야한다.&#x20;

## Fork/Join 패턴&#x20;

### 분할(Fork), 처리(Execute), 모음(Join)&#x20;

스레드는 한 번에 하나의 작업을 처리할 수 있다. 따라서 하나의 큰 작업을 여러 스레드가 처리할 수 있는 작은 단위의 작업으로 분할(Fork) 해야 한다. 그리고 이렇게 분할한 작업을 각각의 스레드가 처리(Execute) 하는 것이다. 각 스레드의 분할된 작업 처리가 끝나면 분할된 결과를 하나로 모아야(Join) 한다.

이렇게 **분할(Fork) 처리(Execute) 모음(Join)**&#xC758; 단계로 이루어진 멀티스레딩 패턴을 **Fork/Join 패턴**이라고 부른다. \
이 패턴은 병렬 프로그래밍에서 매우 효율적인 방식으로, 복잡한 작업을 병렬적으로 처리할 수 있게 해준다.

지금까지 우리는 이런 과정을 다음과 같이 직접 처리했다. 우리가 진행했던 예제들을 그림과 함께 다시 정리해보자.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.19.41.png" alt=""><figcaption></figcaption></figure>

자바는 Fork/Join 프레임워크를 제공해서 개발자가 이러한 패턴을 더 쉽게 구현할 수 있도록 지원한다.&#x20;

## Fork/Join 프레임워크1 - 소개&#x20;

### Fork/Join 프레임워크 소개&#x20;

자바의 Fork/Join 프레임워크는 자바 7부터 도입된 `java.util.concurrent` 패키지의 일부로, 멀티코어 프로세서를 효율적으로 활용하기 위한 병렬 처리 프레임워크이다. 주요 개념은 다음과 같다.

#### 분할 정복(Divied and Conquer) 전략&#x20;

* 큰 작업을 작은 단위로 재귀적으로 분할(Fork)&#x20;
* 각 작은 작업의 결과를 합쳐(Join) 최종 결과를 생성&#x20;
* 멀티코어 환경에서 작업을 효율적으로 분산 처리&#x20;

#### 작업 훔치기(Work Stealing) 알고리즘&#x20;

* 각 스레드는 자신의 작업 큐를 가짐
* 작업이 없는 스레드는 다른 바쁜 스레드의 큐에서 작업을 "훔쳐와서" 대신 처리
* 부하 균형을 자동으로 조절하여 효율성 향상

### 주요 클래스&#x20;

Fork/Join 프레임워크를 이루는 주요 클래스는 다음과 같다.&#x20;

* `ForkJoinPool`&#x20;
* `ForkJoinTask`&#x20;
  * `RecursiveTask`&#x20;
  * `RecursiveAction`

#### ForkJoinPool

* Fork/Join 작업을 실행하는 특수한 `ExecutorService` 스레드 풀
* 작업 스케줄링 및 스레드 관리를 담당
* 기본적으로 사용 가능한 프로세서 수 만큼의 스레드 생성&#x20;
  * 예) CPU 코어가 10 코어면 10개의 스레드 생성
* 쉽게 이야기해서 **분할 정복과 작업 훔치기에 특화된 스레드 풀**이다.

```java
// 기본 풀 생성 (프로세서 수에 맞춰 스레드 생성)
ForkJoinPool pool = new ForkJoinPool();

// 특정 병렬 수준으로 풀 생성
ForkJoinPool customPool = new ForkJoinPool(4);
```

#### ForkJoinTask

* `ForkJoinTask` 는 Fork/Join 작업의 기본 추상 클래스이다.&#x20;
* `Future` 를 구현했다.&#x20;
* 개발자는 주로 다음 두 하위 클래스를 구현해서 사용한다.
  * `RecursiveTask<V>` : 결과를 반환하는 작업
  * `RecursiveAction` : 결과를 반환하지 않는 작업(`void` )

#### **RecursiveTask / RecursiveAction의 구현 방법**

* `compute()` 메서드를 재정의해서 필요한 작업 로직을 작성한다.
* 일반적으로 일정 기준(임계값)을 두고, **작업 범위가 작으면 직접 처리**하고, **크면 작업을 둘로 분할**하여 각각 병렬로 처리하도록 구현한다.

#### **fork() / join() 메서드**

* `fork()` : 현재 스레드에서 다른 스레드로 작업을 **분할**하여 보내는 동작(비동기 실행)
* `join()` : 분할된 작업이 끝날 때까지 기다린 후 결과를 가져오는 동작

> **참고**: Fork/Join 프레임워크를 실무에서 직접적으로 다루는 일은 드물다. 따라서 이런게 있다 정도만 알아두고 넘어가자.
>
> 개념 정도만 대략 알아두면 충분하다.

### Fork/Join 프레임워크 활용

실제 Fork/Join 프레임워크를 사용해서 우리가 앞서 처리한 예시를 개발해보자.

기본적인 처리 방식은 다음과 같다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.27.46.png" alt=""><figcaption></figcaption></figure>

핵심은 작업의 크기가 임계값 보다 크면 분할하고, 임계값 보다 같거나 작으면 직접 처리하는 것이다.

예를 들어, 작업의 크기가 8이고, 임계값이 4라고 가정해보자.

1. **Fork**: 작업의 크기가 8이면 임계값을 넘었다. 따라서 작업을 절반으로 분할한다.
2. **Execute**: 다음으로 작업의 크기가 4라면 임계값의 범위 안에 들어오므로 작업을 분할하지 않고, 처리한다.
3. **Join**: 최종 결과를 합친다.

Fork/Join 프레임워크를 사용하려면 `RecursiveTask.compute()` 메서드를 재정의해야 한다.

다음에 작성한 `SumTask` 는 `RecursiveTask<Integer>` 를 상속받아 리스트의 합을 계산하는 작업을 병렬로 처리하는 클래스이다. 이 클래스는 Fork/Join 프레임워크의 **분할 정복** 전략을 구현한다.

```java
package parallel.forkjoin;

import parallel.HeavyJob;

import java.util.List;
import java.util.concurrent.RecursiveTask;

import static util.MyLogger.*;

public class SumTask extends RecursiveTask<Integer> {
    private static final int THRESHOLD = 4; // 임계값

    private final List<Integer> list;

    public SumTask(List<Integer> list) {
        this.list = list;
    }

    @Override
    protected Integer compute() {
        // 작업 범위가 작으면 직접 계산
        if (list.size() <= THRESHOLD) {
            log("[처리 시작] " + list);
            int sum = list.stream()
                    .mapToInt(HeavyJob::heavyTask)
                    .sum();
            log("[처리 완료] " + list + " -> sum : " + sum);
            return sum;
        } else {
            // 작업 범위가 크면 반으로 나누어 병렬 처리
            int mid = list.size() / 2;
            List<Integer> leftList = list.subList(0, mid);
            List<Integer> rightList = list.subList(mid, list.size());
            log("[분할] " + list + " -> LEFT" + leftList + ", RIGHT" + rightList);

            SumTask leftTask = new SumTask(leftList);
            SumTask rightTask = new SumTask(rightList);

            // 왼쪽 작업은 다른 스레드에서 처리
            leftTask.fork();    // 1~4
            // 오른쪽 작업은 현재 스레드에서 처리
            Integer rightResult = rightTask.compute();  // 5~8

            // 왼쪽 작업 결과를 기다림
            Integer leftResult = leftTask.join();
            int joinSum = leftResult + rightResult;
            log("LEFT[" + leftResult + "] + RIGHT[" + rightTask + "] -> sum:" + joinSum);
            return joinSum;
        }
    }
}
```

*   **THRESHOLD (임계값)**: 작업을 더 이상 분할하지 않고 직접 처리할 리스트의 크기를 정의한다. 여기서는 4로

    설정되어, 리스트 크기가 4 이하일 때 직접 계산한다. 4보다 크면 작업을 분할한다.
* **작업 분할**: 리스트의 크기가 임계값보다 크면, 리스트를 반으로 나누어 `leftList` 와 `rightList` 로 분할한다.
* **fork(), compute()**
  * `fork()` 는 왼쪽 작업을 다른 스레드에 위임하여 병렬로 처리한다.
  * `compute()` 는 오른쪽 작업을 현재 스레드에서 직접 수행한다(재귀 호출).
* **join()**: 분할된 왼쪽 작업이 완료될 때까지 기다린 후 결과를 가져온다.
* **결과 합산**: 왼쪽과 오른쪽 결과를 합쳐 최종 결과를 반환한다.

이렇게 구현한 코드를 실제 실행해보자.&#x20;

```javascript
package parallel.forkjoin;

import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ForkJoinMain1 {

    public static void main(String[] args) {
        List<Integer> data = IntStream.rangeClosed(1, 8)
                .boxed()
                .toList();

        log("[생성]" + data);

        // ForkJoinPool 생성 및 작업 수행
        long startTime = System.currentTimeMillis();
        ForkJoinPool pool = new ForkJoinPool(10);
        SumTask task = new SumTask(data);   // 1~8

        // 병렬로 합을 구한 후 결과 출력
        Integer result = pool.invoke(task);
        pool.close();
        long endTime = System.currentTimeMillis();
        log("time: " + (endTime - startTime) + "ms, sum: " + result);
        log("pool:" + pool);
    }
}
```

1. **데이터 생성**: `IntStream.rangeClosed(1, 8)` 를 사용해 1부터 8까지의 숫자 리스트를 생성한다.
2. **ForkJoinPool 생성**:
   1. `new ForkJoinPool(10)` 으로 최대 10개의 스레드를 사용할 수 있는 풀을 생성한다.
   2. 참고로 기본 생성자(`new ForkJoinPool()`) 을 사용하면 시스템 프로세서 수에 맞춰 스레드가 생성된다.&#x20;
3.  **invoke()**: 메인 스레드가 `pool.invoke(task)` 를 호출하면 `SumTask` 를 스레드 풀에 전달한다. `SumTask`&#x20;

    는 `ForkJoinPool` 에 있는 별도의 스레드에서 실행된다. 메인 스레드는 작업이 완료될 때까지 기다린 후 결과

    를 받는다.
4. **pool.close()**: 더 이상 작업이 없으므로 풀을 종료한다.
5. **결과 출력**: 계산된 리스트의 합과 실행 시간을 출력한다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.40.05.png" alt=""><figcaption></figcaption></figure>

작업이 2개로 분할 되어서 총 4초의 시간이 걸린 것을 확인할 수 있다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.36.46.png" alt=""><figcaption></figcaption></figure>

#### 정리&#x20;

* Fork/Join 프레임워크를 사용하면 RecursiveTask 를 통해 작업을 재귀적으로 분할하는 것을 확인할 수 있다. 여기서는 작업을 단순히 2개로만 분할해서 스레드도 동시에 2개만 사용할 수 있다.&#x20;
*   `THRESHOLD` (임계값)을 더 줄여서 작업을 더 잘게 분할하면 더 많은 스레드를 활용할 수 있다. 물론 이 경우 풀의

    스레드 수도 2개보다 더 많아야 효과가 있다.

## Fork/Join 프레임워크2 - 작업 훔치기&#x20;

### 더 분할하기&#x20;

이번에는 임계값을 줄여서 작업을 더 잘게 분할해보자. 다음 코드를 참고해서 `THRESHOLD` 값 4를 2로 변경하자.\
그러면 8개의 작업이 4개의 작업으로 분할될 것이다.

그리고 `ForkJoinMain1` 을 실행해보자.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.36.22.png" alt=""><figcaption></figcaption></figure>

임계값(`THRESHOLD` )을 4에서 2로 낮춘 결과, 작업이 더 잘게 분할되어 더 많은 스레드가 병렬로 작업을 처리하는 것을 확인할 수 있다. 여기서는 총 4개의 작업으로 분할되고, 2초의 시간이 소요되었다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 15.40.36.png" alt=""><figcaption></figcaption></figure>

#### 효율성 향상&#x20;

* 임계값을 낮춤으로써 더 많은 스레드(총 4개) 가 병렬로 작업을 처리했다.
* 이전 실행(임계값 4) 에서는 2개의 스레드만 사용되었다.&#x20;
* 로그를 보면 계산이 거의 동시에 시작되어 거의 동시에 완료된 것을 확인할 수 있다.&#x20;

## 작업 훔치기 알고리즘 (크게 중요하지 않기에 선택 사항)

지금까지 설명을 단순화 하기 위해 작업 훔치기(Work-Stealing) 알고리즘은 설명하지 않았다. 이번에는 작업 훔치기에 대해 자세히 알아보자.

* **Fork/Join 풀의 스레드는 각자 자신의 작업 큐를 가진다.**
  * 덕분에 작업을 큐에서 가져가기 위한 스레드간 경합이 줄어든다.
*   그리고 자신의 작업이 없는 경우, 그래서 스레드가 할 일이 없는 경우에 **다른 스레드의 작업 큐에 대기중인 작업을**

    **훔쳐서 대신 처리**한다.

~~이번 예제의 작업 훔치기에 대해서 그림으로 자세히 알아보고 싶으면 교제 참고하자.. 그림이 너무 많어;;)~~

#### 작업 훔치기 알고리즘&#x20;

이 예제에서는 작업량이 균등하게 분배되었지만, 실제 상황에서 작업량이 불균형할 경우 작업 훔치기 알고리즘이 동작하여 유휴 스레드가 다른 바쁜 스레드의 작업을 가져와 처리함으로써 전체 효율성을 높일 수 있다.

#### 정리

임계값을 낮춤으로써 작업이 더 잘게 분할되고, 그 결과 더 많은 스레드가 병렬로 작업을 처리할 수 있게 되었다. 이는Fork/Join 프레임워크의 핵심 개념인 **분할 정복(Divide and Conquer)** 전략을 명확하게 보여준다. 적절한 임계값 설정은 병렬 처리의 효율성에 큰 영향을 미치므로, 작업의 특성과 시스템 환경에 맞게 조정하는 것이 중요하다.

### Fork/Join 적절한 작업 크기 선택&#x20;

너무 작은 단위로 작업을 분할하면 스레드 생성과 관리에 드는 오버헤드가 커질 수 있으며, 너무 큰 단위로 분할하면 병렬 처리의 이점을 충분히 활용하지 못할 수 있다.

이 예제에서는 스레드 풀의 스레드가 10개로 충분히 남기 때문에 1개 단위로 더 잘게 쪼개는 것이 더 나은 결과를 보여준다.\
이렇게 하면 8개의 작업을 8개의 스레드가 동시에 실행할 수 있다. 따라서 1초만에 작업을 완료할 수 있다.

하지만 예를 들어, 1 \~ 1000까지 처리해야 하는 작업이라면 어떨까? 1개 단위로 너무 잘게 쪼개면 1000개의 작업으로 너무 잘게 분할된다. 스레드가 10개이므로 한 스레드당 100개의 작업을 처리해야 한다. 이 경우 스레드가 작업을 찾고 관리하는 부분도 늘어나고, 분할과 결과를 합하는 과정의 오버헤드도 너무 크다. 1000개로 쪼개고, 쪼갠 1000개를 합쳐야 한다.

예) 1 \~ 1000까지 처리해야 하는 작업, 스레드는 10개

* 1개 단위로 쪼개는 경우: 1000개의 분할과 결합이 필요. 한 스레드당 100개의 작업 처리
* 10개 단위로 쪼개는 경우: 100개의 분할과 결합이 필요. 한 스레드당 10개의 작업 처리
* 100개 단위로 쪼개는 경우: 10개의 분할과 결합이 필요. 한 스레드당 1개의 작업 처리
* 500개 단위로 쪼개는 경우: 2개의 분할과 결합이 필요. 스레드 최대 2개 사용 가능

작업시간이 완전히 동일하게 처리된다고 가정하면 이상적으로는 한 스레드당 1개의 작업을 처리하는 것이 좋다. \
왜냐하면 스레드를 100% 사용하면서 분할과 결합의 오버헤드도 최소화 할 수 있기 때문이다.

하지만 작업 시간이 다른 경우를 고려한다면 한 스레드당 1개의 작업 보다는 더 잘게 쪼개어 두는 것이 좋다. 왜냐하면`ForkJoinPool` 은 스레드의 작업이 끝나면 다른 스레드가 처리하지 못하고 대기하는 작업을 훔쳐서 처리하는 기능을 \
제공하기 때문이다.&#x20;

따라서 쉬는 스레드 없이 최대한 많은 스레드를 활용할 수 있다.

**그리고 실질적으로는 작업 시간이 완전히 균등하지 않은 경우가 많다. 작업별로 처리 시간이 다르고, 시스템 환경에 따라 스레드 성능도 달라질 수 있다. 이런 상황에서 최적의 임계값 선택을 위해 고려해야 할 요소들은 다음과 같다.**

* **작업의 복잡성**: 작업이 단순하면 분할 오버헤드가 더 크게 작용한다. 작업이 복잡할수록 더 작은 단위로 나누는 것이 유리할 수 있다. 예를 들어 `1 + 2 + 3 + 4` 의 아주 단순한 연산을 `1 + 2`, `3 + 4` 로 분할하게 되면 분할하고 합하는 비용이 더 든다.
*   **작업의 균일성**: 작업 처리 시간이 불균일할수록 작업 훔치기(work stealing)가 효과적으로 작동하도록 적절히 작

    은 크기로 분할하는 것이 중요하다.
* **시스템 하드웨어**: 코어 수, 캐시 크기, 메모리 대역폭 등 하드웨어 특성에 따라 최적의 작업 크기가 달라진다.
* **스레드 전환 비용**: 너무 작은 작업은 스레드 관리 오버헤드가 증가할 수 있다.

적절한 작업의 크기에 대한 정답은 없지만, CPU 바운드 작업이라고 가정할 때, CPU 코어수에 맞추어 스레드를 생성하고,\
작업 수는 스레드 수에 4 \~ 10배 정도로 생성하자. 물론 작업의 성격에 따라 다르다. 그리고 성능 테스트를 통해 적절한 값으로 조절하면 된다. \
-> **결론적으로, 스레드 당 4 \~ 10개의 작업을 수행하고 이후 튜닝을 통해 최적값을 찾아나가는 과정이 합리적이다.**&#x20;

## Fork/Join 프레임워크3 - 공용 풀&#x20;

### Fork/Join 공용 풀(Common Pool) 설명

자바 8에서는 공용 풀(Common Pool)이라는 개념이 도입되었는데, 이는 Fork/Join 작업을 위한 자바가 제공하는 기본 스레드 풀이다.

```java
// 자바 8 이상에서는 공용 풀(common pool) 사용 가능
ForkJoinPool commonPool = ForkJoinPool.commonPool();
```

#### Fork/Join 공용 풀의 특징&#x20;

* **시스템 전체에서 공유**: 애플리케이션 내에서 **단일 인스턴스로 공유**되어 사용된다.
* **자동 생성**: 별도로 생성하지 않아도 `ForkJoinPool.commonPool()` 을 통해 접근할 수 있다.
*   **편리한 사용**: 별도의 풀을 만들지 않고도 `RecursiveTask` /`RecursiveAction` 을 사용할 때 기본적으로 이

    공용 풀이 사용된다.
* **병렬 스트림 활용**: 자바 8의 병렬 스트림은 내부적으로 이 공용 풀을 사용한다. (뒤에서 설명한다.)
*   **자원 효율성**: 여러 곳에서 별도의 풀을 생성하는 대신 공용 풀을 사용함으로써 시스템 자원을 효율적으로 관리할

    수 있다.
*   **병렬 수준 자동 설정**: 기본적으로 시스템의 가용 프로세서 수에서 1을 뺀 값으로 병렬 수준(parallelism)이 설정

    된다. 예를 들어 CPU 코어가 14개라면 13개의 스레드가 사용된다. (자세한 이유는 뒤에서 설명)

Fork/Join 공용 풀은 쉽게 이야기해서, 개발자가 편리하게 Fork/Join 풀을 사용할 수 있도록 자바가 기본으로 제공하는 Fork/Join 풀의 단일 인스턴스이다.

Fork/Join 공용 풀을 어떻게 사용하는지 코드로 알아보자.

```java
package parallel.forkjoin;

import java.util.List;
import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ForkJoinMain2 {

    public static void main(String[] args) {
        int processorCount = Runtime.getRuntime().availableProcessors();
        ForkJoinPool commonPool = ForkJoinPool.commonPool();
        log("processorCount = " + processorCount + ", commonPool = "  + commonPool.getParallelism());

        List<Integer> data = IntStream.rangeClosed(1, 8)
                .boxed()
                .toList();

        log("[생성]" + data);

        // ForkJoinPool 생성 및 작업 수행
        SumTask task = new SumTask(data);
        Integer result = task.invoke(); // 공용 풀 사용
        log("최종 결과 : " + result);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.03.13.png" alt=""><figcaption></figcaption></figure>

#### 작업 실행 과정&#x20;

* 메인 스레드와 워커 스레드들이 함께 작업을 처리한다.&#x20;
* 워커 스레드 이름이 `worker-1`, `worker-2`, `worker-3` 으로 표시된다.&#x20;
* 메인 스레드도 작업 처리에 참여하는 것을 볼 수 있다.&#x20;

#### 정리&#x20;

* 공용 풀은 JVM이 종료될 때까지 계속 유지되므로, 별도로 풀을 종료(`shutdown()` )하지 않아도 된다.
*   이렇게 공용 풀(`ForkJoinPool.commonPool`)을 활용하면, 별도로 풀을 생성/관리하는 코드를 작성하지 않

    아도 간편하게 병렬 처리를 구현할 수 있다.

### 공용 풀이 CPU - 1 만큼 스레드를 생성하는 이유&#x20;

#### 메인 스레드의 참여

Fork/Join 작업은 공용 풀의 워커 스레드뿐만 아니라 메인 스레드도 연산에 참여할 수 있다. 메인 스레드가 단순히 대기하지 않고 직접 작업을 도와주기 때문에, 공용 풀에서 스레드를 14개까지 만들 필요 없이 13개의 워커 스레드 + 1개의 메인 스레드로 충분히 CPU 코어를 활용할 수 있다.

#### 다른 프로세스와의 자원 경쟁 고려&#x20;

애플리케이션이 실행되는 환경에서는 OS나 다른 애플리케이션, 혹은 GC(가비지 컬렉션) 같은 내부 작업들도 CPU를 사용해야 한다. 모든 코어를 최대치로 점유하도록 설정하면 다른 중요한 작업이 지연되거나, 컨텍스트 스위칭 비용(context switching)이 증가할 수 있다. 따라서 하나의 코어를 여유분으로 남겨 두어 전체 시스템 성능을 보다 안정적으로 유지하려는 목적이 있다.

#### 효율적인 자원 활용

일반적으로는 CPU 코어 수와 동일하게 스레드를 만들더라도 성능상 큰 문제는 없지만, 공용 풀에서 CPU 코어 수 - 1을 기본값으로 설정함으로써, 특정 상황(다른 작업 스레드나 OS 레벨 작업)에서도 병목을 일으키지 않는 선에서 효율적으로 CPU를 활용할 수 있다.

## 자바 병렬 스트림&#x20;

드디어 **자바의 병렬 스트림**(`parallel()` )을 사용해보자.

병렬 스트림은 Fork/Join 공용 풀을 사용해서 병렬 연산을 수행한다.

### 예제4 - 자바 병렬 스트림&#x20;

```java
package parallel;

import java.util.concurrent.ForkJoinPool;
import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ParallelMain4 {

    public static void main(String[] args) {
        int processorCount = Runtime.getRuntime().availableProcessors();
        ForkJoinPool commonPool = ForkJoinPool.commonPool();
        log("processorCount = " + processorCount + ", commonPool = "  + commonPool.getParallelism());

        long startTime = System.currentTimeMillis();

        int sum = IntStream.rangeClosed(1, 8)
                .parallel()
                .map(HeavyJob::heavyTask)
                .reduce(0, (a, b) -> a + b);

        long endTime = System.currentTimeMillis();
        log("time: " + (endTime - startTime) + "ms, sum: " + sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.10.42.png" alt=""><figcaption></figcaption></figure>

* 로그를 보면 `ForkJoinPool.commonPool-worker-N` 스레드들이 동시에 일을 처리하고 있다.
* 예제1에서 8초 이상 걸렸던 작업이, 이 예제에서는 모두 병렬로 실행되어 시간이 약 1초로 크게 줄어든다.
  * 만약 CPU 코어가 4개라면 공용 풀에는 3개의 스레드가 생성된다. 따라서 시간이 더 걸릴 수 있다.
* **직접 스레드를 만들 필요 없이** 스트림에 `parallel()` 메서드만 호출하면, 스트림이 자동으로 병렬 처리된다.

어떻게 복잡한 멀티스레드 코드 없이, `parallel()` 단 한 줄만 선언했는데, 해당 작업들이 병렬로 처리될 수 있을까?\
바로 앞서 설명한 공용 `ForkJoinPool` 을 사용하기 때문이다.&#x20;

스트림에서 `parallel()` 를 선언하면 스트림은 공용 `ForkJoinPool` 을 사용하고, 내부적으로 병렬 처리 가능한 스레드 숫자와 작업의 크기 등을 확인하면서, `Spliterator` 를 통해 데이터를 자동으로 분할한다. 분할 방식은 데이터 소스의 특성에 따라 최적화되어 있다. 그리고 공용 풀을 통해 작업을 적절한 수준으로 분할(Fork), 처리(Execute)하고,그 결과를 모은다(Join)

이때 요청 스레드(여기서는 메인 스레드)도 어차피 결과가 나올 때 까지 대기해야 하기 때문에, 작업에 참여해서 작업을 도운다.

개발자가 스트림을 병렬로 처리하고 싶다고 `parallel()`로 **선언**만 하면, 실제 `어떻게` 할지는 자바 스트림이 내부적으로 알아서 처리하는 것이다!

코드를 보면 복잡한 멀티스레드 코드 하나 없이 `parallel()` 단 한 줄만 추가했다. 이것이 바로 람다 스트림을 활용한 선언적 프로그래밍 방식의 큰 장점이다.

## 병렬 스트림 사용시 주의점1&#x20;

스트림에 `parallel()` 을 추가하면 병렬 스트림이 된다. 병렬 스트림은 Fork/Join 공용 풀을 사용한다. Fork/Join\
공용 풀은 CPU 바운드 작업(계산 집약적인 작업)을 위해 설계되었다. 따라서 스레드가 주로 대기해야 하는 I/O 바운드 작업에는 적합하지 않다.

* I/O 바운드 작업은 주로 네트워크 호출을 통한 대기가 발생한다. 예) 외부 API 호출, 데이터베이스 조회

### 주의사항 - Fork/Join 프레임워크는 CPU 바운드 작업에만 사용해라!

**Fork/Join 프레임워크는 주로 CPU 바운드 작업(계산 집약적인 작업)을 처리하기 위해 설계**되었다. 이러한 작업은 CPU 사용률이 높고 I/O 대기 시간이 적다. CPU 바운드 작업의 경우, 물리적인 CPU 코어와 비슷한 수의 스레드를 사용하는 것이 최적의 성능을 발휘할 수 있다. 스레드 수가 코어 수보다 많아지면 컨텍스트 스위칭 비용이 증가하고, 스레드 간 경쟁으로 인해 오히려 성능이 저하될 수 있기 때문이다.

따라서, I/O 작업처럼 블로킹 대기 시간이 긴 작업을 ForkJoinPool 에서 처리하면 다음과 같은 문제가 발생한다.&#x20;

1. **스레드 블로킹에 따른 CPU 낭비**
   1. `ForkJoinPool` 은 CPU 코어 수에 맞춰 제한된 개수의 스레드를 사용한다. (특히 공용 풀)
   2. I/O 작업으로 스레드가 블로킹되면 CPU가 놀게 되어, 전체 병렬 처리 효율이 크게 떨어진다.
2. **컨텍스트 스위칭 오버헤드 증가**
   1. I/O 작업 때문에 스레드를 늘리면, 실제 연산보다 대기 시간이 길어지는 상황이 발생할 수 있다.
   2. 스레드가 많아질수록 컨텍스트 스위칭(context switching) 비용도 증가하여 오히려 성능이 떨어질 수 있다.
3. **작업 훔치기 기법 무력화**
   1. `ForkJoinPool` 이 제공하는 작업 훔치기 알고리즘은, CPU 바운드 작업에서 빠르게 작업 단위를 계속 처리하도록 설계되었다. (작업을 훔쳐서 쉬는 스레드 없이 계속 작업)
   2. I/O 대기 시간이 많은 작업은 스래드가 I/O로 인해 대기하고 있는 경우가 많아, 작업 훔치기가 빛을 발휘하기 어렵고, 결과적으로 병렬 처리의 장점을 살리기 어렵다.
4. **분할-정복(작업 분할) 이점 감소**
   1. Fork/Join 방식을 통해 작업을 잘게 나누어도, I/O 병목이 발생하면 CPU 병렬화 이점이 크게 줄어든다.
   2. 오히려 분할된 작업들이 각기 I/O 대기를 반복하면서, `fork()`, `join()` 에 따른 오버헤드만 증가할 수 있다.

#### 정리&#x20;

공용 풀(Common Pool)은 Fork/Join 프레임워크의 편리한 기능으로, 별도의 풀 생성 없이도 효율적인 병렬 처리를 가능하게 한다. 하지만 블로킹 작업이나 특수한 설정이 필요한 경우에는 커스텀 풀을 고려해야 한다.

* CPU 바운드 작업이라면 `ForkJoinPool` 을 통해 병렬 계산을 극대화할 수 있지만, I/O 바운드 작업은 별도의\
  전용 스레드 풀을 사용하는 편이 더 적합하다.
* 예) `Executors.newFixedThreadPool()` 등등

### 병렬 스트림 - 예제5

예제를 통해 병렬 스트림 사용 시 주의점을 알아보자. 특히 여러 요청이 동시에 들어올 때 공용 풀에서 어떤 문제가 발생할 수 있는지 알아보자.

이 예제는 다음과 같은 시나리오를 시뮬레이션 한다.&#x20;

* 여러 사용자가 동시에 서버를 호출하는 상황
* 각 요청은 병렬 스트림을 사용하여 몇 가지 무거운 작업을 처리
* 모든 요청이 동일한 공용 풀(`ForkJoinPool.commonPool` )을 공유

```java
package parallel;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ParallelMain5 {

    public static void main(String[] args) throws InterruptedException {
        // 병렬 수준 3으로 제한
        System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "3");
//        System.out.println("ForkJoinPool.commonPool() = " + ForkJoinPool.commonPool());

        // 요청 풀 추가
        ExecutorService requestPool = Executors.newFixedThreadPool(100);
        int nThreads = 3;   // 1, 2, 3, 10, 100
        for (int i = 0; i < nThreads; i++) {
            String requestName = "request" + i;
            requestPool.submit(() -> logic(requestName));
            Thread.sleep(100);  // 스레드 순서를 확인하기 위해서 약간 대기
        }
        requestPool.close();
    }

    private static void logic(String requestName) {
        log("[" + requestName + "] START");
        long startTime = System.currentTimeMillis();

        int sum = IntStream.rangeClosed(1, 4)
                .parallel()
                .map(i -> HeavyJob.heavyTask(i, requestName))
                .reduce(0, (a, b) -> a + b);
        long endTime = System.currentTimeMillis();
        log("[" + requestName + "] time : " + (endTime - startTime) + "ms, sum:" + sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.23.23.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.24.27.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.24.35.png" alt=""><figcaption></figcaption></figure>

* 공용 풀의 제한된 병렬성
  *   공용 풀은 병렬 수준(parallelism)이 3으로 설정되어 있어, 최대 3개의 작업만 동시에 처리할 수 있다. 여기

      에 요청 스레드도 자신의 작업에 참여하므로 각 작업당 총 4개의 스레드만 사용된다.
  * 따라서 총 12개의 요청(각각 4개의 작업)을 처리하는데 필요한 스레드 자원이 부족하다.
* 처리 시간의 불균형&#x20;
  * request1: 1012ms (약 1초)
  * request2: 1931ms (약 2초)
  * request3: 2836ms (약 3초)
  * 첫 번째 요청은 거의 모든 공용 풀 워커를 사용할 수 있었지만, 이후 요청들은 제한된 공용 풀 자원을 두고 경쟁해야 한다. 따라서 완료 시간이 점점 느려진다.
* 스레드 작업 분배
  * 일부 작업은 요청 스레드(pool-1-thread-N)에서 직접 처리되고, 일부는 공용 풀(ForkJoinPool.commonPool-worker-N)에서 처리된다.
  * 요청 스레드가 작업을 도와주지만, 공용 풀의 스레드가 매우 부족하기 때문에 한계가 있다.

요청이 증가할수록 이 문제는 더 심각해진다. `nThreads` 의 숫자를 늘려서 동시 요청을 늘리면, 응답 시간이 확연하게 \
늘어나는 것을 확인할 수 있다.

#### 핵심 문제점&#x20;

* **공용 풀 병목 현상**: 모든 병렬 스트림이 동일한 공용 풀을 공유하므로, 요청이 많아질수록 병목 현상이 발생한다.
  * I/O 바운드 작업을 하기에 문제가 되는 것이다!
  * CPU 바운드 작업은 아닐수도 있다.&#x20;
* **자원 경쟁**: 여러 요청이 제한된 스레드 풀을 두고 경쟁하면서 요청의 성능이 저하된다.
* **예측 불가능한 성능**: 같은 작업이라도 동시에 실행되는 다른 작업의 수에 따라 처리 시간이 크게 달라진다.

특히 실무 환경에서는 주로 여러 요청을 동시에 처리하는 애플리케이션 서버를 사용하게 된다. 이때 수 많은 요청이 공용 풀을 사용하는 것은 매우 위험할수 있다. 따라서 병렬 스트림을 남용하면 전체 시스템 성능이 저하될 수 있다.

`nThreads` 를 너무 늘리는 것 보다, 차라리 `parallel()` 을 제거하는 것이 작업이 더 빨리 처리 될 수 있다.\
`nThreads` 를 `20` 으로 설정한 다음 실행해보면 매우 느린 응답을 확인할 수 있다. 이때 차라리 `parallel()` 을 제거하면 더 빠른 응답을 확인할 수 있다.

참고로 이번 예제에서 사용한 `heavyTask()` 는 1초간 스레드가 대기하는 작업이다. 따라서 I/O 바운드 작업에 가깝다. 이런 종류의 작업은 Fork/Join 공용 풀 보다는 별도의 풀을 사용하는 것이 좋다.

그렇다면 여러 작업을 병렬로 처리해야 하는데, I/O 바운드 작업이 많을 때는 어떻게 하면 좋을까? 이때는 스레드를 직접 사용하거나, `ExecutorService` 등의 별도의 스레드 풀을 사용해야 한다.&#x20;

## 병렬 스트림 사용시 주의점2&#x20;

### 별도의 풀 사용&#x20;

별도의 전용 스레드 풀을 사용해서 앞선 예제의 문제를 해결해보자.&#x20;

### 병렬 스트림 - 예제6

```java
package parallel;

import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

import static util.MyLogger.log;

public class ParallelMain6 {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService requestPool = Executors.newFixedThreadPool(100);

        // logic 처리 전용 스레드 풀 추가
        ExecutorService logicPool = Executors.newFixedThreadPool(400);

        int nThreads = 3;   // 1, 2, 3, 10, 100
        for (int i = 0; i < nThreads; i++) {
            String requestName = "request" + i;
            requestPool.submit(() -> logic(requestName, logicPool));
            Thread.sleep(100);  // 스레드 순서를 확인하기 위해서 약간 대기
        }
        requestPool.close();
        logicPool.close();
    }

    private static void logic(String requestName, ExecutorService es) {
        log("[" + requestName + "] START");
        long startTime = System.currentTimeMillis();

        Future<Integer> f1 = es.submit(() -> HeavyJob.heavyTask(1, requestName));
        Future<Integer> f2 = es.submit(() -> HeavyJob.heavyTask(2, requestName));
        Future<Integer> f3 = es.submit(() -> HeavyJob.heavyTask(3, requestName));
        Future<Integer> f4 = es.submit(() -> HeavyJob.heavyTask(4, requestName));

        int sum;
        try {
            Integer v1 = f1.get();
            Integer v2 = f2.get();
            Integer v3 = f3.get();
            Integer v4 = f4.get();
            sum = v1 + v2 + v3 +v4;
        } catch (Exception e) {
            throw new RuntimeException(e);
        }

        long endTime = System.currentTimeMillis();
        log("[" + requestName + "] time : " + (endTime - startTime) + "ms, sum:" + sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.35.16.png" alt=""><figcaption></figcaption></figure>

이 예제는 작업 유형에 적합한 전용 스레드 풀을 사용하는 것의 이점을 보여준다. 특히 I/O 바운드 작업이나 많은 동시 요청을 처리하는 서버 환경에서는 이러한 접근 방식이 더 효과적이다.

### 병렬 스트림 - 예제7&#x20;

앞선 예제는 쉽게 설명하기 위해 코드를 풀어서 사용했다. 간결하게 개선해보자.

```java
package parallel;

import java.util.List;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.stream.IntStream;

import static util.MyLogger.log;

public class ParallelMain7 {

    public static void main(String[] args) throws InterruptedException {
        ExecutorService requestPool = Executors.newFixedThreadPool(100);

        // logic 처리 전용 스레드 풀 추가
        ExecutorService logicPool = Executors.newFixedThreadPool(400);

        int nThreads = 3;   // 1, 2, 3, 10, 100
        for (int i = 0; i < nThreads; i++) {
            String requestName = "request" + i;
            requestPool.submit(() -> logic(requestName, logicPool));
            Thread.sleep(100);  // 스레드 순서를 확인하기 위해서 약간 대기
        }
        requestPool.close();
        logicPool.close();
    }

    private static void logic(String requestName, ExecutorService es) {
        log("[" + requestName + "] START");
        long startTime = System.currentTimeMillis();

        List<Future<Integer>> futures = IntStream.rangeClosed(1, 4)
                .mapToObj(i -> es.submit(() -> HeavyJob.heavyTask(i, requestName)))
                .toList();

        int sum = futures.stream()
                .mapToInt(f -> {
                    try {
                        return f.get();
                    } catch (Exception e) {
                        throw new RuntimeException(e);
                    }
                })
                .sum();

        long endTime = System.currentTimeMillis();
        log("[" + requestName + "] time : " + (endTime - startTime) + "ms, sum:" + sum);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-23 16.36.51.png" alt=""><figcaption></figcaption></figure>
