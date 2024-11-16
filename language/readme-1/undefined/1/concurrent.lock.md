# 고급 동기화 - concurrent.Lock

## 1. LockSupport1

* `synchronized` 는 자바 1.0 부터 제공되는 매우 편한 기능이지만, 다음과 같은 한계가 있다.
* **`synchronized` 단점**&#x20;
  * **무한 대기 : `BLOCKED` 상태의 스레드는 락이 풀릴 때 까지 무한 대기한다.**&#x20;
    * **특정 시간까지만 대기하는 타임아웃 X**&#x20;
    * **중간에 인터럽트 X**&#x20;
  * **공정성 : 락이 들어왔을 때 `BLOCKED` 상태의 여러 스레드 중에 어떤 스레드가 락을 획득할 지 알 수 없다. 최악의 경우 특정 스레드가 너무 오랜시간 락을 획득하지 못할 수 있다.**&#x20;
* 결국 더 유연하고, 세밀한 제어가 가능한 방법들이 필요하게 되었다.&#x20;
* 이런 문제를 해결하기 위해서 자바 1.5 부터 `java.util.concurrent` 라는 동시성 문제 해결을 위한 라이브러리 패키지가 추가된다.&#x20;
* 이 라이브러리에는 수 많은 클래스가 있지만, 가장 기본이 되는 `LockSupport` 에 대해서 먼저 알아보자.&#x20;
  * `LockSupport` 을 통해서 `synchronized` 의 가장 큰 단점인 무한 대기 문제를 해결할 수 있다.

### LockSupport 기능&#x20;

* **`LockSupport` 는 스레드를 `WAITING` 상태로 변경한다.**
  * **`synchronized` 가 `BLOCKED` 상태로 변경하는 것과 가장 큰 차이점이다.**&#x20;
* `WAITING` 상태는 누가 깨워주기 전까지는 계속 대기한다. 그리고 CPU 스케줄링에 들어가지 않는다.&#x20;
* `LockSupport` 의 대표적인 기능은 다음과 같다.&#x20;
  * `park()` : 스레드를 `WAITING` 상태로 변경한다.&#x20;
  * `parkNanos(nanos)` : 스레드를 나노초 동안만 `TIME_WAITING` 상태로 변경한다.
    * 지정한 나노초가 지나가면, `TIME_WAITING` 상태에서 빠져나오고 `RUNNABLE` 상태로 변경된다.&#x20;
  * `unpark(thread)` : `WAITING` 상태의 대상 스레드를 `RUNNABLE` 상태로 변경한다.&#x20;

