# Thread 기본

## Thread 가 무엇인가?

* 자바 프로그램을 실행하게 되면 적어도 하나의 JVM 이 실행되게 된다.\
  -> 보통 이렇게 JVM 이 시작되면 자바 프로세스가 실행되게 된다.
* 이러한 프로세스 내에서 쓰레드가 동작한다.\
  &#xNAN;**-> 즉, 하나의 프로세스 내에 여러 쓰레드가 동작하고,**\
  &#xNAN;**-> 여러 프로세스가 공유하는 하나의 쓰레드가 수행되는 일은 절대 없다...**

> 아무런 쓰레드를 생성하지 않아도 JVM 을 관리하기 위한 여러 쓰레드가 존재한다.\
> 예를 들면, 자바의 쓰레기 객체를 청소하는 GC 관련 쓰레드가 여기에 속한다.

### Thread 는 왜 만들어 졌을까?

* 프로세스 하나를 시작하려면 많은 자원이 필요하다.
* 만약 여러개의 작업을 동시에 수행하려 할 때 여러 개의 프로세스를 띄워서 실행하면 각각 메모리를 할당해 주어야만 한다.
  * **JVM 은 별도의 설정 없이 실행하면 적어도 32-64MB의 물리 메모리를 점유한다.**
  * **반면에 쓰레드를 하나 추가하면 1MB 이내의 메모리를 점유한다.**\
    **(때문에 쓰레드를 경량 프로세스라 부르기도 한다)**
* 최근에는 PC 장비도 2코어 이상이기 때문에, 대부분의 작업은 다중 쓰레드로 실행하는 것이 더 빠른 시간의 결과를 제공한다.

## `Runnable` 인터페이스와 `Thread` 클래스

#### 쓰레드를 생성하는 것은 크게 2가지 방법이 있다.

1. `Runnable` 인터페이스 사용 \
   -> `run()` 메서드 하나만 선언되어 있다.&#x20;
2. `Thread` 클래스 사용 \
   -> `run()` 메서드를 포함한 수 많은 메서드와 생성자를 제공한다.&#x20;

* `Runnable` 인터페이스와, `Thread` 클래스를 사용하는 간단한 예제를 살펴보자&#x20;
* 아래 예제에서 특징적인 부분은 다음과 같다.&#x20;
  * **쓰레드가 수행되는 우리가 구현한 메서드는 `run()` 메서드이다.**&#x20;
  * **하지만, 쓰레드를 시작하는 메서드는 `start()` 메서드이다.** \
    **-> 우리가 `start()` 메서드를 만들지 않아도 알아서 자바에서 `run()` 메서드를 실행하도록 되어있다.**&#x20;

```java
// Runnable 사용 
public class RunnableSample implemenets Runnable {
    public void run() {
        System.out.println("This is RunnableSample's run() method");
    }
}

// Thread 사용 
public class ThreadSample extends Thread {
    public void run() {
        System.out.println("This is ThreadSample's run() method");
    }
}

// 쓰레드 사용 예제 
public class RunThreads {
    public static void main(String[] args){
        RunThreads threads = new RunThreads(); 
        threads.runBasic();
    }
    
    public void runBasic(){
        RunnableSample runnable = new RunnableSample(); 
        new Thread(runnable).start(); 
        
        ThreadSample thread = new ThreadSample(); 
        thread.start();
    }
}
```

### `Thread` 실행의 결과는 매번 동일하지 않다..

* **위의 코드를 실행한 결과가 매번 동일하지 않다..**
* 그 이유는 다음과 같다.
  * `start()` 메서드를 실행한다는 것은, 프로세스가 아닌 하나의 쓰레드를 JVM(프로세스)에 추가하여 실행한다는 것이다.
  * 이 때, 쓰레드 클래스에 있는 `run()` 메서드가 끝나든 끝나지 않든, 쓰레드를 시작한 메소드에서 그 다음 줄에 있는 코드를 실행한다. (비동기)

### 왜 `Runnable` 와 `Thread` 를 왜 구분해서 만들었을까?

#### 다음의 상황을 가정해보자&#x20;

* 만약 어떤 클래스가 어떤 다른 클래스를 `extends` 를 사용해 확장해야 하는 상황인데, 쓰레드로 구현해야 한다.&#x20;
* 게다가, 그 부모 클래스는 `Thread` 를 확장하지 않았다면 어떻게 해야할까?

#### 어떻게 해결할까?

* **자바에서 다중 상속은 불가능하기 때문에, 해당 클래스를 쓰레드로 만들 수 없다. 하지만 인터페이스는 여러 개의 인터페이스를 구현해도 전혀 문제가 발생하지 않는다.**&#x20;
* **따라서 위와 같은 상황에서는 `Runnable` 인터페이스를 상속해서 구현하면 된다.**&#x20;

## `Thread` 클래스의 생성자

* `Thread()` : 새로운 쓰레드 생성
* `Thread(Runnable target)` : 매개변수의 `run()` 메서드를 수행하는 쓰레드
* `Thread(Runnable target, String name)` : 매개변수의 `run()` 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드
  * 쓰레드의 이름을 정하지 않는다면, 그 쓰레드의 이름은 "Thread-n" 이다.
  * 쓰레드의 이름이 겹친다고 해도 예외나 에러가 발생하지 않는다.&#x20;
