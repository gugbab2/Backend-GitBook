# Thread 를 통제하는 메서드

## Thread 를 통제하는 메서드&#x20;

* Thread.State getState() : 쓰레드의 상태를 확인한다.
* void join() : 수행중인 쓰레드가 중지할 때까지 대기한다.
* void join(long millis) : 매개변수에 지정된 시간만큼 대기한다.&#x20;
* void join(long millis, int nanos) : 매개변수에 지정된 시간만큼 대기한다.
* void interrupt() : 수행중인 쓰레드에 중지 요청을 한다. \
  \-> 그냥 중지되는 것이 아닌, InterruptException 을 발생하면서 중지한다.&#x20;

## Thread 상태

* NEW : 쓰레드 객체는 생성되었지만, 아직 시작되지는 않는 상태
* RUNNABLE : 쓰레드가 실행중인 상태
* BLOCKED : 쓰레드가 실행 중지 상태이며, 모니터락이 풀리기를 기다리는 상태
* WAITING : 쓰레드가 대기중인 상태
* TIMED\_WATTING : 특정 시간만큼 쓰레드가 대기중인 상태
* TERMINATED : 쓰레드가 종료된 상태

## Object 클래스에 선언되어 있는 쓰레드와 관련된 메소드들&#x20;

* void wait() : 다른 쓰레드가 Object 객체에 대한, notify() 메서드나 notifyAll() 메서드를 호출할 때까지 현재 쓰레드가 대기하고 있도록 한다.&#x20;
* void wait(long timeout) : wait() 메서드와 동일하며, 매개변수 동안 대기하고 있는다.&#x20;
* void wait(long timeout, int nanos) :&#x20;
* void notify() : Object 객체의 모니터에 대기하고 있는 단일 쓰레드를 깨운다.
* void notifyAll() : Object 객체의 모니터에 대기하고 있는 모든 쓰레드를 깨운다.&#x20;

```java
public class StateThread extends Thread{
    private Object monitor;
    public StateThread(){
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
            
            Thread.sleep();
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

## ThreadGroup

* ThreadGroup 은 쓰레드의 관리를 용이하게 만들기 위한 클래스이다.&#x20;
* ThreadGroup 클래스의 메서드는 다음과 같다.
  * int activeCount() : 실행중인 쓰레드이 개수를 리턴한다.
  * int activeGroupCount() : 실행중인 쓰레드 그룹의 개수를 리턴한다.&#x20;
  * int enumerate(Thread\[] list) : 현재 쓰레드 그룹에 있는 모든 쓰레드의 매개 변수로 넘어온 쓰레드 배열에 담는다. \
    \-> **enumerate() 메서드는 매개변수에 Thread 그룹을 담거나,Thread 를 담는데, 배열을 만들기 이전에 activeCount() 메서드 호출을 통해서 크기를 미리 지정해 두면 보다 효율적인 객체 생성을 할 수 있다.**&#x20;
  * int enumerate(Thread\[] list, boolean recurse) : 현재 쓰레드 그룹에 있는 모든 쓰레드의 매개 변수로 넘어온 쓰레드 배열에 담는다. recurse 가 true 이면, 하위에 있는 쓰레드 그룹에 있는 쓰레드 목록도 포함된다.&#x20;
  * int enumerate(ThreadGroup\[] list) : 현재 그룹에 있는 모든 쓰레드 그룹을 매개변수로 넘어온 쓰레드 그룹 배열에 담는다.&#x20;
  * int enumerate(ThreadGroup\[] list, boolean recurse) : 현재 그룹에 있는 모든 쓰레드 그룹을 매개 변수로 넘어온 쓰레드 그룹 배열에 담는다. recurse 가 true 이면, 하위에 있는 쓰레드 그룹에 있는 쓰레드 목록도 포함된다.&#x20;
  * String getName() : 쓰레드 그룹의 이름을 리턴한다.&#x20;
  * ThreadGroup getParent() : 부모 쓰레드 그룹을 리턴한다.&#x20;
  * void list() : 쓰레드 그룹의 상세정보를 출력한다.
  * void setDaemon(boolean daemon) : 지금 쓰레드 그룹에 속한 쓰레드들을 데몬으로 지정한다.
