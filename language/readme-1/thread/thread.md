# Thread 기본

## Thread 가 무엇인가?

* 자바 프로그램을 실행하게 되면 적어도 하나의 JVM 이 실행되게 된다.\
  \-> 보통 이렇게 JVM 이 시작되면 자바 프로세스가 실행되게 된다.
* 이러한 프로세스 내에서 쓰레드가 동작한다. \
  **-> 즉, 하나의 프로세스 내에 여러 쓰레드가 동작하고,** \
  **-> 여러 프로세스가 공유하는 하나의 쓰레드가 수행되는 일은 절대 없다...**

> 아무런 쓰레드를 생성하지 않아도 JVM 을 관리하기 위한 여러 쓰레드가 존재한다. \
> 예를 들면, 자바의 쓰레기 객체를 청소하는 GC 관련 쓰레드가 여기에 속한다.&#x20;

### Thread 는 왜 만들어 졌을까?

* 프로세스 하나를 시작하려면 많은 자원이 필요하다.&#x20;
* 만약 여러개의 작업을 동시에 수행하려 할 때 여러 개의 프로세스를 띄워서 실행하면 각각 메모리를 할당해 주어야만 한다. \
  \-> JVM 은 별도의 설정 없이 실행하면  **적어도 32-64MB의 물리 메모리를 점유한다.** \
  \-> 반면에 쓰레드를 하나 추가하면 **1MB 이내의 메모리를 점유한다.** \
  \-> 때문에 쓰레드를 경량 프로세스라 부르기도 한다.&#x20;
* 최근에는 피시 장비도 2코어 이상이기 때문에, 대부분의 작업은 다중 쓰레드로 실행하는 것이 더 빠른 시간의 결과를 제공한다.&#x20;

## Runnable 인터페이스와 Thread 클래스&#x20;

* 쓰레드를 생성하는 것은 크게 2가지 방법이 있다. \
  \-> 클래스가 다른 클래스를 확장해야 할 때는 Runnable 인터페이스를 사용하고, 그렇지 않은 경우는 쓰레드 클래스를 사용하는 것이 편하다.&#x20;

1. Runnable 인터페이스 사용\
   \-> void run() : 쓰레드가 시작되면 수행되는 메서드\
   \-> 해당 메서드 하나만 선언되어 있다. \
   \-> Thread 클래스를 확장시킬 때에는, run() 메서드를 시작점으로 작성해야만 한다.&#x20;
2. Thread 클래스 사용\
   \-> Runnable 인터페이스를 상속한다.&#x20;

```java
public class RannableSample impelement Runnable {
    public void run(){
        System.out.println("runnable!");
    }
}

public class ThreadSample extends Thread {
    public void run(){
        System.out.println("runnable!");
    }
}

public class RunThread(){
    public static void main(String[] args){
        RunThread threads = new RunThread();
        threads.runBasic();
    }
    
    //Runnable, Thread 쓰레드를 호출하는 방식이 서로 다르다. 
    public void runBasic(){
        RannableSample runnable = new RunnableSample();
        new Thread(runnable).start();
        
        ThreadSample thread = new ThreadSample();
        thread.start();
        
        System.out.println("done!");
    }
}

```

## Thread 실행의 결과는 매번 동일하지 않다..&#x20;

* 위의 코드를 실행한 결과가 매번 동일하지 않다..&#x20;
* 그 이유는 다음과 같다.
  * start() 메서드를 실행한다는 것은, 프로세스가 아닌 하나의 쓰레드를 JVM(프로세스)에 추가하여 실행한다는 것이다.&#x20;
  * 이 때, 쓰레드 클래스에 있는 run() 메서드가 끝나든 끝나지 않든, 쓰레드를 시작한 메소드에서 그 다음 줄에 있는 코드를 실행한다.(비동기)

## Thread 클래스의 생성자&#x20;

