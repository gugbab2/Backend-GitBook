# Thread 와 관련이 많은, Synchronized

* 여러 쓰레드가 한(동일한) 객체에 선언된 메서드에 접근하여 데이터를 처리하려고 할 때 동시에 연산을 수행하여 값이 꼬이는 경우가 발생한다.
* 단, 메서드에서 인스턴스 변수를 수정하려 할 때만 이런 문제가 생긴다.
* 이러한 문제를 해결하기 위한 방법이 **synchronized** 키워드를 사용하는 것이다!\
  \-> 아래 방법을 통해서 하나의 쓰레드가 해당 메서드를 점유한다.

```java
public synchronized void plus(int value){
    amount += value;
}    

public synchronized void minus(int value){
    amount += value;
} 
```

## synchronized 를 사용해보자!

```java
public class ModifyAmountThread extends Thread{
    private CommonCalculate calc;
    private boolean addFlag;
    public ModifyAmountThread(CommonCalculate calc, boolean addFlag){
        this.calc = calc;
        this.addFlag = addFlag;
    }
    
    public void run(){
        for(int loop = 0; loop<10000; loop++){
            if(addFlag){
                calc.plus(1);
            } else {
                calc.minus(1);
            }
        }
    }
}

public class RunSync {
    public static void main(String[] args){
        RunSync runSync = new RunSync();
        runSync.runCommonCalculate();
    }
    
    public void runCommonCalculate(){
        CommonCalculate calc = new CommonCalculate();
        ModifyAmountThread thread1 = new ModifyAmountThread(calc, true);
        ModifyAmountThread thread2 = new ModifyAmountThread(calc, true);
        
        thread1.start();
        thread2.start();
 
        try{
            thread1.join();
            thread2.join();
        }catch(InterrupedException e){
            e.printStackTrace();
        }
    }
}    
```

* join() 메서드는 쓰레드가 종료할 때까지 기다리라는 의미이다.
* synchronized 키워드를 사용하지 않고 위 코드를 실행했을 때, 우리는 20000 을 기대하지만 20000 이 나오는 경우는 없다고 봐야한다..\
  \-> 앞서 말했던 쓰레드는 비동기적으로 실행되기 때문이다.
* 우리가 예상하는 대로 결과를 얻기 위해서 synchronized 키워드를 사용해야 한다.

## synchronized 블럭을 사용해보자

* 위와 같은 방법으로 동기화를 해주어 문제를 해결할 수도 있지만 고민해야 하는 부분이 생긴다..
* 만약 plus 메서드에서 인스턴스 변수를 연산하는 줄이 1줄이고 아닌 줄이 100줄이라고 가정해보자,
* 위와 같이 메서드 전체를 동기화 해버리면 인스턴스 변수와 상관없는 100줄의 대기 시간이 발생하게 된다.
* 이런 경우 아래와 같이 동기화가 필요한 부분만 블럭으로 처리를 해야한다.

```java
public synchronized void plus(int value){
    synchronized(this){
        amount += value;
    }
} 
```

* 위와 같이 this 로 지정하는 것이 아닌, 일반적으로는 별도의 객체를 선언하여 사용한다.
* 아래의 lock 객체는 문지기 객체로 synchronized 블럭 안에서 하나의 쓰레드만 작업할 수 있도록 할 수 있다.

```java
Object lock = new Object();

public synchronized void plus(int value){
    
    synchronized(lock){
        amount += value;
    }
} 
```

* 만약 하나의 문지기 객체를 가지고 2개의 메서드를 synchronized 설정하게 되면, 한개의 변수를 처리할 때, 다른 변수를 처리할 수 없다..
* 때문에 각각의 메서드마다 각각의 문지기 객체를 만들자.

```java
Object lock1 = new Object();
Object lock2 = new Object();

public synchronized void plus1(int value){
    
    synchronized(lock1){
        amount1 += value;
    }
} 

public synchronized void plus2(int value){
    
    synchronized(lock2){
        amount2 += value;
    }
} 
```
