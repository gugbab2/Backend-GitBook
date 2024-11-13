# CAS - 동기화와 원자적 연산

## 원자적 연산&#x20;

* 컴퓨터 과학에서 사용하는 **원자적 연산(atomic operation)** 의 의미는 **해당 연산이 더 이상 나눌 수 없는 단위로 수행된다는 것을 의미**한다. 즉, 원자적 연산은 중단되지 않고, 다른 연산과 간섭 없이 완전히 실행되거나 전혀 실행되지 않는 성질을 가지고 있다. **쉽게 이야기해서 멀티스레드 상황에서 다른 스레드의 간섭 없이 안전하게 처리되는 연산이라는 뜻이다.**&#x20;
* 예를 들어, 다음과 같은 필드가 있을 때\
  `volatile int i = 0;`&#x20;
* 다음 연산은 둘로 쪼갤 수 없는 원자적 연산이다. \
  `i = 1;`
  * 왜냐하면 이 연산은 다음 단 하나의 순서로 실행되기 때문이다.&#x20;
    * 오른쪽에 있는 1의 값은 왼쪽의 i 변수에 대입한다.&#x20;
* 하지만, 다음 연산은 원자적 연산이 아니다. \
  `i = i + 1;`&#x20;
  * 왜냐면 이 연산은 다음 순서로 나누어 실행되기 때문이다.&#x20;
    * 오른쪽에 있는 i 값을 읽는다. i 의 값은 10이다.&#x20;
    * 읽은 10에 1을 더해서 11을 만든다.&#x20;
    * 더한 11을 왼쪽의 i 변수에 대입한다.&#x20;
* 원자적 연산은 멀티스레드 상황에서 아무런 문제가 발생하지 않는다. 하지만 원자적 연산이 아닌 경우에는 `synchronized` 블럭이나 `Lock` 을 사용해서 안전한 임계 영역을 만들어야만 한다.&#x20;

### 원자적 연산 - 시작&#x20;

* `THREAD_COUNT` 수 만큼 스레드를 생성하고, `incrementInteger.increment()` 를 호출한다.&#x20;
* 스레드를 1000개 생성했다면, `increment()` 메서드도 1000번 호출되기 때문에, 결과는 1000이 되어야 한다.&#x20;
* 참고로 스레드가 너무 빨리 실행되기 때문에, 여러 스레드가 동시에 실행되는 상황을 확인하기 어렵다. 그래서 `run()` 메서드에 `sleep(10)` 을 두어서, 최대한 많은 스레드가 동시에 `increment()` 를 호출하도록 한다.&#x20;

```java
package thread.cas.increment;

public interface IncrementInteger {
    void increment();

    int get();
}
```

```java
package thread.cas.increment;

public class BasicInteger implements IncrementInteger{

    private int value;

    @Override
    public void increment() {
        value++;
    }

    @Override
    public int get() {
        return this.value;
    }
}
```

```java
package thread.cas.increment;

import java.util.ArrayList;
import java.util.List;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class IncrementThreadMain {

    public static final int THREAD_COUNT = 1000;

    public static void main(String[] args) throws InterruptedException {
        test(new BasicInteger());
//        test(new VolatileInteger());
//        test(new SyncInteger());
//        test(new MyAtomicInteger());
    }

    private static void test(IncrementInteger incrementInteger) throws InterruptedException {
        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                sleep(10);  // 너무 빨리 실행되기 때문에, 다른 스레드와 동시 실행을 위해 잠깐 쉬었다가 실행
                incrementInteger.increment();
            }
        };

        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < THREAD_COUNT; i++) {
            Thread thread = new Thread(runnable);
            threads.add(thread);
            thread.start();
        }

        for (Thread thread : threads) {
            thread.join();
        }

        int result = incrementInteger.get();
        log(incrementInteger.getClass().getSimpleName() + " result: " + result);

    }
}
```

* 실행 결과를 보면 기대한 1000이 아니라, 다른 숫자가 보인다. 아마도 실행 환경에 따라서 다르겠지만, 1000 이 아니라 조금 더 적은 숫자로 보일 것이다. 물론 실행 환경에 따라서 1000이 보일 수 있다.&#x20;
* 이 문제는 앞서 설명한 것처럼 여러 스레드가 동시에 원자적이지 않은 `value++` 를 호출했기 때문에, 발생한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 11.56.38.png" alt=""><figcaption></figcaption></figure>

### 원자적 연산 - volatile, synchronized&#x20;

#### volatile

```java
package thread.cas.increment;

public class VolatileInteger implements IncrementInteger{

    volatile private int value;

    @Override
    public void increment() {
        value++;
    }

    @Override
    public int get() {
        return this.value;
    }
}
```

#### synchronized

```java
package thread.cas.increment;

public class SyncInteger implements IncrementInteger{

    private int value;

    @Override
    public synchronized void increment() {
        value++;
    }

    @Override
    public synchronized int get() {
        return this.value;
    }
}
```

