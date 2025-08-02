# 메모리 가시성

## 1. volatile, 메모리 가시성1

메모리 가시성을 이해하기 위해서 간단한 예제를 살펴보자.

* `task.flag` 를 `false` 로 변경하는 순간, `while` 문을 빠져나오기를 바라지만, 실제로는 반영되지 않는 것을 확인할 수 있다.&#x20;

```java
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class VolatileCountMain {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread t = new Thread(task);
        t.start();

        sleep(1000);

        task.flag = false;
        log("flag = " + task.flag + ", count " + task.count + " in main()");
    }

    static class MyTask extends Thread {

        boolean flag = true;
        long count;

//        volatile boolean flag = true;
//        volatile long count;

        @Override
        public void run() {
            while (flag) {
                count++;
                if(count % 100_000_000 == 0){
                    log("flag = " + flag + ", count " + count + " in while()");
                }
            }
            log("flag = " + flag + ", count " + count + " 종료");
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.09.40.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.10.26.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.10.08.png" alt=""><figcaption></figcaption></figure>

## 2. volatile, 메모리 가시성2

### 일반적으로 우리가 생각하는 메모리 접근 방식

1. 자바 프로그램을 실행하고, `main` 스레드와 `work` 스레드 모두 메인 메모리의 `runFlag` 값을 읽는다.
2. 프로그램 시작 시점에는 `runFlag` 값을 변경하지 않기 때문에, 모든 스레드에서 `true` 값을 읽는다.
3. 그 이후, `runFlag` 값을 `false` 로 설정하고 `while` 문을 빠져나오는 동작 방식을 생각했을 것이다.

하지만, 우리가 생각한 흐름대로 동작하지 않는다..

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.11.41.png" alt="" width="519"><figcaption></figcaption></figure>

### 실제 메모리의 접근 방식&#x20;

CPU는 처리 성능을 개선하기 위해서 중간에 **"캐시 메모리"** 라는 것을 사용한다.

메인 메모리는 CPU 입장에서 보면 거리도 멀고, 속도도 상대적으로 느리다. 대신에 상대적으로 가격이 저렴해서 \
큰 용량을 쉽게 구할 수 있다.

CPU 연산은 매우 빠르기 때문에, CPU 연산을 따라가려면, CPU 가까이에 매우 빠른 메모리가 필요한데, \
이것이 바로 캐시 메모리이다.

* 캐시 메모리는 CPU 와 가까이 붙어 있고, 속도도 매우 빠른 편이다. 하지만 상대적으로 가격이 비싸기 때문에, \
  큰 용량을 구성하기가 어렵다.

현재의 CPU 대부분은 코어 단위로 캐시 메모리를 각각 보유하고 있다.

* 참고로 여러 코어가 공유하는 캐시 메모리도 있다.

각 스레드가 `runFlag` 값을 사용하면 CPU 는 이 값을 효율적으로 처리하기 위해서 먼저 `runFlag` 를 캐시 메모리에 \
불러온다. 그리고 이후에는 캐시 메모리에 있는 `runFlag` 를 사용하게 된다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.15.25.png" alt="" width="479"><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.19.01.png" alt="" width="487"><figcaption></figcaption></figure>

여기서 차이점이 발생하는데, 우리가 생각하는 것과 다르게 동작하게 되는데, `runFlag` 값을 `false` 로 변경하였을 때, 우리는 바로 해당 값이 메모리에 반영되고 그 값을 읽어올 것이라고 생각한다.&#x20;

**하지만, 실제로는 변경 된 값이 캐시 메모리에 반영이 되고, 읽어오는 동작 또한 메모리에서 읽어오는 것이 아닌 캐시** \
**메모리에서 읽어오게 된다.**

**결국, 메인 메모리에는 반영이 되지도 않고, 메인 메모리에서 값을 읽어오지 않을수도 있다는 것이다!**

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.20.37.png" alt=""><figcaption></figcaption></figure>

그렇다면, 캐시 메모리에 있는 `runFlag` 값은 언제 메모리에 반영될까?&#x20;

* **이 부분의 대한 정답은 "알 수 없다" 이다. CPU 설계 방식과 종류에 따라서 다르다.** \
  (극단적으로 생각하면 평생 반영되지 않을 수도 있다 ;; )

메인 메모리에 반영이 된다고 해도 문제는, 여기서 끝이 아니다.\
**메인 메모리에 반영된** `runFlag` **값을** `work` **스레드가 사용하는 캐시 메모리에 다시 불러와야 한다.**

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.24.58.png" alt=""><figcaption></figcaption></figure>

메인 메모리에 변경된 `runFlag` 값이 언제 CPU 코어2의 캐시 메모리에 반영될까?

* **이 부분에 대한 정답도 "알 수 없다" 이다. CPU 설계 방식과 종류에 따라 다르다.**\
  (극단적으로 보면 평생 반영되지 않을수도 있다 ;;)

언젠가 CPU 코어2의 캐시 메모리에 `runFlag` 값을 불러오게 되면 `work` 스레드가 확인하는 `runFlag` 값이 `false` 가 되므로, `while` 문을 탈출하게 된다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.24.25.png" alt=""><figcaption></figcaption></figure>

그렇다면, 대부분 언제 캐시 메모리를 메인 메모리에 반영하거나, 메인 메모리의 변경 내역을 캐시 메모리에 다시 불러오는 동작이 이루어질까?

* **이 부분 역시, CPU 설계 방식과 실행 환경에 따라서 다르다.**\
  (즉시 반영될 수도 있고, 몇 밀리초 후에 될 수도 있고, 몇초 후가 될 수도 있다)
* 주로 컨텍스트 스위칭이 될 때, 메모리도 함께 갱신되는데 이 부분도 환경에 따라서 달라질 수 있다.

### 메모리 가시성(memory visibillity)

이처럼 멀티스레드 환경에서 한 스레드가 변경한 값이 다른 스레드에서 언제 보이는지에 대한 문제를 **메모리 가시성(memory visibillity)** 이라고 한다.

그렇다면 한 스레드에서 변경한 값이 다른 스레드에서 즉시 보이게 하려면 어떻게 해야 할까?

## 3. volatile, 메모리 가시성3

캐시 메모리를 사용하면 CPU 처리 성능을 개선할 수 있다. 하지만 때로는 이런 성능 향상 보다는,\
여러 스레드에서 같은 시점에 정확히 같은 데이터를 보는 것이 더 중요할 수 있다.

**해결방안은 아주 단순하다. 성능을 포기하는 대신, 값을 읽을 때, 값을 쓸 때 모두 메인 메모리에 직접 접근하면 된다.**\
**(물론, 성능에서의 손해는 감수해야 한다!)**

**자바에서는 `volatile` 키워드가 이런 기능을 제공한다.**&#x20;

#### 아래 코드를 살펴보자

바뀐 것은 `flag`, `count` 변수에 `volatile` 키워드를 붙여준 것뿐이다.

물론 `volatile` 키워드가 장점만 있는 것은 아니다.&#x20;

* **변수를 사용할 때마다 메인 메모리에서 값을 읽어오고, 메인 메모리에 값을 변경해주어야 하기 때문에,** \
  **비용이 더 많이 들 수 밖에 없다.**
* **때문에, 꼭 필요한 상황에서만 사용해야 한다.**

```java
import static util.MyLogger.log;
import static util.ThreadUtils.sleep;