* `Thread(String name)` : 이름을 갖는 쓰레드
* `Thread(ThreadGroup group, Runnable target)` : 그룹을 갖고, 매개변수의 run() 메서드를 수행하는 쓰레드
* `Thread(ThreadGroup group, Runnable target, String name)` : 그룹을 갖고, 매개변수의 `run()` 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드
* `Thread(ThreadGroup group, Runnable target, String name, long stackSize)` : 그룹을 갖고, 매개변수의 `run()` 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드, 해당 쓰레드의 스택 크기는 stackSize 만큼 가능
* `Thread(ThreadGroup group, String name)` : 그룹을 갖고, 이름을 갖는 쓰레

### 생성자를 사용한 예제를 살펴보자 &#x20;

```java
// 매개변수를 통해 쓰레드 이름을 설정해보자 
public class NameThread extends Thread{ 
    public NameThread(String name) {
        super(name); 
    }
    
    public void run() {
    
    }
}

// 쓰레드에게 매개변수를 넘겨보자 
public class NameCalcThread extends Thread{
    private int calcNumber;
    public NameCalcThread(String name, int calcNumber){
        super(name);
        this.calcNumber = calcNumber;
    }
    public void run() {
        calcNumber++;
    }
}
```



## Thread 클래스의 주요 메서드

* `void run()` : 쓰레드 실행 시 돌아가는 메서드(가장 먼저 구현해야 한다!)
* `long getId()` : 쓰레드 고유 ID 리턴 (JVM 에서 자동으로 생성)
* `String getName()` : 쓰레드 이름 리턴
* `void setName(String name)` : 쓰레드 이름 설정
* `int getPriority()` : 쓰레드 우선순위 확인
* `void setPriority(int newPriority)` : 쓰레드 우선순위 설정
* `boolean isDeamon()` : 쓰레드가 데몬인지 확인
* `void setDeamon(boolean on)` : 쓰레드를 데몬으로 설정할 지 않할지..
* `StackTraceElement[] getStackTrace()` : 쓰레드의 스택 정보를 확인한다.
* `Thread.state getState()` : 쓰레드의 상태를 확인한다.
* `ThreadGroup getThreadGroup()` : 쓰레드의 그룹을 확인한다.

### 많이 사용되는 Sleep 메소드에 대해 알아보자

* `Thread` 클래스에 있는 static 메서드는 대부분 JVM 에 있는 쓰레드를 관리하기 위해서 사용된다.
* `sleep` 메서드는 쓰레드가 매개변수 시간만큼 대기한다.&#x20;
  * `static void sleep(long millis)`
  * `static void sleep(long millis, int nanos)`
* 추가적으로 `Thread().sleep()` 메서드를 사용할 때는 `try-catch`로 묶어 주어야 한다.
  * `sleep()` 메서드는 `interruptedException`을 발생시키기 때문이다.

## 데몬 쓰레드

### 데몬 쓰레드란?

* Java 의 쓰레드는 두가지 유형으로 나뉜다.&#x20;
  * 사용자 쓰레드&#x20;
    * 사용자 쓰레드는 우선순위가 높은 쓰레드이다.&#x20;
    * JVM 은 사용자 쓰레드가 작업을 완료할 때까지 기다린 후 종료한다.&#x20;
  * 데몬 쓰레드
    * **사용자 쓰레드에 서비스를 제공하는 것만을 담당하는 낮은 우선순위의 쓰레드이다.**&#x20;
    * **JVM 은 데몬쓰레드가 살아 있더라도 모든 사용자 쓰레드가 작업을 완료했다면 프로세스를 종료한다.**&#x20;
    * **그래서 일반적으로 데몬 쓰레드에 존재하는 무한 루프는 문제를 일으키지 않는다.** \
      **-> 데몬 쓰레드는 I/O 작업에 권장되지 않는다.** \
      **(사용자 쓰레드가 종료되는 경우 데몬 쓰레드가 사용하던 리소스를 닫지 않고 종료될 수 있다)**&#x20;

### 데몬 쓰레드 사용&#x20;

* 데몬 쓰레드는 가비지 수집, 메모리 해제, 캐시에서 원치 않는 항목 제거 등 백그라운드 지원 작업 시 유용하게 사용 가능하다.&#x20;
* 대부분의 JVM 쓰레드는 데몬 쓰레드이다.&#x20;
* 데몬 쓰레드는 쓰레드 시작 전 데몬 쓰레드로 설정을 해주어야 한다. \
  -> 만약, 쓰레드 시작 후 데몬 쓰레드로 설정한다면 `IllegalThreadStateException` 이 발생한다.&#x20;

```java
// 정상 케이스 
Thread daemonThread = new NewThread();
daemonThread.setDaemon(true);
daemonThread.start();

// 오류 케이스 
Thread daemonThread = new NewThread();
daemonThread.start();
daemonThread.setDaemon(true);    // IllegalThreadStateException 
```