* `volatile`&#x20;
  * 실행 결과를 보면 여전히 `VolatileInteger` 도 여전히 1000 이 아니라 더 작은 숫자가 나온다.&#x20;
  * `volatile` 은 여러 CPU 사이에 발생하는 캐시 메모리와 메인 메모리가 동기화 되지 않는 문제를 해결할 뿐이다.&#x20;
  * `volatile` 을 사용하면 CPU 캐시 메모리를 무시하고, 메인 메모리를 직접 사용하도록 한다. 하지만 지금 이 문제는 캐시 메모리가 영향을 줄 수는 있지만, 캐시 메모리를 사용하지 않고, 메인 메모리를 직접 사용해도 여전히 발생하는 문제이다.&#x20;
  * **이 문제는 연산 자체가 나누어져 있기 때문에, 발생한다.** `volatile` 은 연산 자체를 원자적으로 묶어주는 기능이 아니다.&#x20;
* `synchronized`&#x20;
  * 이렇게 연산 자체가 나누어진 경우, `synchronized` 블럭이나, `Lock` 등을 사용해서 안전한 임계 영역을 만들어야 한다.&#x20;
  * 실행 결과를 보면 SyncInteger 는 1000 이 나온다.&#x20;
  * 모니터 락을 통해서 임계 영역을 설정했기 때문이다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 11.59.23.png" alt=""><figcaption></figcaption></figure>

### 원자적 연산 - AtomicInteger&#x20;

* 앞서 만든 SyncInteger 와 같이 멀티스레드 상황에서 안정하게 증가 연산을 수행할 수 있는 AtomicInteger 라는 클래스를 제공한다.
  * &#x20;이름 그대로 원자적인 Integer 라는 뜻이다.&#x20;

```java
package thread.cas.increment;

import java.util.concurrent.atomic.AtomicInteger;

public class MyAtomicInteger implements IncrementInteger{

    AtomicInteger atomicInteger = new AtomicInteger(0);

    @Override
    public void increment() {
        atomicInteger.incrementAndGet();
    }

    @Override
    public int get() {
        return atomicInteger.get();
    }
}

```

* 실행 결과를 보면 `AtomicInteger` 를 사용하면 `MyAtomicInteger` 의 결과도 1000인 것을 확인할 수 있다.&#x20;
  * 1000개의 스레드가 안전하게 증가 연산을 수행한 것이다.&#x20;
* `AtomicInteger` 는 멀티스레드 상황에서 안전하고 또 다양한 값 증가, 감소 연산을 제공한다. 특정 값을 증가하거나, 감소해야 하는데 여러 스레드가 해당 값을 공유해야 한다면, `AtomicInteger` 응 사용하면 된다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.09.26.png" alt=""><figcaption></figcaption></figure>

### 원자적 연산 - 성능 테스트&#x20;

#### BasicInteger&#x20;

* 가장 빠르다.&#x20;
* CPU 캐시를 적극 사용한다. CPU 캐시의 위력을 알 수 있다.&#x20;
* 안전한 임계 영역도 없고, `volatile` 도 사용하지 않기 때문에, 멀티스레드 상황에서는 사용할 수 없다.&#x20;
* 단일 스레드가 사용하는 경우에 효율적이다.&#x20;

#### Volatile&#x20;

* `volatile` 을 사용해서 CPU 캐시를 사용하지 않고 메인 메모리를 사용한다.&#x20;
* 안전한 임계 영역이 없기 때문에, 멀티 스레드 상황에서는 사용할 수 없다.&#x20;
* 단일 스레드가 사용하기에는 다. `BasicInteger` 보다 느리다. 그리고 멀티스레드 상황에도 안전하지 않다.&#x20;

#### SyncInteger

* `synchronized` 를 사용한 안전한 임계 영역이 있기 때문에, 멀티스레드 상황에서도 안정하게 사용할 수 있다.&#x20;
* `MyAtomicInteger` 보다 성능이 느리다.&#x20;

#### MyAtomicInteger&#x20;

* 자바가 제공하는 `AtomicInteger` 를 사용한다. 멀티스레드 상황에서 안전하게 사용할 수 있다.&#x20;
* 성능도 `synchronized`, `Lock(ReentrantLock)` 을 사용하는 경우보다 1.5 \~ 2배 정도 빠르다.&#x20;
* 놀랍게도 AtomicInteger 가 제공하는 incrementAndGet() 메서드는 락을 사용하지 않고 원자적 연산을 만들어낸다.&#x20;