```java
import java.util.concurrent.locks.LockSupport;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class LockSupportMainV1 {

    public static void main(String[] args) {
        Thread thread1 = new Thread(new ParkTest(), "Thread-1");
        thread1.start();

        // 잠시 대기사여 Thread-1 이 park 에 빠질 시간을 준다.
        sleep(100);
        log("Thread-1 state : " + thread1.getState());

        log("main -> unpark(Thread-1)");
        LockSupport.unpark(thread1);    // 1. unpark 사용
//        thread1.interrupt();    // 2. interrupt() 사용
    }

    static class ParkTest implements Runnable {

        @Override
        public void run() {
            log("park 시작");
            LockSupport.park();
            log("park 종료, state : " + Thread.currentThread().getState());
            log("인터럽트 상태 : " + Thread.currentThread().isInterrupted());
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 22.42.58.png" alt=""><figcaption></figcaption></figure>

### 인터럽트 사용&#x20;

* **`WAITING` 상태의 스레드에 인터럽트가 발생하면 `WAITING` 상태에서 `RUNNABLE` 상태로 변하면서 깨어난다.**&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 22.44.40.png" alt=""><figcaption></figcaption></figure>

## 2. LockSupport2&#x20;

### 시간 대기&#x20;

* 이번에는 스레드를 특정 시간 동안만 대기하는 `parkNanos(nanos)` 를 호출하자&#x20;

```java
import java.util.concurrent.locks.LockSupport;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class LockSupportMainV2 {

    public static void main(String[] args) {
        Thread thread1 = new Thread(new ParkTest(), "Thread-1");
        thread1.start();

        // 잠시 대기하여 Thread-1 이 park 에 빠질 시간을 준다.
        sleep(100);
        log("Thread-1 state : " + thread1.getState());

    }

    static class ParkTest implements Runnable {

        @Override
        public void run() {
            log("park 시작");
            LockSupport.parkNanos(2000_000000); // parkNanos 사용
            log("park 종료, state : " + Thread.currentThread().getState());
            log("인터럽트 상태 : " + Thread.currentThread().isInterrupted());
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 22.46.16.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 22.46.28.png" alt=""><figcaption></figcaption></figure>

### BLOCKED vs WAITING

* `WAITING` 상태에 특정 시간까지만 대기하는 기능이 포함된 것이 `TIMED_WAITING` 이다. 여기서는 둘을 묶어서 `WAITING` 상태로 표현하겠다.&#x20;

#### 인터럽트&#x20;

* `BLOCKED` 상태는 인터럽트가 걸려도 대기 상태를 빠져나오지 못한다. 여전히 `BLOCKED` 상태이다.
  * `synchronized` 는 `BLOCKED` 가 된다.&#x20;
* `WAITING`, `TIMED_WAITING` 상태는 인터럽트가 걸리면 대기 상태를 빠져나오고, `RUNNABLE` 상태로 변한다.&#x20;

#### 용도&#x20;

* `BLOCKED` 상태는 자바의 `synchronized` 에서 락을 획득하기 위해 대기할 때 사용된다.&#x20;
* `WAITING`, `TIME_WAITING` 상태는 스레드가 특정 조건이나 시간 동안 대기할 때 발생하는 상태이다.&#x20;
* `WAITING` 상태는 다양한 상황에서 사용된다.&#x20;
  * `Thread.join()`, `LockSupport.park()`, `Object.wait()` 와 같은 메서드 호출 시 `WAITING` 상태가 된다.&#x20;
* `TIMED_WAITING` 상태는 다양한 상황에서 사용된다.&#x20;
  * `Thread.join(millis)`, `LockSupport.parkNanos(nanos)`, `Object.wait(timeout)` 와 같은 메서드 호출 시 `TIMED_WAITING` 상태가 된다.&#x20;

**`BLOCKED`, `WAITING`, `TIMED_WAITING` 상태 모두 스레드가 대기하며, 실행 스케줄링에 들어가지 않기 때문에, CPU 입장에서 보면 실행하지 않은 것과 비슷한 상태이다.**&#x20;

* **`BLOCKED` 상태는 `synchronized` 에서만 사용하는 특별한 대기 상태라고 이해하면 된다.**&#x20;
* **`WAITING`, `TIME_WAITING` 상태는 범용적으로 활용할 수 있는 대기 상태라고 이해하면 된다.**

### LockSupport 정리&#x20;

* `LockSupport` 를 사용하면 스레드를 `WAITING`, `TIME_WAITING` 상태로 변경할 수 있고, 또 인터럽트를 받아서 스레드를 깨울 수도 있다.&#x20;
* 이런 기능들을 잘 활용하면  `synchronized` 의 단점인 무한 대기 문제를 해결할 수 있을 것 같다.&#x20;
* 물론 그냥 되는 것은 아니고 `LockSupport` 를 활용해서 안전한 임계 영역을 만드는 어떤 기능을 개발해야 한다.&#x20;
* 예를 들면 아래와 같다.&#x20;

```java
if (!lock.tryLock(10초)) { // 내부에서 parkNanos() 사용 
    log("[진입 실패] 너무 오래 대기했습니다.");
    return false;
}

//임계 영역 시작
...
//임계 영역 종료

lock.unlock() // 내부에서 unpark() 사용
```

* 하지만 이런 기능을 직접 구현하기는 매우 어렵다.&#x20;
  * 예를 들어 스레드 10개를 동시에 실행했는데, 그중에 딱 1개의 스레드만 락을 가질 수 있도록 락 기능을 만들어야 한다.&#x20;
  * 그리고 나머지 9개의 스레드가 대기해야 하는데, 어떤 스레드가 대가하고 있는지 알수 있는 자료구조를 만들어야 하고,&#x20;
  * 대기 중인 스레드 중에 어떤 스레드를 깨울지에 대한 우선순위 결정도 필요하다.&#x20;
* **한마디로 `LockSupport` 는 너무 저수준이다. `synchronized` 처럼 고수준의 기능이 필요하다.**&#x20;

## 3. ReentrantLock - 이론&#x20;

* 자바 1.0 부터 존재한 `synchronized` 와 `BLOCKED` 상태를 통한 임계 영역 관리의 한계를 극복하기 위해서, \
  자바 1.5 부터 `Lock` 인터페이스와 `ReentrantLock` 구현체를 제공한다.&#x20;
* **synchronized 의 단점**&#x20;
  * **무한대기**&#x20;
  * **공정성**&#x20;

#### Lock 인터페이스&#x20;

* `Lock` 인터페이스는 다음과 같은 메서드를 제공한다. 대표적인 구현체로 `ReentrantLock` 이 있다.&#x20;
* 결론적으로 아래의 메서드들을 사용하면 고수준의 동기화 기법을 구현할 수 있다.&#x20;
* `Lock` 인터페이스는 `synchronized` 블록보다 더 많은 유연성을 제공하며, 특히 락을 특정 시간 만큼만 사용하거나, 인터럽트 가능한 락을 사용할 때도 유용하다!

> #### 주의!&#x20;
>
> * **여기서 사용하는 락은 객체 내부에 있는 모니터 락이 아니다!**
> * **`Lock` 인터페이스와 `ReentrantLock` 이 제공하는 기능이다!**&#x20;
> * **모니터 락과 `BLOCKED` 상태는 `synchronized` 에서만 사용된다.**&#x20;

<pre class="language-java"><code class="lang-java">package java.util.concurrent.locks;
<strong>
</strong><strong>public interface Lock {
</strong>     void lock();
     void lockInterruptibly() throws InterruptedException;
     boolean tryLock();
     boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
     void unlock();
     Condition newCondition();
}
</code></pre>

* `void lock()`
  * 락을 획득한다. 만약 다른 스레드가 이미 락을 획득했다면, 락이 풀릴 때까지 현재 스레드는 대기(`WAITING`) 한 다.&#x20;
  * **이 메서드는 인터럽트에 응답하지 않는다.**&#x20;
    * ex) 맛집에 한번 줄을 서면 끝까지 기다린다. 친구가 다른 맛집을 찾았다고 중간에 연락해도 포기하지 않고 기다린 다.
* `void lockInterruptibly()`
  * **락 획득을 시도하되, 다른 스레드가 인터럽트할 수 있도록 한다.**&#x20;
  * 만약 다른 스레드가 이미 락을 획득했다면, 현재 스레드는 락을 획득할 때까지 대기한다.&#x20;
  * **대기 중에 인터럽트가 발생하면 `InterruptedException` 이 발생하면 락 획득을 포기한다.**
    * ex) 맛집에 한번 줄을 서서 기다린다. 다만 친구가 다른 맛집을 찾았다고 중간에 연락하면 포기한다.
* `boolean tryLock()`
  * 락 획득을 시도하고, 즉시 성공 여부를 반환한다.&#x20;
  * **만약 다른 스레드가 이미 락을 획득했다면 `false` 를 반환하고, 그렇지 않으면 락을 획득하고 `true` 를 반환한다.**
  * **바로 락 획득 결과를 확인할 수 있기 때문에, 인터럽트와는 연관이 없다.**&#x20;
    * ex) 맛집에 대기 줄이 없으면 바로 들어가고, 대기 줄이 있으면 즉시 포기한다.