* Thread() : 새로운 쓰레드 생성
* Thread(Runnable target) : 매개변수의 run() 메서드를 수행하는 쓰레드
* Thread(Runnable target, String name) : 매개변수의 run() 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드\
  \-> 쓰레드의 이름을 정하지 않는다면, **그 쓰레드의 이름은 "Thread-n" 이다.**
* Thread(String name) : 이름을 갖는 쓰레드
* Thread(ThreadGroup group, Runnable target)  : 그룹을 갖고,  매개변수의 run() 메서드를 수행하는 쓰레드
* Thread(ThreadGroup group, Runnable target, String name)  : 그룹을 갖고,  매개변수의 run() 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드
* Thread(ThreadGroup group, Runnable target, String name, long stackSize)  : 그룹을 갖고,  매개변수의 run() 메서드를 수행하는 쓰레드, 추가적으로 이름을 갖는 쓰레드, 해당 쓰레드의 스택 크기는 stackSize 만큼 가능
* Thread(ThreadGroup group, String name) : 그룹을 갖고, 이름을 갖는 쓰레드

## 많이 사용되는 Sleep 메소드에 대해 알아보자&#x20;

* Thread 클래스에 있는 static 메서드는 대부분 JVM 에 있는 쓰레드를 관리하기 위해서 사용된다.
  * static void sleep(long millis)
  * static void sleep(long millis, int nanos)
* **추가적으로 Thread().sleep() 메서드를 사용할 때는 try-catch로 묶어 주어야 한다.** \
  **-> sleep() 메서드는 interruptedException을 발생시키기 때문이다.**\
  **-> 적어도 interruptedException 로 예외를 묶어 주어야 한다.**&#x20;

## Thread 클래스의 주요 메서드

> * void run() : 쓰레드 실행 시 돌아가는 메서드(가장 먼저 구현해야 한다!)
> * long getId() : 쓰레드 고유 ID 리턴
> * String getName() : 쓰레드 이름 리턴&#x20;
> * void setName(String name ) : 쓰레드 이름 설정
> * int getPriority() : 쓰레드 우선순위 확인
> * void setPriority(int newPriority) : 쓰레드 우선순위 설정
> * boolean isDeamon() : 쓰레드가 데몬인지 확인
> * void setDeamon(boolean on) : 쓰레드를 데몬으로 설정할 지 않할지..
> * StackTraceElement\[] getStackTrace() : 쓰레드의 스택 정보를 확인한다.&#x20;
> * Thread.state getState() : 쓰레드의 상태를 확인한다.&#x20;
> * ThreadGroup getThreadGroup() : 쓰레드의 그룹을 확인한다.&#x20;

* 쓰레드의 우선순위는 대부분 기본값으로 사용하는 것을 권장한다.(잘못 설정하면 장애의 원인)

### 데몬쓰레드??

* **데몬쓰레드가 아닌 사용자 쓰레드는 JVM(프로세스 이)해당 쓰레드가 끝날 때 까지 기다린다...**\
  \-> 즉, 어떤 쓰레드를 데몬쓰레드로 지정하면, 쓰레드가 수행되고 있던 없던 상관없이 JVM이 끝날 수 있어야 한다. \
  \-> 단, start() 메서드 이전에 데몬쓰레드가 지정되어야 한다.&#x20;
* **데몬 쓰레드는 해당 쓰레드가 종료되지 않아도, 다른 실행중인 일반 쓰레드가 없다면 멈춰버린다!**
* 사용되는 예
  * 모니터링 하는 쓰레드를 별도로 띄워 모니터링하다가, 주요 쓰레드가 종료되면, 관련된 모니터링 쓰레드가 종료되어야 프로세스가 종료될 수 있다.&#x20;
  * 하지만, 모니터링 쓰레드를 데몬쓰레드로 설정하지 않는다면, 프로세스는 종료할 수 없게 된다.&#x20;
  * **이렇게 부가적인 작업을 수행하는 쓰레드를 선언할 때 데몬쓰레드를 만든다.**