```java
package thread.cas.increment;

import static util.MyLogger.log;

public class IncrementPerformanceMain {

    public static final long COUNT = 100_000_000;

    public static void main(String[] args) {
        test(new BasicInteger());
        test(new VolatileInteger());
        test(new SyncInteger());
        test(new MyAtomicInteger());
    }

    private static void test(IncrementInteger incrementInteger) {
        long startMs = System.currentTimeMillis();

        for (long i = 0; i < COUNT; i++) {
            incrementInteger.increment();
        }

        long endMs = System.currentTimeMillis();
        log(incrementInteger.getClass().getSimpleName() + " : ms=" + (endMs - startMs));
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.12.10.png" alt=""><figcaption></figcaption></figure>

## CAS 연산1&#x20;

### 락 기반 방식의 문제점&#x20;

* SyncInteger 와 같은 클래스를 데이터를 보호하기 위해서 락을 사용한다.&#x20;
* 여기서 말하는 락은 synchronized, Lock(ReentrantLock) 등을 사용하는 것을 말한다.&#x20;
* 락은 특정 자원을 보호하기 위해 스레드가 해당 자원에 대한 접근하는 것을 제한한다. 락이 걸려 있는 동안 다른 스레드들은 해당 자원에 접근할 수 없고, 락이 해제될 때까지 대기해야 한다.&#x20;
* 또한 락 기반 접근에서는 락을 획득하고 해제하는데 시간이 소요된다.&#x20;
* 예를 들어, 락을 사용하는 연산이 있다고 가정하자. 락을 사용하는 방식은 다음과 같이 작동한다.&#x20;
  * 락이 있는지 확인한다.&#x20;
  * 락을 획득하고, 임계 영역에 들어간다.&#x20;
  * 작업을 수행한다.&#x20;
  * 락을 반납한다.&#x20;
* 여기서 락을 획득하고 반납하는 과정이 계속 반복된다. 10000번의 연산이 있다면 10000번의 연산 모두 같은 과정을 반복한다.&#x20;
* 이렇듯 락을 사용하는 방식은 직관적이지만 상대적으로 무거운 방식이다.&#x20;

### CAS&#x20;

* 이런 문제를 해결하기 위해 락을 걸지 않고 원자적인 연산을 수행할 수 있는데, 이것을 CAS(Compare-And-Swap) 연산이라고 한다. 이 방법은 락을 사용하지 않기 때문에 락 프리(lock-free) 기법이라고 한다.&#x20;
* 참고로 CAS 연산은 락을 완전히 대체하는 것은 아니고, 작은 단위의 일부 영역에 적용할 수 있다. **기본은 락을 사용하고, 특별한 경우에 CAS 를 적용할 수 있다고 생각하면 된다.**&#x20;
* 아래 코드를 살펴보자&#x20;
  * `compareAndSet(0, 1)`&#x20;
    * `atomicInteger` 가 가지고 있는 값이 현재 0이면 이 값을 1로 변경하라는 매우 단순한 메서드이다.&#x20;
    * 만약 `atomicInteger` 가 가지고 있는 값이 현재 0이면 `atomicInteger` 의 값이 1로 변경되고, `true` 를 반환한다.&#x20;
    * 반대의 경우, `atomicInteger` 의 값은 변경되지 않는다. 이 경우 `false` 를 반환한다.&#x20;
  * 여기서 가장 중요한 내용이 있는데, **이 메서드는 원자적으로 실행된다는 점이다!** \
    ( 연산 자체가 나뉘어져 있는데? 원자적으로 실행? -> 아래 그에 대한 해답이 나온다)&#x20;

```java
package thread.cas;

import java.util.concurrent.atomic.AtomicInteger;

public class CasMainV1 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        System.out.println("start value = " + atomicInteger.get());

        boolean result1 = atomicInteger.compareAndSet(0, 1);
        System.out.println("result1 = " + result1 + "; value = " + atomicInteger.get());    // 1

        boolean result2 = atomicInteger.compareAndSet(0, 1);
        System.out.println("result2 = " + result2 + "; value = " + atomicInteger.get());    // 1
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.23.46.png" alt=""><figcaption></figcaption></figure>

### 실행 순서 분석&#x20;

#### CAS - 성공 케이스&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.28.32.png" alt="" width="494"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.37.19.png" alt="" width="432"><figcaption></figcaption></figure>

* 여기서는 `AtomicInteger` 내부에 있는 `value` 값이 0 이라면 1로 변경하고 싶다.&#x20;
* `compareAndSet(0, 1)` 을 호출한다. 매개변수의 왼쪽이 기대하는 값, 오른쪽이 변경하는 값이다.&#x20;
* CAS 연산은 메모리에 있는 값이 기대하는 값이라면 원하는 값으로 변경한다.&#x20;
* 메모리에 있는 `value` 값이 0 이므로 1 로 변경할 수 있다.&#x20;
* **그런데 생각해보면 이 명령어는 2개로 나누어진 명령어이다. 따라서 원자적이지 않은 연산처럼 보인다.**&#x20;
  * 먼저 메인 메모리에 있는 값을 확인한다.&#x20;
  * 해당 값이 기대하는 값(0) 이라면 원하는 값(1) 으로 변경한다.&#x20;

