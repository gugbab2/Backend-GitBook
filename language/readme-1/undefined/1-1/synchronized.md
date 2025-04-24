# 동기화 - synchronized

## 1. 출금 예제 - 시작&#x20;

멀티스레드를 사용할 때 가장 주의해야 할 점은, 같은 자원(리소스)에 여러 스레드가 동시에 접근할 때 발생하는 동시성  문제이다.

* 참고로 여러 스레드가 접근하는 자원을 공유 자원이라 한다. 대표적인 공유 자원은 인스턴트 필드(멤버변수) 이다.&#x20;

**멀티스레드를 사용할 때는 이런 공유 자원에 대한 접근을 적절하게 동기화(synchronized) 해서 동시성 문제가 발생하지 않게 방지하는 것이 중요하다.**

#### 아래 출금 예제를 살펴보자.

```java
package thread.sync;

public interface BankAccount {
     boolean withdraw(int amount);
     int getBalance();
}
```

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV1 implements BankAccount {

    volatile private int balance;

    public BankAccountV1(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

        // step1. 검증 
        // 잔고가 출금액 보다 적으면, 진행 불가
        log("[검증 시작] : " + amount + ", 잔액 : " + balance);
        if (balance < amount) {
            log("[검증 실패] : " + amount + ", 잔액 : " + balance);
            return false;
        }

        // step2. 출금 
        // 잔고가 출금액 보다 많으면, 진행
        log("[검증 완료] : " + amount + ", 잔액 : " + balance);
        sleep(1000);    // 출금에 걸리는 시간으로 가정
        balance -= amount;
        log("[출금 완료] : " + amount + ", 잔액 : " + balance);

        log("거래 종료");
        return true;
    }

    @Override
    public int getBalance() {
        return balance;
    }
}
```

```java
package thread.sync;

public class WithDrawTask implements Runnable {

    private BankAccount account;
    private int amount;

    public WithDrawTask(BankAccount account, int amount) {
        this.account = account;
        this.amount = amount;
    }

