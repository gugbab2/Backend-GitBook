# Thread 를 통제하는 메서드

## Thread 를 통제하는 메서드

* `Thread.State getState()` : 쓰레드의 상태를 확인한다.
* `void join()` : 실행중인 쓰레드가 다른 쓰레드의 작업이 끝날때까지 대기한다.
* `void join(long millis)` : 매개변수에 지정된 시간만큼 대기한다.
* `void join(long millis, int nanos)` : 매개변수에 지정된 시간만큼 대기한다.
* `void interrupt()` : 수행중인 쓰레드에 중지 요청을 한다.
  * **차단된 상태**에서 호출된 경우:\
    스레드가 `sleep()`, `wait()`, 또는 `join()` 등의 차단 상태에 있을 때 `interrupt()` 메서드를 호출하면 스레드가 깨어나면서 **즉시** `InterruptedException`이 발생합니다. 이 예외는 해당 스레드가 인터럽트되었음을 알리는 방식입니다.
  * **차단되지 않은 상태**에서 호출된 경우:\
    스레드가 실행 중이고 차단 상태가 아니라면 `InterruptedException`은 발생하지 않으며, 대신 해당 스레드의 **인터럽트 플래그**가 설정됩니다. 이 상태에서 나중에 스레드가 차단 메서드를 호출하면 `InterruptedException`이 발생할 수 있습니다.

## Thread 상태 (Thread.State.NEW 형식으로 사용 가능)

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* `NEW`&#x20;
  * 쓰레드 객체는 생성되었지만, 아직 시작되지는 않는 상태
  * `new Thread()` 로 객체가 생성된 후, 아직 `start()` 메서드가 호출되지 않은 상태&#x20;
  * 이 상태에서는 쓰레드가 실제로 실행되지 않았기 때문에, CPU 자원을 사용하지 않는다.&#x20;
* `RUNNABLE`&#x20;
  * 쓰레드가 실행중이거나, 실행 준비가 된 상태이다.&#x20;
  * `start()` 메서드가 호출된 후, 쓰레드는 이 상태로 전환된다.&#x20;
  * 이 상태의 쓰레드는 CPU 에서 실행중이거나 또는 운영체제 스케줄러에 의해서 실행할 준비가 된 상태이다. \
    \-> 즉, CPU 자원을 얻었다가 놓치면 다시 `RUNNABLE` 상태로 돌아간다.&#x20;
* `BLOCKED`
  * 쓰레드가 실행 중지 상태이며, 모니터락이 풀리기를 기다리는 상태
  * 만약 특정 객체가 `synchronized` 로 인해 락이 걸려 있다면 `BLOCKED` 상태가 된다.&#x20;
* `WAITING`&#x20;
  * 쓰레드가 다른 쓰레드의 작업이 완료되기를 기다리는 상태이다.&#x20;
  * 쓰레드는 `Object.wait()`, `Thread.join()`, `LockSupport.park()` 같은 메서드에 의해 `WAITING` 상태에 놓인다.&#x20;
  * 이 상태에 있는 쓰레드는 다른 쓰레드가 특정 신호를 보내거나(`notify`, `notifyAll`) 대기 중인 작업이 완료되었을 때만 다시 실행 가능한 상태로 돌아간다.&#x20;
* `TIMED_WATTING`
  * 특정 시간만큼 쓰레드가 대기중인 상태, 시간이 지나면 자동으로 실행 가능 상태로 돌아온다.&#x20;
  * `Thread.sleep()`, `Object.wait(long timeout)`, `Thread.join(long millis)` 같은 메서드에 의해 이 상태로 진입한다.&#x20;
* `TERMINATED`&#x20;
  * 쓰레드가 종료된 상태
  * 정상적인 종료, 예외로 인해 작업이 중단된 케이스 모두 해당된다.&#x20;

## Object 클래스에 선언되어 있는 쓰레드와 관련된 메소드들

* `void wait()` : 다른 쓰레드가 `Object` 객체에 대한, `notify()` 메서드나 `notifyAll()` 메서드를 호출할 때까지 현재 쓰레드가 대기하고 있도록 한다.
* `void wait(long timeout)` : `wait()` 메서드와 동일하며, 매개변수 동안 대기하고 있는다.
* `void wait(long timeout, int nanos)` :
* `void notify()` : `Object` 객체의 모니터에 대기하고 있는 단일 쓰레드를 깨운다.
* `void notifyAll()` : `Object` 객체의 모니터에 대기하고 있는 모든 쓰레드를 깨운다.

```java
public class StateThread extends Thread{
    private Object monitor;
    public StateThread(Object monitor){
        this.monitor = monitor;
    }
    
    public void run(){
        try{
            for(int loop=0; loop<10000; loop++){
                String a="A";
            }
            synchronized(monitor){
                monitor.wait();
            }
            System.out.println("test");
            Thread.sleep(1000);
        }catch(InterruptException){
        
        }
    }
}

public class ThreadSample {
    public static void main(String[] args){
        ThreadSample sample = new ThreadSample();
        sample.checkThreadState3();
    }
    
    public void checkThreadState3(){
        Object monitor = new Object();
        StateThread thread = new StateThread(monitor);
        StateThread thread2 = new StateThread(monitor);
        
        try{
            thread.start();
            thread2.start();
            
            Thread.sleep(100);
            synchronized(monitor){
                // 먼저 대기하고 있는 Thread 부터 풀어주기 때문에 
                // thread2 는 풀리지 않았다. 
                // monitor.notifyAll(); 을 사용하자.
                monitor.notify();
            }
            
            Thread.sleep(100);
            
            thread.join();
            thread2.join();
            
        }catch(InterruptException e){
        
        }
    }
}
```

##