> **CPU 하드웨어의 지원**&#x20;
>
> * CAS 연산은 이렇게 원자적이지 않은 두 개의 연산을 CPU 하드웨어 차원에서 특별하게 하나의 원자적인 연산으로 묶어서 제공하는 기능이다. **이것은 소프트웨어가 제공하는 기능이 아니라, 하드웨어가 제공하는 기능이다.** 대부분의 현태 CPU 들은 CAS 연산을 위한 명령어를 제공한다.&#x20;
> * **CPU 는 다음 두 과정을 묶어서 하나의 원자적인 명령으로 만들어버린다. 따라서 중간에 다른 스레드가 개입할 수 없다.**&#x20;
>   * x001 의 값을 확인한다.&#x20;
>   * 읽은 값이 0이면 1로 변경한다.&#x20;
> * **CPU 는 두 과정을 하나의 원자적인 명령으로 만들기 위해 1번과 2번 사이에 다른 스레드가 x001 의 값을 변경하지 못하게 막는다.**&#x20;
>   * 참고로 1,2 번 사이의 시간은 CPU 입장에서 보면 정말 찰나의 시간이다. 그래서 성능에 큰 영향을 끼치지 않는다.&#x20;
>   * CPU 가 1초에 얼마나 많은 연산을 수행하는지에 대해서 생각해보면 이해가 될 것이다.&#x20;

#### CAS - 실패 케이스&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.38.12.png" alt="" width="479"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.38.31.png" alt="" width="436"><figcaption></figcaption></figure>

## CAS 연산2&#x20;

* 어떤 값을 하나 증가시키는 `value++` 연산은 원자적인 연산이 아니다.&#x20;
* 때문에, `value++` 연산을 여러 스레드에서 사용한다면 락을 건 다음에 값을 증가해야 한다.&#x20;
* CAS 연산을 활용해서 락 없이 값을 증가하는 기능을 만들어보자.&#x20;
  * `AtomicInteger` 가 제공한는, `IncrementAndGet()` 메서드가 어떻게 CAS 연산을 활용해서 락 없이 만들어졌는지 직접 구현해보자.&#x20;
* 코드를 살펴보자&#x20;
  * CAS 연산을 사용하면 여러 스레드가 같은 값을 사용하는 상황에서도 락을 걸지 않고, 안전하게 값을 증가할 수 있다. 여기서는 락을 걸지 않고 CAS 연산을 사용해서 값을 증가했다.&#x20;
    * `getValue = atomicInteger.get()` 을 사용해서 `value` 값을 읽는다.&#x20;
    * `compareAndSet(getValue, getValue + 1)` 을 사용해서, 방금 읽은 `value` 값이 메모리의 `value` 값과 같다면 `value` 값을 하나 증가한다. 여기서 CAS 연산이 사용된다.&#x20;
    * 만약 CAS 연산이 성공한다면, `true` 를 반환하고, `do-while` 문을 빠져나온다.&#x20;
    * 만약 CAS 연산이 실패한다면, `false` 를 반환하고, `do-while` 문을 다시 시작한다.&#x20;
  * 지금은 `main` 스레드 하나로 순서대로 실행되기 때문에, CAS 연산이 실패하는 상황을 볼 수 없다. 우리가 기대하는 실패하는 상황은 연산의 중간에 다른 스레드가 값을 변경해버리는 것이다. 멀티스레드로 실행해서 CAS 연산이 실패하는 경우 어떻게 작동하는지 알아보자.&#x20;