    @Override
    public void run() {
        account.withdraw(amount);
    }
}
```

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BackMain {
    public static void main(String[] args) throws InterruptedException {
        BankAccount account = new BankAccountV1(1000);

        Thread t1 = new Thread(new WithDrawTask(account, 800), "t1");
        Thread t2 = new Thread(new WithDrawTask(account, 800), "t2");

        t1.start();
        t2.start();

        sleep(500);
        log("t1 state : " + t1.getState());
        log("t2 state : " + t2.getState());

        t1.join();
        t2.join();

        log("최종 잔액 : " + account.getBalance());
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-16 22.02.54.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-16 22.03.57.png" alt=""><figcaption></figcaption></figure>

### 동시성 문제 확인

위 시나리오는 악의적인 사용자가 2대의 PC 에서 동시에 같은 계좌의 돈을 출금한다고 가정한다.

* `t1`, `t2` 스레드는 거의 동시에 실행되지만, 아주 약간의 차이로 `t1` 스레드가 먼저 실행되고, `t2` 스레드가 그 다음에 실행된다고 가정하자.&#x20;
* 처음 계좌의 잔액은 1000 원이다. `t1` 스레드가 800원 출금하면 잔액은 200원 남는다.&#x20;
* 이제 계좌의 잔액은 200원이다. `t2` 스레드가 800원 출금하면 잔액보다더 많은 돈을 출금하게 되므로 출금에 실패해야 한다.&#x20;
* **그런데 실행 결과를 보면 기대와는 다르게 t1, t2 는 각각 800원씩 총 1600원 출금에 성공한다...**

## 2. 동시성 문제&#x20;

### t1, t2 순서로 실행 가정&#x20;

위 케이스와 같이 `t1`, `t2` 스레드가 미묘한 차이로 실행된다고 생각해보자.

우리는 아래 로직에서 잔고가 출금액 보다 적으면 출금을 진행하지 않을 것이라 생각했다.&#x20;

```java
// step1. 검증 
// 잔고가 출금액 보다 적으면, 진행 불가
log("[검증 시작] : " + amount + ", 잔액 : " + balance);
if (balance < amount) {
    log("[검증 실패] : " + amount + ", 잔액 : " + balance);
    return false;
}
```

하지만 2개의 요청이 거의 같은 시점에 들어왔다면?

**`t1` 이 검증 로직을 통과하고 잔액을 줄이기도 전에, t2 가 검증 로직을 확인한 것이다.** \
**이렇게 되면 `t1`, `t2` 모두 검증을 통과한 상태가 되고 출금액을 2번 인출하는 사태가 발생한다...**&#x20;

* **최종결과 -600원**

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-16 22.28.54.png" alt=""><figcaption></figcaption></figure>

### t1, t2 동시 실행 가정&#x20;

이번에는 `t1`, `t2` 스레드가 정확히 동시점에 실행된다고 생각해보자.&#x20;

위 케이스와 마찬가지로 검증 로직을 통해 중복 출금은 되지 않을 것이라고 생각했다.&#x20;

**하지만, 정확히 동시점에 실행되는 경우는 2개의 스레드가 동시에 검증을 통과하고,** \
**2개의 스레드가 동시에 출금을 진행해 1번의 출금만 이루어진 것처럼 보여질 수 있다...**&#x20;

* **최종결과 200원**
* **사실 해당 경우가, 더 큰일이다! 왜냐면 이전의 경우는 2번의 인출이 이루어진 것이기 때문에 찾아갈 수는 있지만,** \
  **해당 경우는 찾아가기가 매우 어렵다 ;;**

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-16 22.29.46.png" alt=""><figcaption></figcaption></figure>

## 3. 임계 영역(critical section)

**이런 문제가 발생한 근본 원인은 여러 스레드가 함께 공유하는 공유 자원을 여러 단계로 나누어 사용하기 때문이다.**

* **검증 단계 : 잔액이 출금액보다 많은지 확인한다.**
* **출금 단계 : 잔액을 출금액만큼 줄인다.**&#x20;

#### 이 로직에는 하나의 큰 가정이 있다.

스레드 하나의 관점에서 **"출금"** 을 보면, 검증 단계에서 확인한 잔액 1000원은 \
출금 단계에서 출금 직전까지 같은 1000원으로 유지되어야 한다.

* 그래야 검증 단계에서 확인한 금액으로, 출금 단계에서 정확한 잔액을 계산할 수 있다.

**결국 여기서는 내가 사용하는 값이 중간에 변경되지 않을 것이라는 가정이 있다!**

그런데 만약 중간에, 다른 스레드가 잔액의 값을 변경한다면 큰 혼란이 발생한다. \
1000원이라 생각한 잔액이 다른 값으로 변경되면 잔액이 전혀 다른 값으로 계산될 수 있다.

#### 공유 자원

`balance` 변수는 여러 스레드가 함께 사용하는 공유 자원이다.

출금 로직을 수행하는 중간에 다른 스레드에서 이 값을 얼마든지 변경할 수 있다.&#x20;

**따라서 다른 스레드가 출금 메서드를 호출하면서, 사용중인 출금 값을 중간에 변경해버릴 수 있다.**&#x20;

#### 한 번에 하나의 스레드만 실행

만약 "출금" 이라는 메서드를 한 번에 하나의 스레드만 실행할 수 있게 제한 한다면 어떻게 될까?&#x20;

이렇게 하면 공유 자원인 `balance` 를 한 번에 하나의 스레드만 변경할 수 있다.&#x20;

**따라서 계산 중간에 다른 스레드가 `balance` 의 값을 변경하는 부분을 걱정하지 않아도 된다.**&#x20;

### 임계 영역(critical section)&#x20;

**여러 스레드가 동시에 접근하면 데이터 불일치나 예상치 못한 동작이 발생할 수 있는 위험한 코드 영역을 의미한다.**&#x20;

* 여러 스레드가 동시에 접근해서는 안 되는 공유 자원(특히 돈 관련..) 을 접근하거나 수정하는 부분을 의미한다.&#x20;
  * ex) 공유 변수나 공유 객체를 수정

앞서 우리가 살펴본 "출금" 로직이 바로 임계 영역이다.

* 더 자세히는 출금을 진행할 때 잔액(`balance`) 을 검증하는 단계부터 잔액의 계산을 완료할 때 까지가 임계 영역이다.&#x20;
* 여기서 잔액(`balance`) 은 여러 스레드가 동시에 접근해서는 안되는 공유 자원이다.
* 이런 임계 영역은 한 번의 하나의 스레드만 접근할 수 있도록 안전하게 보호해야 한다.&#x20;

**여러가지 방법이 있지만 `synchonrized` 키워드를 통해 아주 간단하게 임계 영역을 보호할 수 있다.**&#x20;

## 4. synchronized 메서드&#x20;

자바의 `synchronized` 키워드를 사용하면 한 번에 하나의 스레드만 실행할 수 있는 코드 구간을 만들 수 있다.&#x20;

아래 코드를 살펴보자.

* `BankAccountV1` 코드와 크게 달라진 부분은 없다. `withdraw()`, `getBalance()` 코드에 `synchronized` 키워드가 추가되었다.&#x20;
* 실행 결과를 보면 `t1` 이 `withdraw()` 메서드를 시작부터 끝까지 완료하고 나서, \
  그 다음에 `t2` 가 `withdraw()` 메서드를 수행하는 것을 확인할 수 있다.&#x20;

```java
package thread.sync;

