# 스레드 생성과 실행

## 1. 스레드 생성

### 스레드 생성 - Thread 상속&#x20;

```java
public class HelloThread extends Thread{
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " : run()");
    }
}

// ...

public class HelloThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + ": main() start");
        HelloThread helloThread = new HelloThread();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 전");
        helloThread.start();
        System.out.println(Thread.currentThread().getName() + ": start() 호출 후");
        System.out.println(Thread.currentThread().getName() + ": main() end");
    }
}

// 실행 결과
main: main() start 
main: start() 호출 전 
main: start() 호출 후
Thread-0: run()
main: main() end
```

위 코드의 실행 결과를 보게 되면 스레드 생성 전후 메모리 변화가 아래와 같이 일어나는 것을 확인할 수 있다.&#x20;

* 메서드를 실행하면 스택프레임을 스택에 올리면서 시작한다.&#x20;

> #### 스레드 간의 실행 순서는 얼마든지 달라질 수 있다.&#x20;
>
> * CPU 코어가 2개여서 물리적으로 정말 동시에 실행될 수 있고, 하나의 CPU 코어에서 시간을 나누어 실행될 수도 있다.&#x20;
> * 그리고 한 스레드가 얼마나 오랜기간 실행되는지도 보장하지 않는다. 한 스레드가 먼저 다 수행한 후 다른 스레드가 수행될 수도 있고, 둘이 번갈아 가면서 수행할 수도 있다.&#x20;
> * 스레드는 순서와 실행 기간을 모두 보장하지 않는다.&#x20;
> * 이것이 멀티 스레드이다!

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-09 13.10.35.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-09 13.11.09.png" alt=""><figcaption></figcaption></figure>

### 데몬 스레드&#x20;

스레드는 사용자 스레드와 데몬 스레드 2 종류로 구분할 수 있다.&#x20;

#### 사용자 스레드

* 프로그램의 주요 작업을 수행한다.
* 작업이 완료될 때가지 실행된다.
* 모든 사용자 스레드가 종료되면 JVM 도 종료된다.

#### 데몬 스레드

* 백그라운드에서 보조적인 작업을 수행한다.
* 모든 사용자 스레드가 종료 되면 데몬 스레드는 자동으로 종료된다.

```java
public class DaemonThreadMain {
    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + " :  main() start");

        DaeminThread daeminThread = new DaeminThread();
        daeminThread.setDaemon(true);
        daeminThread.start();

        System.out.println(Thread.currentThread().getName() + " :  main() end");
    }

    static class DaeminThread extends Thread {

        @Override
        public  void run(){
            System.out.println(Thread.currentThread().getName() + " : run()");

            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }

            System.out.println(Thread.currentThread().getName() + " : end()");

        }
    }
}
```

### 스레드 생성 - Runnable 상속&#x20;

* 스레드를 만들 때는 `Thread` 클래스를 상속 받는 방법과 `Runnable` 인터페이스를 구현하는 방법이 있다.

```java
public class HelloRunnable implements Runnable {

    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName() + " : run()");

    }
}

public class HelloRunnableMain {

    public static void main(String[] args) {
        System.out.println(Thread.currentThread().getName() + " :  main() start");

        HelloRunnable runnable = new HelloRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        System.out.println(Thread.currentThread().getName() + " :  main() end");
    }
}

// 실행 결과
 main: main() start
 main: main() end
 Thread-0: run()
```

### Thread 상속 vs Runnable 구현&#x20;

쓰레드를 사용할 때는 `Runnable` 인터페이스를 구현하는 방법을 사용하자.&#x20;

#### Thread 상속&#x20;

장점&#x20;

* 간단한 구현

단점&#x20;

* 상속의 제한 : 자바는 클래스는 단일 상속이다. (`extends`)
  * **하지만 현실에서는 다른 클래스들을 상속하면서 동시에 쓰레드를 사용할 경우가 많다.**&#x20;
  * 이런 경우 `Runnable` 인터페이스 상속을 통해서 개선할 수 있다.&#x20;
* 유연성 부족 : 인터페이스를 사용하는 방법에 비해서 유연성이 떨어진다.
  * **비지니스 로직, 쓰레드 상태(`started`, `interrupted` 등), 실행 메커니즘이 하나의  쓰레드 객체에 섞인다.**&#x20;
  * `Runnable` 인터페이스 사용을 통해 다음과 같이 작업을 분리해야 유연성이 높아진다.&#x20;
    * `Runnable` : 실행 로직&#x20;
    * `Thread` : 쓰레드 제어&#x20;

#### Runnable 구현&#x20;

장점

* **상속의 자유로움 : 인터페이스는 다중 상속이 가능하다. (implements)**
* **코드의 분리 : 스레드 실행 로직과 스레드 제어 작업을 분리해 가독성(유지보수)을 향상시킬 수 있다.**
* **자원의 공유 : 여러 스레드가 동일한 Runnable 객체를 공유할 수 있다.**

단점

* 코드가 약간 복잡해 질 수 있다.

## 2. Runnable 을 만드는 다양한 방법&#x20;

### 정적 중첩 클래스 사용&#x20;

```java
import static util.MyLogger.log;

public class InnerRunnableMainV1 {

    public static void main(String[] args) {
        log("main() start");

        MyRunnable runnable = new MyRunnable();
        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }

    static class MyRunnable implements Runnable {

        @Override
        public void run() {
            log("run()");
        }
    }
}
```

### 익명 클래스 사용&#x20;

```java
import static util.MyLogger.log;

public class InnerRunnableMainV2 {

    public static void main(String[] args) {
        log("main() start");

        Runnable runnable = new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        };

        Thread thread = new Thread(runnable);
        thread.start();

        log("main() end");
    }
}
```

### 익명 클래스 변수 없이 직접 전달&#x20;

```java
import static util.MyLogger.log;

public class InnerRunnableMainV3 {

    public static void main(String[] args) {
        log("main() start");

        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                log("run()");
            }
        });
        thread.start();

        log("main() end");
    }
}
```

### 람다&#x20;

```java
import static util.MyLogger.log;

public class InnerRunnableMainV4 {

    public static void main(String[] args) {
        log("main() start");

        Thread thread = new Thread(() -> log("run()"));
        thread.start();

        log("main() end");
    }
}
```