public class VolatileCountMain {
    public static void main(String[] args) {
        MyTask task = new MyTask();
        Thread t = new Thread(task);
        t.start();

        sleep(1000);

        task.flag = false;
        log("flag = " + task.flag + ", count " + task.count + " in main()");
    }

    static class MyTask extends Thread {

//        boolean flag = true;
//        long count;

        volatile boolean flag = true;
        volatile long count;

        @Override
        public void run() {
            while (flag) {
                count++;
                if(count % 100_000_000 == 0){
                    log("flag = " + flag + ", count " + count + " in while()");
                }
            }
            log("flag = " + flag + ", count " + count + " 종료");
        }
    }
}

```

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.34.35.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-10 22.34.47.png" alt=""><figcaption></figcaption></figure>

## 4. 자바 메모리 모델(Java Memory Model)

### 메모리 가시성(memory visibillity)

멀티스레드 환경에서 한 스레드가 변경한 값이 다른 스레드에서 언제 보이는지에 대한 것을 메모리 가시성이라고 한다.\
이름 그대로 메모리에 변경한 값이 보이는가 보이지 않는가의 문제이다.

### Java Memoey Model

Java Memoey Model(JMM) 은 자바 프로그램이 어떻게 메모리에 접근하고 수정할 수 있는지를 규정하며, \
특히 멀티스레드 프로그래밍에서 스레드 간의 상호작용을 정의한다.

JMM 에서도 여러가지 내용이 있지만, \
**핵심은 스레드들의 작업 순서를 보장하는 happends-before 관계에 대한 정의이다.**

### happens-before

happens-before 관계는 자바 메모리 모델에서 스레드 간의 작업 순서를 정의하는 개념이다.&#x20;

만약 A 작업이 B 작업보다 happens-before 관계에 있다면, A 작업에서의 모든 메모리 변경 사항은 B 작업에서 볼 수 있다. 즉, A 작업에서 변경된 내용은 B 작업이 시작되기 전에 모두 메모리에 반영된다.&#x20;

* happens-before 관계는 이름 그대로, 한 동작이 다른 동작보다 먼저 발생함을 보장한다.
* happens-before 관계는 스레드 간의 메모리 가시성을 보장하는 규칙이다.
* happens-before 관계가 성립하면, 한 스레드의 작업을 다른 스레드에서 볼 수 있게 된다.
* 즉, 한 스레드에서 수행한 작업을 다른 스레드가 참조할 때 최신 상태가 보장되는 것이다.

이 규칙을 따르면 프로그래머가 멀티스레드 프로그램을 작성할 때 예상치 못한 동작을 피할 수 있다.

### heppens-before 관계가 발생하는 경우&#x20;

#### 프로그램 순서 규칙&#x20;

* 단일 스레드 내에서, 프로그램이 순서대로 작성된 모든 명령문은 happens-before 순서로 실행된다.
* 예를 들어, `int a=1;intb=2;` 에서 `a=1` 은 `b=2` 보다 먼저 실행된다.\
  -> 당연한 말이다 ;;&#x20;

#### volatile 변수 규칙&#x20;

* 한 스레드에서 volatile 변수에 대한 쓰기 작업은 해당 변수를 읽는 모든 스레드에 보이도록 한다.&#x20;
* 즉, `volatile` 변수에 대한 쓰기 작업은 그 변수를 읽는 작업보다 happens-before 관계를 형성한다.&#x20;

#### 스레드 시작 규칙

* 한 스레드에서 `Thread.start()` 를 호출하면, 해당 스레드 내의 모든 작업은 `start()` 호출 이후에 실행된 \
  작업보다 happens-before 관계가 성립한다.
* 아래 코드에서 `start()` 호출 이전에 수행된 모든 작업은 새로운 스레드가 시작된 후의 작업보다 happens-before 관계를 가진다.

```java
Thread t = new Thread(task);
t.start();
```

#### 스레드 종료 규칙&#x20;

* 한 스레드에서 `Thread.join()` 을 호출하면, join 대상 스레드의 모든 작업은 `join()` 이 반환된 후의 작업보다 happens-before 관계를 가진다. 예를 들어, `thread.join()` 호출 전에 `thread` 의 모든 작업이 완료되어야 하며, 이 작업은 `join()` 이 반환된 후에 참조 가능하다. `1 ~ 100` 까지 값을 더하는 `sumTask` 예시를 떠올려보자.

#### 인터럽트 규칙&#x20;

* 한 스레드에서 `Thread.interrupt()` 를 호출하는 작업이, 인터럽트된 스레드가 인터럽트를 감지하는 시점의 작업 보다 happens-before 관계가 성립한다. 즉, `interrupt()` 호출 후, 해당 스레드의 인터럽트 상태를 확인하는 작업 이 happens-before 관계에 있다. 만약 이런 규칙이 없다면 인터럽트를 걸어도, 한참 나중에 인터럽트가 발생할 수 있 다.

#### 객체 생성 규칙&#x20;

* 객체의 생성자는 객체가 완전히 생성된 후에만 다른 스레드에 의해 참조될 수 있도록 보장한다. 즉, 객체의 생성자에서 초기화된 필드는 생성자가 완료된 후 다른 스레드에서 참조될 때 happens-before 관계가 성립한다.

#### 모니터 락 규칙&#x20;

* 한 스레드에서 `synchronized` 블록을 종료한 후, 그 모니터 락을 얻는 모든 스레드는 해당 블록 내의 모든 작업을 볼 수 있다. 예를 들어, `synchronized(lock) { ... }` 블록 내에서의 작업은 블록을 나가는 시점에 happens- before 관계가 형성된다. 뿐만 아니라 `ReentrantLock` 과 같이 락을 사용하는 경우에도 happens-before 관계가 성립한다.