* `boolean tryLock(long time, TimeUnit unit)`
  * 주어진 시간 동안 락 획득을 시도한다.&#x20;
  * **주어진 시간 안에 락을 획득하면 `true` 를 반환한다. 주어진 시간이 지나도 락을 획득하지 못한 경우 `false` 를 반환한다.**&#x20;
  * **이 메서드는 대기 중 인터럽트가 발생하면`InterruptedException` 이 발생하며 락 획득을 포기한다.**&#x20;
    * ex) 맛집에 줄을 서지만 특정 시간 만큼만 기다린다. 특정 시간이 지나도 계속 줄을 서야 한다면 포기한다. 친구가 다른 맛집을 찾았다고 중간에 연락해도 포기한다.
* `void unlock()`
  * 락을 해제한다. 락을 해제하면 락 획득을 대기 중인 스레드 중 하나가 락을 획득할 수 있게 된다.
  * **락을 획득한 스레드가 호출해야 하며, 그렇지 않으면 `IllegalMonitorStateException` 이 발생할 수 있다.**
    * ex) 식당안에 있는 손님이 밥을 먹고 나간다. 식당에 자리가 하나 난다. 기다리는 손님께 이런 사실을 알려주어야 한다. 기다리던 손님중 한 명이 식당에 들어간다.

