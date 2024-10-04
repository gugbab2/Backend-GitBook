# Guide to the Volatile Keyword in Java

> 참고 링크&#x20;
>
> [https://www.baeldung.com/java-volatile](https://www.baeldung.com/java-volatile)

## 1. 개요&#x20;

* 필요한 동기화가 없다면 컴파일러, 프로세서는 온갖 종류의 최적화를 지원한다. 이러한 최적화는 일반적으로 유익하지만 때로는 미묘한 문제를 일으킬 수 있다.&#x20;
* Java, JVM 은 메모리 순서를 제어하는 여러가지 방법을 제공하며 `Volatile` 또한 그 중 하나이다.&#x20;

## 2. 공유 멀티프로세서와 아키텍처&#x20;

* 프로세서는 프로그램 명령어를 실행하는 역할을 한다. 따라서 RAM 에서 프로그램 명령어와 필요한 데이터를 검색해야 한다.&#x20;
* CPU 는 RAM 에 비해서 초당 많은 명령어를 처리할 수 있으므로, 매번 RAM 에서 데이터를 가져오는 것은 바람직 하지 않다..&#x20;
* 이 상황을 개선하기 위해서 프로세서는 [Out of Order Execution](https://en.wikipedia.org/wiki/Out-of-order\_execution) , [Branch Prediction](https://en.wikipedia.org/wiki/Branch\_predictor) , [Speculative Execution](https://en.wikipedia.org/wiki/Speculative\_execution) , Caching 과 같은 트릭을 사용한다.&#x20;
* 다양한 코어가 더 많은 명령과 데이터를 처리함에 따라서, 그들은 더 관련성 있는 데이터와 명령어로 캐시를 채운다.&#x20;
* 이것은 [**캐시 일관성**](https://en.wikipedia.org/wiki/Cache\_coherence) **문제가 생길 수 있지만, 전반적으로 성능을 향상시킨다.** \
  \-> 캐시 일관성 문제 : 각각의 코어 내 캐시에 담긴 공유 리소스 데이터의 균일성일 깨지는 것을 의미한다.&#x20;
* **하나의 쓰레드가 캐시된 값을 업데이트 할 때 어떤 일이 발생하는지 한번 더 생각해보아야 한다!**

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2024-10-03 17.32.57.png" alt="" width="563"><figcaption></figcaption></figure>

## 3. 캐시 일관성 문제&#x20;

* 아래 `TaskRunner` 는 `number`, `ready` 라는 간단한 변수를 정의한다.&#x20;
* `run()` 메서드 내부에서 `ready` 변수가 `false` 라면 다른 대기중인 쓰레드에게 우선권을 넘겨준다. 그 후 `ready` 변수가 `true` 가 되면 `number` 변수를 출력한다.&#x20;
* **많은 사람들은 짧은 시간 후에 `42` 를 출력할 것이라고 생각할 수 있지만, 지연이 훨씬 길 수도 있다..** \
  **-> 이러한 현상은 적절한 메모리 가시성과 재정렬이 부족하기 때문이다.**&#x20;

> `Thread.yield()`
>
> * 현재 실행 중인 쓰레드가 실행 상태에서 **대기 상태로 전환되도록 힌트를 주는 메서드이다.**&#x20;
> * 이 메서드를 호출하면 **현재 쓰레드가 CPI 자원을 다른 쓰레드에게 양보하려고 시도한다.**&#x20;
> * **그러나 이는 권고 사항일 뿐, JVM 스케줄러가 반드시 이 요청을 수용하는 것은 아니다.**&#x20;



```java
public class TaskRunner {

    private static int number;
    private static boolean ready;

    private static class Reader extends Thread {

        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }

            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new Reader().start();
        number = 42;
        ready = true;
    }
}
```

### 3-1. 메모리 가시성&#x20;

* 위 코드에서 두 개의 애플리케이션 쓰레드가 있다. 메인 쓰레드와 `Reader` 쓰레드이다.&#x20;
* OS 가 두 개의 다른 CPU 코어에서 해당 쓰레드를 스케줄링하는 시나리오를 생각해보자,&#x20;
  * 메인 쓰레드는 `number`, `ready` 변수의 사본을 로컬 캐시에 보관한다.&#x20;
  * `Reader` 쓰레드도 `number`, `ready` 변수의 사본을 로컬캐시에 보관한다.
  * 메인 쓰레드는 캐시된 값을 업데이트 한다.&#x20;
* Java 메모리 모델에서는 각 쓰레드가 CPU 의 로컬 캐시( L1, L2 Cache)를 사용하여 변수 값을 저장하고 읽을 수 있는데,&#x20;
* **명시적으로 동기화되지 않았기 때문에, 특정 쓰레드에서 값을 변경하더라도 다른 쓰레드에서 그 변경 사항을 즉시 볼 수 있을 것이라는 보장이 없다..**&#x20;
* **다시 말해서 `Reader` 쓰레드는 약간의 지연과 함께 업데이트 된 값을 즉시 볼수도, 전혀 못볼수도 있다.**&#x20;

### 3-2. 재정렬&#x20;

* 문제를 더 악화시키는 것은, `Reader` 쓰레드가 실제 프로그램 순서가 아닌 다른 순서로 실행될 수 있다.
  * `number = 42;` 와 `ready = true;` 의 순서가 변경되어 실행될 수 있다.&#x20;
* 재정렬은 성능 개선을 위한 최적화 기술이다.&#x20;

## 4.  volatile 을 통한 일관성문제 해결&#x20;

* `volatile` 을 사용하면 캐시 일관성 문제를 해결할 수 있다.&#x20;
  * `volitile` 과 관련된 변수는 재정렬을 수행하지 않는다.&#x20;
  * `volitile` 과 관련된 변수는 캐시 메모리에 읽고 쓰는 것이 아니라 메인 메모리에 읽고 쓴다.&#x20;

```java
public class TaskRunner {

    private volatile static int number;
    private volatile static boolean ready;

    // same as before
}
```

## 5. "Happens-Before" 규칙과 `volatile` 변수

* `volatile` 변수를 이용해 두 쓰레드 간 메모리 가시성이 확실하게 보장된다.&#x20;
* **`volatile` 변수 자체뿐만 아니라, 그 변수를 쓰기 전에 쓰레드 A 가 조작한 다른 변수들의 값도 쓰레드 B 에게 보이게 되는 것이다. 이게 바로 Happens-Before 규칙의 핵심이다.** \
  **-> `volatile` 변수 이전의 값들 또한 메모리 가시성을 보장한다.**&#x20;
* 구체적으로 어떻게 동작하는지 살펴보자&#x20;
  * 쓰레드 A 가 먼저 `volatile` 변수를 변경했으면, 그 이전에 쓰레드 A 가 메모리에서 읽거나 쓴 다른 변수들도 메인 메모리에 저장된다.&#x20;
  * 쓰레드 B 가 나중에 그 `volatile` 변수를 읽으면, 쓰레드 A 가 쓰기 전에 사용한 다른 변수들의 최신 값도 함꼐 보인다.&#x20;

```java
public class Example {
    private static int nonVolatileVar;
    private static volatile boolean flag;

    public static void main(String[] args) {
        Thread writer = new Thread(() -> {
            nonVolatileVar = 42;  // nonVolatileVar에 값 42를 씀
            flag = true;          // flag를 true로 설정 (volatile)
        });

        Thread reader = new Thread(() -> {
            while (!flag) {
                // flag가 true가 될 때까지 기다림
            }
            System.out.println(nonVolatileVar);  // flag가 true가 되면 nonVolatileVar 값을 읽음
        });

        writer.start();
        reader.start();
    }
}
```

## 7. volatile 키워드의 장단점

### 7-1. 장점&#x20;

* 메모리 가시성 보장&#x20;
  * `volatile` 변수는 한 쓰레드에서 변경될 때 즉시 메인 메모리에 기록되므로, 다른 쓰레드는 변경된 값을 즉시 볼 수 있다.&#x20;
  * 따라서 쓰레드 간의 데이터 일관성을 확보하는데 유용하다.&#x20;
* 성능&#x20;
  * `synchronized` 에 비해 경량화된 동기화 방법으로, 락을 사용하는 것이 아니기 때문에 성능 저하가 적다.&#x20;
  * 단순히 플래그나 상태 변수의 경우 `volatile` 를 사용하는 것이 효율적이다.&#x20;
* 단순성&#x20;
  * `volatile` 은 매우 간단하게 사용할 수 있으며, 특별한 설정 없이도 쉽게 적용이 가능하다.&#x20;

### 7-2. 단점&#x20;

* 원자성 부족&#x20;
  * `volatile` 은 메모리 가시성을 보장하지만, 원자성을 보장하지 않는다.
  * 즉 여러 쓰레드가 동시에 `volatile` 변수를 읽거나 쓸 경우 데이터 일관성의 문제가 생길 수 있다.&#x20;
  * 예를 들어, `volatile` 변수를 읽고 변경하는 작업이 여러 쓰레드에서 발생한다면, 예상치 못한 결과를 초래할 수 있다.&#x20;
* 복잡한 상태 관리 불가&#x20;
  * `volatile` 은 단순한 플래그 변수나 상태 변수에 적합하지만, 복잡한 상태 관리에는 적합하지 않다.&#x20;
  * 예를 들어, 카운터 증가와 같은 복잡한 작업은 `volatile` 로 안전하게 처리할 수 없다.&#x20;
* 다른 매커니즘과 결합 필요&#x20;
  * `volatile` 만으로는 동기화가 충분하지 않은 경우가 많다 ..&#x20;
  * 따라서 `volatile` 을 사용할 때는 다른 동기화 매커니즘과 결합하여 사용하는 것이 필요할 수 있다.&#x20;
