# Guide to the java.util.concurrent.Future

> 참고 링크&#x20;
>
> [https://www.baeldung.com/java-future](https://www.baeldung.com/java-future)\
> [https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html)
>
> [https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinTask.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ForkJoinTask.html)

## 1. Future 인스턴스 생성&#x20;

* `Future` 클래스는 비동기 연산에 대한 상태 및 결과를 확인할 수 있다.&#x20;
* 오래 걸리는 작업(메서드) 에 대해서 비동기 통신을 사용하게 되면 완료될 때까지 기다리지 않고 다른 프로세스를 실행시킬 수 있기 때문에, `Future` 클래스를 통한 비동기 처리는 좋은 선택지가 될 수 있다.&#x20;
* `Future` 의 비동기적 특성을 활용하는 작업의 예는 다음과 같다.&#x20;
  * 계산 집약적 프로세스(수학적, 과학적 계산)&#x20;
  * 대용량 데이터 구조 조작
  * 원격 메서드 호출(파일 다운로드, HTML 스크래핑, 웹서비스)

### 1-1. FutureTask 를 사용하여 Futures 구현&#x20;

* 예를 들어, `Integer` 의 제곱을 계산하는 매우 간단한 클래스를 만든다 생각하자.&#x20;
* `Callable` 은 결과를 반환하는 작업을 나타내는 인터페이스이며, 단일 `call()` 메서드가 있다. 여기서는 람다 표현식을 사용하여 나타냈다.
* `Callable` 인스턴스를 생성하고 `submit` 의 매개변수로 전달하면, 내부적으로 `FutureTask` 객체를 생성하고, 이 `FutureTask` 가 쓰레드 풀에서 실행될 작업을 나타낸다.&#x20;
  * 구체적으로는 `Callable`, `Runnable` 작업을 받아서 이를 감싸는 `FutureTask` 를 만들고, 이를 쓰레드 풀에 제출한다.
  * 이 `FutureTask` 는 비동기 작업의 결과를 처리하고, 그 결과를 `Future` 인터페이스를 통해 반환한다.&#x20;

```java
public class SquareCalculator {
    
    private ExecutorService executor = Executors.newSingleThreadExecutor(); 
    
    public Future<Integer> calculate(Integer input) {
        return executor.submit(() -> {
            System.out.println("calculating square for: " + input);
            Thread.sleep(1000);
            return input * input;
        });
    }
}
```

## 2. Future 인스턴스 사용&#x20;

### 2-1. isDone(), get() 을 사용하여 결과 얻기

* `calculate()` 를 호출 하고 반환된 `Future` 를 사용하여 `Integer` 를 가져와야 한다.&#x20;
* `Future.isDone()` 은 `executor` 가 작업 처리를 완료했는지 알려준다. 작업이 완료되면 `true` 를 반환하고, 그렇지 않으면 `false` 를 반환한다.&#x20;
* 계산에서 실제 결과를 반환하는 메서드는 `Future.get()` 이다. 이 메서드는 작업이 완료될 때까지 실행을 차단하는 것을 볼 수 있다. \
  \-> `Future.get()` 을 호출하기 이전에 `Future.isDone()` 을 호출해 작업이 완료되지 않았다면, 다른 작업을 수행하는 형식으로 쓰레드 자원을 효율적으로 사용할 수 있다.&#x20;

<pre class="language-java"><code class="lang-java"><strong>Future&#x3C;Integer> future = new SquareCalculator().calculate(10); 
</strong>
while(!future.isDone()) {
    System.out.println("Calculating...");
    Thread.sleep(300);
}

Integer result = future.get(); 
</code></pre>

* 아래와 같이 `get(long, TimeUnit)` 을 사용하는 경우도 있는데, 이는 지정된 시간 초과 기간 내에 작업이 반환되지 않으면 `TimeoutException` 을 `throw` 한다.&#x20;

```java
Integer result = future.get(500, TimeUnit.MILLISECONDS);
```

### 2-2. cancel() 로 Future 취소하기&#x20;

* 작업을 요청했지만, 어떤 이유에서 결과에 더 이상 관심이 없다고 가정해보자. `Future.cancel(boolean)` 을 사용하여 `executor` 에게 작업을 중기하고 기본 쓰레드를 중지하라고 할 수 있다.&#x20;
* 아래 코드에서 `Future` 인스턴스는 결코 작업을 완료할 수 없다. 사실 `cancel()` 을 호출한 후 해당 인스턴스에서 `get()` 을 호출하려 하면 `CancellationException` 을 `throw` 한다.&#x20;
* `Future.isCancelled()` 는 `Future` 가 이미 취소되었는지 알려준다. 이는 `CancellationException` 을 피하는데 매우 유용할 수 있다.&#x20;
* `Future.cancel(true)` 인 경우는 지금 실행중인 작업 또한 취소시킨다는 것을 의미한다.&#x20;

```java
Future<Integer> future = new SquareCalculator().calcolate(4); 

boolean canceled = future.cancel(true); 
```

## 3. ThreadPool 을 사용한 멀티스레딩 강화&#x20;

* 아래 코드의 `ExecutorService` 는 `Executors.newSingleThreadExecutor` 로 얻었기 때문에 단일 쓰레드이다.&#x20;
* 이 단일 쓰레드를 강조하기 위해서 동시에 실행시켜보자.&#x20;
  * **아래 결과를 살펴보면 프로세스가 병렬적이지 않다는 것은 분명하다.**&#x20;
  * **두 번째 작업은 첫번째 작업이 완료된 후에야 시작되는데, 전체 프로세스가 완료되는 데 약 2초가 걸린다.**&#x20;

```java
SquareCalculator squareCalculator = new SquareCalculator(); 

Future<Integer> future1 = squareCalculator.calculator(10); 
Future<Integer> future2 = squareCalculator.calculator(100); 

while(!(future1.isDone() && future2.isDone())){
    System.out.println(
        String.format(
            "future1 is %s and future2 is %s",
            future1.isDone() ? "done" : "not done",
            future2.isDone() ? "done" : "not done"
        )
    );
    Thread.sleep(300); 
}

Integer result1 = future1.get(); 
Integer result2 = future2.get(); 

System.out.println(result1 + " and " + result2); 

squareCalculator.shotdown(); 
```

```java
calculating square for: 10
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
calculating square for: 100
future1 is done and future2 is not done
future1 is done and future2 is not done
future1 is done and future2 is not done
100 and 10000
```

* 프로그램을 진짜 멀티쓰레드화 시키려면 `ExecutorService` 의 구현체를 다른 것을 사용해야 한다.&#x20;
* `Executors.newFixedThreadPool()` 에서 제공하는 스레드 풀을 사용하면 예제의 동작이 어떻게 변경되는지 살펴보자.&#x20;
  * **변경 후 두 작업이 동시에 실행되고 종료되는 것을 볼 수 있으며, 전체 프로세스가 완료되는데 약 1초가 걸린다.**&#x20;
  * **멀티쓰레드 환경으로 바뀌었다.**

```java
public class SquareCalculator {
 
    private ExecutorService executor = Executors.newFixedThreadPool(2);
    
    //...
}
```

```java
calculating square for: 10
calculating square for: 100
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
future1 is not done and future2 is not done
100 and 10000
```

## 4. ForkJoinTask 개요&#x20;

* `ForkJoinTask` 는 `Future` 를 구현하는 추상 클래스이며, `ForkJoinPool` 에 있는 소수의 실제 쓰레드가 호스팅하는 많은 수의 작업을 실행할 수 있다.&#x20;
* `ForkJoinTask` 의 주요 특징은 일반적으로 주요 작업을 완료하는 데 필요한 작업의 일부로 새로운 하위 작업을 생성한다는 것이다. \
  \-> `fork()` 로 호출하여 새로운 작업을 생성하고, `join()` 으로 모든 결과를 수집하므로 클래스의 이름이 되었다.&#x20;
* `ForkJoinTask` 를 구현하는 추상 클래스는 두 개(`RecursiveTask, RecursiveAction`)이다. 이름에서도 알 수 있듯이 이러한 클래스는 복잡한 수학적 계산, 재귀적 작업 등에 사용된다. &#x20;
  * 완료 시 값을 반환하는 `RecursiveTask`,&#x20;
  * 완료 시 아무것도 반환하지 않는 `RecursiveAction` 이다.&#x20;
* 이전 예제를 확장해 `Integer` 가 주어지면 모든 팩토리얼 요소에 대한 제곱 합을 계산하는 클래스를 만들어 보자.\
  \-> 예를 들어, 숫자 4를 계산기에 전달하면 4² + 3² + 2² + 1²의 합인 30을 얻어야 합니다.
* 먼저, `RecursiveTask` 의 구현체를 만들고, 그 `Compute()` 메서드를 구현해야 한다.
* 어떻게 재귀성을 만드는지에 대해 주목하자.&#x20;
  * **`Compute()` 내에서 `FactorialSquareCalculator` 의 새 인스턴스를 만든다.**&#x20;
  * **넌 블록킹 메서드인 `fork()` 를 호출해, ForkJoinPool 에 이 하위 작업의 실행을 시작하도록 요청한다.** \
    **-> 멀티쓰레드로 재귀적으로 생성된 인스턴스에서 `compute()` 메서드가 호출된다.**&#x20;
  * **`join()` 메서드는 결과를 반환하고, 여기에 현재 방문하고 있는 숫자의 제곱을 더한다.**&#x20;

```java
public class FactorialSquareCalculator extends RecursiveTask<Integer> {
    private Integer n; 
    
    public FactorialSquareCalculator(Integer n) {
        this.n = n;
    }
    
    @Override 
    protected Integer compute() {
        if(n <= 1){
            return n;
        }
        
        FactorialSqiareCalculator calculator 
            = new FactorialSquareCalculator(n-1); 
            
        calculator.fork(); 
        
        return n*n + calculator.join(); 
    }
}
```

* 이제 실행  및 쓰레드 관리를 처리하기 위해 ForkJoinPool 을 생성하자.&#x20;

```java
ForkJoinPool forkJoinPool = new ForkJoinPool(); 

FactorialSquareCalculator calculator = new FactorialSquareCalculator();

forkJoinPool.execute(calculator);
```