### 공정성&#x20;

* `Lock` 인터페이스가 제공하는 다양한 기능 덕분에 `synchronized` 의 단점인 무한 대기 문제가 해결 되었다.&#x20;
* 그런데 공정성에 대한 문제는 남아있다.&#x20;
* `Lock` 인터페이스의 대표적인 구현체로 `ReentrantLock` 이 있는데, 이 클래스는 스레드가 공정하게 락을 얻을 수 있는 모드를 제공한다.&#x20;
  * `ReentrantLock` 은 락 공정 모드와 비공정 모드로 설정할 수 있으며, 이 두 모드는 락을 획득하는 방식에서 차이가 있다.&#x20;

```java
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockEx { 
    // 비공정 모드 락
    private final Lock nonFairLock = new ReentrantLock(); 
    // 공정 모드 락
    private final Lock fairLock = new ReentrantLock(true);
    
    public void nonFairLockTest() {
         nonFairLock.lock();
         try {
             // 임계 영역 
         } finally {
             nonFairLock.unlock();
         }
    }
    public void fairLockTest() {
         fairLock.lock();
         try {
             // 임계 영역
         } finally {
             fairLock.unlock();
         }   
    }
}
```

### 비공정 모드&#x20;

* **비공정 모드는 `ReentrantLock` 의 기본 모드이다.**&#x20;
* 이 모드에서는 락을 먼저 요청한 스레드가 락을 먼저 획득한다는 보장이 없다. 대기 중인 스레드 중 아무나 락을 획득할 수 있다.&#x20;
* 이는 락을 빨리 획득할 수 있지만, 특정 스레드가 장기간 락을 획득하지 못할 가능성도 있다.&#x20;

#### 비공정 모드 특징

* **성능 우선 : 락 획득 속도가 빠르다.**
* **선점 가능 : 새로운 스레드가 기존 대기 스레드보다 먼저 락을 획들할 수 있다.**&#x20;
* **기아 현상 가능성 : 특정 스레드가 계속해서 락을 획득하지 못할 수 있다.**&#x20;

### 공정 모드

* 공정 모드는 생성자에 `true` 를 전달하면 된다.&#x20;
* 공정 모드는 락을 요청한 순서대로 스레드가 락을 획득할 수 있게 한다.&#x20;
* 이는 먼저 대기한 스레드가 먼저 락을 획득하게 되어 스레드 간의 공정성을 보장한다.&#x20;

#### 공정 모드 특징&#x20;

* **공정성 보장 : 대기 큐에서 먼저 대기한 스레드가 락을 먼저 획득한다.**&#x20;
* **기아 현상 방지 : 모든 스레드가 언젠가 락을 획득할 수 있게 보장된다.**&#x20;
* **성능 저하 : 락을 획득하는 속도가 느려질 수 있다.**&#x20;