import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV2 implements BankAccount {

    private int balance;

    public BankAccountV2(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public synchronized boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

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

        log("거래 종료");
        return true;
    }

    @Override
    public synchronized int getBalance() {
        return balance;
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 20.06.04.png" alt=""><figcaption></figcaption></figure>

### synchronized 분석

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 20.10.04.png" alt=""><figcaption></figcaption></figure>

모든 객체(인스턴스) 는 내부에 자신만의 락(lock) 을 가지고 있다.&#x20;

* 모니터 락(monitor lock) 이라고도 부른다.
* 객체 내부에 있고 우리가 확인하기는 어렵다.

**스레드가 `synchronized` 키워드가 있는 메서드에 진입하려면 반드시 메서드가 속한 인스턴스의 락이 있어야 한다!**

* 여기서는 `BankAccount(x001)` 인스턴스의 `synchonized withdraw()` 메서드를 호출하므로 이 인스턴스의 락이 필요하다.

스레드 `t1`, `t2` 는 `withdraw()` 를 실행하기 직전이다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 20.12.53.png" alt=""><figcaption></figcaption></figure>

#### 메서드 호출 시나리오를 확인해보자. (t1 이 먼저 실행된다고 가정하겠다)

1. 스레드 `t1` 이 먼저 `synchonized` 키워드가 있는 `withdraw()` 메서드를 호출한다.&#x20;
2. `synchronized` 메서드를 호출하려면 먼저 해당 인스턴스의 락이 필요하다.
   1. `BankAccount(x001)` 인스턴스에 락이 있으므로 스레드 `t1` 은 락을 획득한다.&#x20;
   2. 스레드 `t1` 은 해당 인스턴스 락을 획득했기 때문에, `withdraw()` 메서드에 진입할 수 있다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-17 20.14.26.png" alt=""><figcaption></figcaption></figure>

3. 스레드 `t2` 도 `withdraw()` 메서드 호출을 시도한다. `synchronized` 메서드를 호출하려면 먼저 해당 인스턴스의 락이 필요하다.&#x20;
   1. 스레드 `t2` 는 `BankAcoount(x001)` 인스턴스에 있는 락 획득을 시도한다. 하지만 락이 없다.&#x20;
   2. 이렇게 락이 없으면 `t2` 스레드는 락을 획들할 때까지 `BLOCKED` 상태로 대기한다.
   3. `t2` 스레드의 상태는 `RUNNABLE` -> `BLOCKED` 상태로 변하고, 락을 획들할 때 까지 무한정 대기한다.\
      (**참고로 `BLOCKED` 상태가 되면 다시 락을 획득하기 전까지 계속 대기하고, CPU 스케줄링에 들어가지 않는다)**
4. `t1` 스레드가 모든 작업을 수행하고 락을 반납하게 되면 `t2` 스레드는 락을 획득하고 `withdraw()` 메서드를 실행하게 된다.&#x20;

#### 결과&#x20;

* `t1` : 800원 출금 완료&#x20;
* `t2` : 잔액 부족으로 출금 실패&#x20;
* 원금 1000원, 최종 잔액은 200원&#x20;

> #### 참고 - 락을 획득하는 순서는 보장되지 않는다.&#x20;
>
> 만약 `withdraw()` 메서드를 수 많은 스레드가 동시에 호출한다면, 1개의 스레드만 락을 획득하고 나머지는 `BLOCKED` 상태가 된다. 그리고 이후에 `BankAccount(x001)` 인스턴스에 락을 반납하면, 해당 인스턴스 락을 기다리는 수 많은 스레드 중 하나의 스레드만 락을 획득하고, 락을 획득한 스레드만 `RUNNABLE` 상태가 된다.
>
> * 이때, 어떤 순서로 락을 획득하는지는 자바 표준에 정의되어 있지 않다. 따라서 순서를 보장하지 않고, 환경에 따라서 순서가 달라질 수 있다.&#x20;
> * `volatile` 를 사용하지 않아도, `synchronized` 안에서 접근하는 변수의 메모리 가시성 문제는 해결된다.\
>   (happend-before)

## 5. synchronized 코드 블럭&#x20;

`synchronized` 의 가장 큰 장점이자 단점은 한 번에 하나의 스레드만 실행할 수 있다는 점이다.&#x20;

* 여러 스레드가 동시에 실행하지 못하기 때문에, 전체로 보면 성능이 떨어질 수 있다.&#x20;

**따라서 `synchronized` 를 통해 여러 스레드를 동시에 실행할 수 없는 구간은 꼭! 필요한 곳으로 한정해서 설정해야 한다.**

#### 아래 코드를 살펴보자.&#x20;

`synchronized` 블럭을 통해서 락인 정말로 필요한 임계 영역에만 설정할 수 있다.&#x20;

```java
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class BankAccountV3 implements BankAccount {

    private int balance;

    public BankAccountV3(int initialBalance) {
        this.balance = initialBalance;
    }

    @Override
    public boolean withdraw(int amount) {
        log("거래 시작 : " + getClass().getSimpleName());

        synchronized (this) {
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
        }

        log("거래 종료");
        return true;
    }

    @Override
    public synchronized int getBalance() {
        return balance;
    }
}

```

## 6. 정리&#x20;

### synchronized 장점&#x20;

* **프로그래밍 언어에 문법으로 제공**&#x20;
* **아주 편리한 사용**&#x20;
* **자동 잠금 해제**
  * &#x20;`synchronized` 메서드나 블록이 완료되면 자동으로 락을 대기중인 다른 스레드의 잠금이 해제 된다.&#x20;
  * 개발자가 직접 특정 스레드를 깨우도록 관리해야 한다면, 매우 어렵고 번거로울 것이다.

### synchronized 단점&#x20;

* **무한 대기**
  * `BLOCKED` 상태의 스레드는 락이 풀릴 때 까지 무한 대기한다.
  * 특정 시간까지만 대기하는 타임아웃 X
  * 중간에 인터럽트 X
* **공정성**&#x20;
  * 락이 돌아왔을 때 `BLOCKED` 상태의 여러 스레드 중에 어떤 스레드가 락을 획득할 지 알 수 없다.&#x20;
  * 최악의 경우 특정 스레드가 너무 오랜기간 락을 획득하지 못할 수 있다.

`synchronized` 의 치명적인 단점은 락을 얻기 위해 `BLOCKED` 상태가 되면 락을 얻을때까지 무한 대기한다는 점이다.&#x20;

예를 들어서 웹 애플리케이션의 경우 고객이 어떤 요청을 했는데, 화면이 계속 요청 중만 뜨고, 응답을 받지 못하는 것이다.

* 차라리 너무 오랜 시간이 지나면, 시스템 사용자가 너무 많아서 다음에 다시 시도해 달라고 하는 식의 응답을 주는 것이 더 나은 선택일 것이다.

결국 더 유연하고, 더 세밀한 제어가 가능한 방법들이 필요하게 되었다. 이런 문제를 해결하기 위해서 자바 1.5부터 `java.util.concurrent` 라는 동시성 문제 해결을 위한 패키지가 추가되었다.&#x20;

하지만 단순하고 편리하게 락을 사용하고자 한다면 `synchronized` 를 사용하자.&#x20;