```java
package thread.cas;

import java.util.concurrent.atomic.AtomicInteger;

import static util.MyLogger.log;

public class CasMainV2 {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger();
        System.out.println("start value = " + atomicInteger.get());

        // incrementAndGet 구현
        int resultValue1 = incrementAndGet(atomicInteger);
        System.out.println("resultValue1 = " + resultValue1);

        int resultValue2= incrementAndGet(atomicInteger);
        System.out.println("resultValue1 = " + resultValue2);
    }

    private static int incrementAndGet(AtomicInteger atomicInteger) {
        int getValue;
        boolean result;
        do{
            getValue = atomicInteger.get();
            log("getValue = " + getValue);
            result = atomicInteger.compareAndSet(getValue, getValue+1);
            log("result = " + result);
        }while (!result);

        return getValue+1;
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-11-12 12.43.23.png" alt=""><figcaption></figcaption></figure>

## CAS 연산3&#x20;

* 멀티스레드를 사용해서 중간에 다른 스레드가 먼저 값을 증가시켜 버리는 경우를 알아보자. 그리고 CAS 연산이 실패하는 경우에 어떻게 되는지 알아보자.&#x20;
* 코드를 살펴보자&#x20;
  * **\[Thread-1 실행]**&#x20;
    * `atomicInteger.get()` 을 사용해서 `value` 값을 읽는다 -> `getValue` 는 0이다.&#x20;
    * `compareAndSet(0, 1)` 을 수행한다.&#x20;
      * `compareAndSet(getValue, getValue + 1)`&#x20;
    * CAS 연산이 성공했으므로 `value` 값은 0에서 1로 증가하고 `true` 를 반환한다.&#x20;
    * `do~while` 문을 빠져나간다.&#x20;
  * **\[Thread-0 실행]**
    * **\[Thread-0]`do~while` 첫 번째 시도**
      * `atomicIngeter.get()` 을 사용해서 `value` 값을 읽는다. -> `getValue` 는 0이다.&#x20;
      * `compareAndSet(0, 1)` 을 수행한다.&#x20;
        * `compareAndSet(getValue, getValue + 1)`&#x20;
      * 그런데 `compareAndSet(0, 1)` 연산은 실패한다.&#x20;
        * CAS 연산에서 현재 `value` 값으로 0 을 기대했지만, `Thread-1` 이 중간에 먼저 실행되면서 `value` 값을 `0 -> 1`로 변경해버렸다.&#x20;
      * CAS 연산이 실패앴으므로 `value` 값은 변경하지 않고, `false` 를 반환한다.&#x20;
      * 실패했으므로, `do~while` 문을 빠져나가지 못한다. `do~while` 문을 다시 시작한다.&#x20;
        * `while(!result)` -> `while(!false)` -> `while(true)` 이므로 반복&#x20;
    * **\[Thread-0] `do~while` 두번째 시도**&#x20;
      * `do~while` 문이 다시 시작된다.&#x20;
      * `atomicInteger.get()` 을 사용해서 `value` 값을 읽는다. -> `getValue` 는 1이다.&#x20;
      * `compareAndSet(1, 2)` 을 수행한다.&#x20;
        * `compareAndSet(getValue, getValue + 1)`&#x20;
      * CAS 연산이 성공했으므로 `value` 값은 1에서 2로 증가하고 `true` 를 반환한다.&#x20;
      * `do~while` 문을 빠져나간다.&#x20;

```java
package thread.cas;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.atomic.AtomicInteger;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class CasMainV3 {

    private static final int THREAD_COUNT = 2;

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicInteger = new AtomicInteger(0);
        System.out.println("start value = " + atomicInteger.get());

        Runnable runnable = new Runnable() {

            @Override
            public void run() {
                incrementAndGet(atomicInteger);
            }
        };

        List<Thread> threads = new ArrayList<>();
        for (int i = 0; i < THREAD_COUNT; i++) {
            Thread thread = new Thread(runnable);
            threads.add(thread);
            thread.start();
        }

        for (Thread thread : threads) {
            thread.join();
        }

        int result = atomicInteger.get();
        System.out.println(atomicInteger.getClass().getSimpleName() + " resultValue : " + result);

    }

    private static int incrementAndGet(AtomicInteger atomicInteger) {
        int getValue;
        boolean result;
        do{
            getValue = atomicInteger.get();
//            sleep(100); // 스레드 동시 실행을 위한 대기
            log("getValue = " + getValue);
            result = atomicInteger.compareAndSet(getValue, getValue+1);
            log("result = " + result);
        }while (!result);

        return getValue+1;
    }
}