#### 비공정, 공정 모드 정리&#x20;

* **비공정 모드는 성능을 중시하고, 스레드가 락을 빨리 획득할 수 있지만, 특정 스레드가 계속해서 락을 획득하지 못할 수 있다.**&#x20;
* **공정 모드는 스레드가 락을 획득하는 순서를 보장하여 공정성을 중시하지만, 성능이 저하될 수 있다.**&#x20;

## 4. ReentrantLock - 활용&#x20;

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV4 implements BankAccount {

    private int balance;
    private final Lock lock = new ReentrantLock();

    public BankAccountV4(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

        lock.lock();    // ReentrantLock 이용하여 lock 을 걸기
        try{
            // 잔고가 출금액 보다 적으면, 진행 불가
            log("[검증 시작] : " + amount + ", 잔액 : " + balance);
            if (balance < amount) {
                log("[검증 실패] : " + amount + ", 잔액 : " + balance);
                return false;
            }

            // 잔고가 출금액 보다 많으면, 진행
            log("[검증 완료] : " + amount + ", 잔액 : " + balance);
            sleep(1000);    // 출금에 걸리는 시간으로 가정
            balance -= amount;
            log("[출금 완료] : " + amount + ", 잔액 : " + balance);
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }

        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        lock.lock();    // ReentrantLock 이용하여 lock 을 걸기
        try{
            return balance;
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 23.29.01.png" alt=""><figcaption></figcaption></figure>

## 5. ReentrantLock - 대기 중단&#x20;

#### `tryLock` 예시&#x20;

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV5 implements BankAccount {

    private int balance;
    private final Lock lock = new ReentrantLock();

    public BankAccountV5(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

        if(!lock.tryLock()) {
            log("[진입 실패] 이미 처리중인 작업이 있습니다.");
            return false;
        }

        try{
            // 잔고가 출금액 보다 적으면, 진행 불가
            log("[검증 시작] : " + amount + ", 잔액 : " + balance);
            if (balance < amount) {
                log("[검증 실패] : " + amount + ", 잔액 : " + balance);
                return false;
            }

            // 잔고가 출금액 보다 많으면, 진행
            log("[검증 완료] : " + amount + ", 잔액 : " + balance);
            sleep(1000);    // 출금에 걸리는 시간으로 가정
            balance -= amount;
            log("[출금 완료] : " + amount + ", 잔액 : " + balance);
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }

        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        lock.lock();    // ReentrantLock 이용하여 lock 을 걸기
        try{
            return balance;
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 23.31.31.png" alt=""><figcaption></figcaption></figure>

#### `tryLock(시간)` 예시

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV6 implements BankAccount {

    private int balance;
    private final Lock lock = new ReentrantLock();

    public BankAccountV6(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

        try {
            if(!lock.tryLock(500, TimeUnit.MILLISECONDS)) {
                log("[진입 실패] 이미 처리중인 작업이 있습니다.");
                return false;
            }
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }

        try{
            // 잔고가 출금액 보다 적으면, 진행 불가
            log("[검증 시작] : " + amount + ", 잔액 : " + balance);
            if (balance < amount) {
                log("[검증 실패] : " + amount + ", 잔액 : " + balance);
                return false;
            }

            // 잔고가 출금액 보다 많으면, 진행
            log("[검증 완료] : " + amount + ", 잔액 : " + balance);
            sleep(1000);    // 출금에 걸리는 시간으로 가정
            balance -= amount;
            log("[출금 완료] : " + amount + ", 잔액 : " + balance);
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }

        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        lock.lock();    // ReentrantLock 이용하여 lock 을 걸기
        try{
            return balance;
        } finally {
            lock.unlock();  // ReentrantLock 이용하여 lock 해제
        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 23.31.51.png" alt=""><figcaption></figcaption></figure>