```

<figure><img src="../../../../.gitbook/assets/image (177).png" alt=""><figcaption></figcaption></figure>

### 정리&#x20;

* `AtomicInteger` 가 제공하는 `incrementAndGet()` 코드도 앞서 우리가 직접 작성한 `incrementAndGet()` 코드와 똑같이 CAS 를 활용하도록 되어 있다. CAS 를 사용하면 락을 사용하지 않지만, 대신에 다른 스레드가 값을 먼저 증가해서 문제가 발생하는 경우 루프를 돌며 재시도를 하는 방식을 사용한다.&#x20;
* 이 방식은 다음과 같이 동작한다.&#x20;
  * 현재 변수의 값을 읽어온다.&#x20;
  * 변수의 값을 1 증가시킬 때, 원래 값이 같은지 확인한다. (CAS 연산 사용 - 하드웨어)&#x20;
  * 동일하다면 증가된 값을 변수에 저장하고 종료한다.&#x20;
  * 동일하지 않다면 다른 스레드가 값을 중간에 변경한 것이므로, 다시 처음으로 돌아가 위 과정을 반복한다.&#x20;
* 두 스레드가 동시에 실행되면서 문제가 발생하는 상황을 스레드가 충돌했다고 표현한다.&#x20;
* 이 과정에서 충돌이 발생할 때마다, 다시 시도하므로, 결과적으로 락 없이 데이터를 안전하게 변경할 수 있다.&#x20;
  * CAS 를 사용하는 방식은 충돌이 드물게 발생하는 환경에서는 락을 사용하지 않으므로, 높은 성능을 발휘할 수 있다.
  * 이는 락을사용하는방식과 비교했을 때, 스레드가 락을 획득하기 위해 대기하지 않기 때문에 대기 시간과 오버헤드가 줄어드는 장점이 있다.&#x20;
* 하지만, 충돌이 빈번하게  발생하는 환경에서는 성능에 문제가 발생할 수 있다 .
  * 여러 스레드가 자주 동시에 동일한 변수의 값을 변경하려고 시도할 때, CAS 는 자주 실패하고 재시도해야 하므로 성능 저하가 발생할 수 있다.&#x20;
  * 이런 상황에서는 반복문을 계속 돌기 때문에, CPU 자원을 많이 소모하게 된다.&#x20;

### CAS(Compare-And-Swap) 와 락(Lock) 방식의 비교&#x20;

#### 락(Lock) 방식&#x20;

* 비관적(pessimistic) 접근법
* 데이터에 접근하기 전에 항상 락을 획득&#x20;
* 다른 스레드의 접근을 막음&#x20;
* **"다른 스레드가 방해할 것이다" 라고 가정**&#x20;

#### CAS(Compare-And-Swap) 방식&#x20;

* 낙관적(optimistic) 접근법&#x20;
* 락을 사용하지 않고 데이터에 바로 접근&#x20;
* 충돌이 발생하면 그때 재시도&#x20;
* **"대부분의 경우 충돌이 없을 것이다" 라고 가정**&#x20;

#### 정리&#x20;

* **충돌이 많이 없는 경우에는 CAS 연산이 빠른 것을 확인할 수 있다.**
* 그럼 충돌이 많이 발생하지 않는 연산은 무엇이 있을까?&#x20;
  * **사실 간단한 CPU 연산은 너무 빨리 처리되기 때문에, 충돌이 자주 발생하지 않는다. 충돌이 발생하기 전에 이미 연산을 완료하는 경우가 더 많다.**&#x20;
*   앞서 여러 스레드가 `value++` 연산을 수행했던 `BasicInteger`, `VolatileInteger` 의 예를 살펴보자.

    <figure><img src="../../../../.gitbook/assets/image (178).png" alt=""><figcaption></figcaption></figure>

    * **`BasicInteger` 의 실행 결과를 보면 최대한 스레드를 충돌하게 만들었는데도, 1000 개 중 50개의 스레드만 충돌한 사실을 확인할 수 있다.**&#x20;
      * 위 예제에서는 최대한 많이 충돌하게 하기 위해서 1000개의 스레드를 동시에 쉬게 만든 다음에 동시에 실행했다.&#x20;
    * 락 방식&#x20;
      * 스레드 충돌을 방지하기 위해서 1000개의 스레드가 모두 락을 획득하고 반환하는 과정을 거친다.&#x20;
      * 락을 사용하기 때문에, 1000개의 스레드는 순서대로 하나씩 수행된다.&#x20;
      * 사실 이 중에 스레드가 충돌하는 경우는 50개의 경우 뿐이다.&#x20;
    * CAS 방식&#x20;
      * 1000개의 스레드를 모두 한번에 실행한다.&#x20;
      * 그리고 충돌이 나는 50개의 경우만 재시도 한다.&#x20;

## CAS 락 구현1&#x20;

* **`synchronized`, `Lock` 을 사용하지 않고 락을 구현하면 어떤 문제가 생길까?**&#x20;
* **실제로 락이 걸리지 않는 것을 확인할 수 있다.**&#x20;
* **그 이유는 다음 두 부분이 원자적이지 않다는 문제가 있다.**&#x20;
  * **락 사용 여부 확인**
  * **락의 값 변경**&#x20;
* 때문에, 이 두개의 연산은 하나의 스레드만 실행해야 한다. 이를 위해 보통은 `synchronized`, `Lock` 을 사용해서 두 코드를 동기화해 안전한 임계 영역을 만들어내야 한다.&#x20;
* **여기서 다른 방안이 있는데, 바로 두 코드를 하나로 묶어서 원자적으로 처리하는 것이다.**
  * CAS 연산을 사용하면 두 연산을 하나로 묶어서 하나의 원자적인 연산으로 처리할 수 있다.&#x20;
  * 락의 사용 여부를 확인하고, 그 값이 기대하는 값과 같다면 변경하는 것이다. 이것은 CAS 연산과 들어맞는다.&#x20;

<pre class="language-java"><code class="lang-java">package thread.cas.spinlock;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class SpinLockBad {

   private volatile boolean lock = false;
   
<strong>   public void lock() {
</strong>      log("락 획득 시도");
      while(true) {
         if (!lock) { // 1. 락 사용 여부 확인
            sleep(100); // 문제 상황 확인용, 스레드 대기
            lock = true; // 2. 락의 값 변경
            break; // while 탈출
         } else {
            // 락을 획득할 때 까지 스핀 대기(바쁜 대기) 한다.
            log("락 획득 실패 - 스핀 대기");
         }
      }
      log("락 획득 완료");
   }
   
   public void unlock() {
      lock = false;
      log("락 반납 완료");
   }
}
</code></pre>

## CAS 락 구현2&#x20;

* 이번에는 CAS 를 사용해서 락을 구현해보자.&#x20;
* 아래코드를 살펴보자.&#x20;
  * **CAS 연산을 지원하는 `AtomicBoolean` 을 사용했다.**&#x20;
  * **이를 통해 다음 두 연산을 하나로 만들었다.**
    * **락 사용 여부 확인**
    * **락의 값 변경**&#x20;
* **아래 코드는 동시성 문제가 발생하지 않는다!**

<pre class="language-java"><code class="lang-java">package thread.cas.spinlock;

<strong>import java.util.concurrent.atomic.AtomicBoolean;
</strong>import static util.MyLogger.log;

public class SpinLock {

   private final AtomicBoolean lock = new AtomicBoolean(false);
   
   public void lock() {
      log("락 획득 시도");
      while (!lock.compareAndSet(false, true)) {
         // 락을 획득할 때 까지 스핀 대기(바쁜 대기) 한다.
         log("락 획득 실패 - 스핀 대기");
      }
      log("락 획득 완료");
   }
   
   public void unlock() {
      lock.set(false);
      log("락 반납 완료");
   }
}
</code></pre>

### 기존 코드와 비교&#x20;

* 기존 코드는 바로 다음 두 부분이 원자적이지 않다는 문제가 있었다.&#x20;
  * **락 사용 여부 확인**&#x20;
  * **락의 값 변경**&#x20;

```java
while(
   if (!lock) { //1. 락 사용 여부 확인
      lock = true; //2. 락의 값 변경
   }
)
```

* 이 문제를 CAS 연산을 통해서 원자적으로 변경하였다.&#x20;
  * **락을 사용하지 않는다면 락의 값을 변경**&#x20;

```java
while(lock.compareAndSet(false, true)) {}
```

* 원자적인 연산은 스레드 입장에서 쪼갤 수 없는 하나의 연산이다. 따라서 여러 스레드가 동시에 실행해도 안전하다. 이렇게 CAS 를 이용해 원자적인 연산을 만든 덕분에 무거운 동기화 작업 없이 아주 가벼운 락을 만들 수 있다.&#x20;
* 동기화 락을 사용하는 경우 스레드가 락을 획득하지 못하면 `BLOCKED`, `WAITING` 등으로 상태가 변한다. 그리고 또 대기 상태의 스레드를 깨워야 하는 무겁고 복잡한 과정이 추가로 들어간다. 따라서 성능이 상대적으로 느릴 수 있다. 반면 CAS 를 활용한 락 방식은 사실 락이 없다. 단순히 `while` 문을 반복할 뿐이다. 따라서 대기하는 스레드도 `RUNNABLE` 상태를 유지하면서 가볍고 빠르게 작동할 수 있다.&#x20;

### CAS 단점&#x20;

* CAS 단점은 무엇이 있을까?&#x20;
* 바로 여러 스레드가 락을 획득하려고 하면 락을 기다리는 스레드가 CPU 를 계속 사용하면서 대기하는 것이다.
  * `BLOCKED`, `WAITING` 상태의 스레드는 CPU 를 거의 사용하지 않지만,&#x20;
  * `RUNNABLE` 상태로 `while` 문을 반복 실행하는 방식은 CPU 자원을 계속해서 사용하는 것이다.&#x20;
* 동기화 락을 사용하면 `RUNNABLE` 상태의 스레드가 `BLOCKED`, `WAITING` 상태에서 다시 `RUNNABLE` 상태로 이동한다. 이 사이에 CPU 자원을 거의 사용하지 않을 수 있다.&#x20;
* 그래서 동기화 락을 사용하는 방식보다 스레드를 `RUNNABLE` 로 살려둔 상태에서 계속 락 획득을 반복 체크하는 것이 더 효율적인 경우에만 이런 방식을 사용해야 한다.&#x20;
  * 이 방식은 스레드의 상태가 변경되지 않기 때문에, 매우 빠르게 락을 획득하고, 또 실행할 수 있다는 장점이 있다.&#x20;
* 그럼 어떤 경우에 CAS 락 방식이 효율적일까?&#x20;
  * 안전한 임계 영역이 필요하지만, 연산이 길지 않고 매우매우매우! 짧게 끝날 때 사용해야 한다.&#x20;
  * **예를 들어, 숫자 값의 증가, 자료 구조의 데이터 추가와 같이 CPU 사이클이 금방 끝나는 연산에 사용하면 효과적이다.** \
    **(CPU 바운드)**
  * **반면에, 데이터베이스의 결과를 데기하거나, 다른 서버의 요청을 기다린다거나 하는 것 처럼 오래 기다리는 작업에 대해서는  최악의 결과가 발생할 수 있다.** \
    **(CPU 를 사용해 계속 락 획득 여부를 체크한다 - IO 바운드)**&#x20;

#### 스핀 락&#x20;

* 스레드가 락이 해제되기를 기다리면서 반복문을 통해 계속해서 확인하는 모습이 마친 제자리에서 회전하는 것처럼 보인다. 그래서 이런 방식을 "스핀 락" 이라고도 부른다.&#x20;
  * 그리고 이런 방식에서 스레드가 락을 획득 할 때 까지 대기하는 것을 스핀 대기(spin-wait) 또는 CPU 자원을 계속 사용하면서 바쁘게 대기한다고 해서 바쁜 대기(busy-wait) 라 한다.&#x20;
* 이런 스핀 락 방식은 아주 짧은 CPU 연산을 수행할 때 사용해야 효율적이다. 잘못 사용하면 오히려 CPU 자원을 더 많이 사용할 수 있다.&#x20;

## 총정리&#x20;

### CAS

#### CAS 장점

* **낙관적 동기화**&#x20;
  * 락을 걸지 않고도 값을 안전하게 업데이트 할 수 있다.&#x20;
  * CAS 는 충돌이 자주 발생하지 않을 것이라고 가정한다. 이는 충돌이 적은 환경에서 높은 성능을 발휘한다.&#x20;
* **락 프리**
  * CAS 는 락을 사용하지 않기 때문에, 락을 획득하기 위해 대기하는 시간이 없다. 따라서 스레드가 블로킹되지 않으며, 병렬 처리가 더 효율적일 수 있다.

CAS 단점

* **충돌이 빈번한 경우**&#x20;
  * 여러 스레드가 동시에 동일한 변수에 접근하여 업데이트를 시도할 때 충돌이 발생할 수 있다.&#x20;
  * 충돌이 발생하면 CAS 는 루프를 돌며 재시도해야하며, 이에 따라 CPU 자원을 계속 소모할 수 있다. \
    반복적인 시도로 인해 오버헤드가 발생할 수 있다.&#x20;
* **스핀락과 유사한 오버헤드**&#x20;
  * CAS 충돌 시 반복적인 재시도를 하므로, 이 과정이 계속 반복되면 스핀락과 유사한 성능 저하가 발생할 수 있다. 특히 충돌 빈도가 높을수록 이런 현상이 두드러진다.&#x20;

### 동기화 락 (synchronized, Lock)&#x20;

동기화 락 장점&#x20;

* 충돌 관리&#x20;
  * 락을 사용하면 하나의 스레드만 리소스에 접근할 수 있으므로 충돌이 발생하지 않는다. 여러 스레드가 경쟁할 경우에도 안정적으로 동작한다 .
* 안정성&#x20;
  * 복잡한 상황에서도 락은 일관성 있는 동작을 보장한다.&#x20;
* 스레드 대기&#x20;
  * 락을 대기하는 스레드는 CPU 를 거의 사용하지 않는다.&#x20;

동기화 락 단점&#x20;

* &#x20;락 획득 대기 시간&#x20;
  * 스레드가 락을 획득하기 위해 대기해야 하므로, 대기 시간이 길어질 수 있다.&#x20;
* 컨텍스트 스위칭 오버헤드&#x20;
  * 락을 사용하면, 락 획득을 대기하는 시점과 또 락을 획득하는 시점에 스레드의 상태가 변경된다. 이때 컨텍스트 스위칭이 발생할 수 있으며, 이로 인해 오버헤드가 증가할 수 있다.&#x20;

### 결론&#x20;

* 일반적으로 동기화 락을 사용하고, 아주 특별한 경우에만 CAS 를 사용해서 최적화해야 한다.&#x20;
* CAS 를 통한 최적화가 더 나은 경우는 스레드가 `RUNNABLE` -> `BLOCKED`, `WAITING` 상태에서 다시 `RUNNABLE` 상태로 가는 것 보다는 스레드를 `RUNNABLE` 로 살려둔 상태에서 계속 락 획득을 반복체크하는 것이 더 효율적일 때만 사용해야 한다.&#x20;
* 하지만 이 경우 대기하는 스레드가 CPU 자원을 계속 소모하기 때문에, 대기 시간이 아주아주아주 짧아야 한다! 따라서 임계 영역이 필요하지만, 연산이 길지 않고 매우 짧게짧게 끝날 때 사용해야 한다.&#x20;
  * 예를 들어, 숫자 값의 증가, 자료 구조의 데이터 추가,  삭제와 같은 CPU 바운드 연산에 대해서 원자적 연산이 필요한 경우에만 사용해야 한다.&#x20;
  * 반면의 IO 바운드 연산에 CAS 를 사용하면 CPU 를 사용해 기다리는 최악의 결과가 발생할 수 있다. 이러한 경우에는 동기화 락을 사용해야 한다.&#x20;
* 또한 CAS 는 충돌 가능성이 낮은 환경에서는 매우 효율적이지만, 충돌 가능성이 높은 환경에서는 성능 저하가 발생할 수 있다. 이런 경우네는 상황에 맞는 적절한 동기화 전략을 사용하는 것이 중요하다.&#x20;
  * 때로는 락이 더 나은 성능을 발휘할 수 있으며, CAS 가 항상 더 빠르다고 단정할 수는 없다.&#x20;
* 따라서, 각 접근 방식의 특성을 이해하고, 애플리케이션의 특정 요구사항과 환경에 맞는 방식을 선택하는 것이 중요하다.&#x20;
* 우리가 사용하는 자바 동시성 라이브러리들, 동기화 컬렉션들은 성능 최적화를 위해서 CAS 를 적극 활용한다. 덕분에 실무에서 CAS 를 직접 사용하는 일은 드물다.. 대신 CAS 를 사용해 최적화되어 있는 라이브러리들을 이해하고 편리하게 사용할 줄 알면 충분하다.&#x20;
