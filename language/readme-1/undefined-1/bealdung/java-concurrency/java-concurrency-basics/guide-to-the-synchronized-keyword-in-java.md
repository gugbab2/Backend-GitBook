# Guide to the Synchronized Keyword in Java

> 참고 링크&#x20;
>
> [https://www.baeldung.com/java-synchronized](https://www.baeldung.com/java-synchronized)



## 1. 개요&#x20;

* 멀티쓰레드 환경에서 두 개 이상의 쓰레드가 동시에 변경 가능한 공유 데이터에 접근하게 되면 **경쟁 조건**이 발생한다.
* **Java 는 쓰레드 엑세스를 공유 데이터에 동기화하여 경쟁 조건을 피하는 매커니즘을 제공한다.**&#x20;

## 2. 동기화가 필요한 이유&#x20;

* 여러 쓰레드가 calculate() 메서드를 실행하는 일반적인 경쟁 조건이 발생하는 상황을 생각해보자.

```java
public class SynchronizedMethods {

    private int sum = 0;

    public void calculate() {
        setSum(getSum() + 1);
    }

    // standard setters and getters
}
```

* 아래 테스트에서는 3개의 쓰레드 풀을 가지고 있는 `ExecutorService` 를 통해서 `calculate()` 를 1000번 실행한다.
* 우리는 아래 테스트의 실행 결과가 1000이 될 것 이라고 생각하지만, 실상은 쓰레드간 경쟁을 통해서 일관되지 않은 결과를 확인하게 된다..&#x20;
* 경쟁 조건을 피하는 가장 간단한 방법은 `synchronized` 키워드를 사용하는 것이다.&#x20;

```java
@Test
public void givenMultiThread_whenNonSyncMethod() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    SynchronizedMethods summation = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(summation::calculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, summation.getSum());
}

// [ERROR MESSAGE]
// java.lang.AssertionError: expected:<1000> but was:<965>
// at org.junit.Assert.fail(Assert.java:88)
// at org.junit.Assert.failNotEquals(Assert.java:834)
// ...
```

## 3. 동기화 방법&#x20;

### 3-1. 동기화 된 인스턴스 메서드&#x20;

* 메서드 선언에 synchronized 키워드를 추가하면 메서드를 동기화할 수 있다.&#x20;

```java
public synchronized void synchronisedCalculate() {
    setSum(getSum() + 1);
}
```

* 메서드를 동기화하면 실제 테스트 케이스 출력이 1000이 되는 것을 확인할 수 있다.&#x20;

```java
@Test
public void givenMultiThread_whenMethodSync() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    SynchronizedMethods method = new SynchronizedMethods();

    IntStream.range(0, 1000)
      .forEach(count -> service.submit(method::synchronisedCalculate));
    service.awaitTermination(1000, TimeUnit.MILLISECONDS);

    assertEquals(1000, method.getSum());
}
```

* 인스턴스 메서드는 메서드를 소유한 클래스 인스턴스를 통해서 동기화된다.&#x20;
* **즉, 클래스 인스턴스 당 하나의 쓰레드만 해당 메서드를 실행할 수 있다.**&#x20;

### 3-2. 동기화 된 정적 메서드&#x20;

* 정적 메서드는 인스턴스 메서드와 마찬가지로 동기화 된다.&#x20;

```java
 public static synchronized void syncStaticCalculate() {
     staticSum = staticSum + 1;
 }
```

* 이러한 메서드는 클래스와 연관된 정적 인스턴스에서 동기화된다.&#x20;
* **JVM 내에서 클래스당 정적 객체가 하나씩 존재하기 때문에, 동기화 된 정적 메서드를 실행할 수 있는 쓰레드는 한개 뿐이다.**

```java
@Test
public void givenMultiThread_whenStaticSyncMethod() {
    ExecutorService service = Executors.newCachedThreadPool();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(SynchronizedMethods::syncStaticCalculate));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, SynchronizedMethods.staticSum);
}
```

### 3-3. 메서드 내 동기화 된 블럭&#x20;

* 때로는 메서드 전체를 동기화하는 것이 아닌 일부만 동기화하고 싶을 수 있다.&#x20;
* 이때 동기화 블럭을 사용하면 된다.&#x20;

```java
public void performSynchronisedTask() {
    synchronized (this) {
        setCount(getCount()+1);
    }
}
```

* **동기화 블럭에 매개변수 this 를 전달한 것에 주목해라.**&#x20;
* **이것은 모니터 객체로, 블록 내부의 코드는 모니터 객체에서 동기화된다.**&#x20;
* **쉽게 말해서, 모니터 객체당 하나의 쓰레드만 해당 코드 블럭을 실행할 수 있다.**&#x20;

```java
@Test
public void givenMultiThread_whenBlockSync() {
    ExecutorService service = Executors.newFixedThreadPool(3);
    SynchronizedBlocks synchronizedBlocks = new SynchronizedBlocks();

    IntStream.range(0, 1000)
      .forEach(count -> 
        service.submit(synchronizedBlocks::performSynchronisedTask));
    service.awaitTermination(100, TimeUnit.MILLISECONDS);

    assertEquals(1000, synchronizedBlocks.getCount());
}
```

* **메서드가 정적이면 객체 참조 대신, 클래스 이름을 전달하고 클래스는 블럭 동기화를 위한 모니터가 된다.**&#x20;

```java
public static void performStaticSyncTask(){
    synchronized (SynchronisedBlocks.class) {
        setStaticCount(getStaticCount() + 1);
    }
}
```

## 4. `synchronized` 키워드의 장단점

### 4-1. 장점&#x20;

* 메모리 가시성 보장
  * `synchronized` 블록에 들어가면, 해당 블록을 수행하기 전에 다른 쓰레드에서 변경된 값들이 메인 메모리에서 로드되며, 블록이 끝나면 그 안에서 변경된 값이 메인 메모리에 저장된다.&#x20;
  * 따라서 쓰레드 간 데이터의 일관성을 보장한다.&#x20;
* 원자성 보장&#x20;
  * `synchonized` 블록 내의 코드가 원자적으로 실행된다.&#x20;
  * 즉, 한 쓰레드가 `synchronized` 블록에 들어가면 다른 쓰레드는 해당 블록에 접근할 수 없으므로 데이터 경쟁 상태를 방지한다.
* 락을 통한 동기화
  * Java 에서 제공하는 기본적인 동기화 매커니즘으로, 객체의 락을 사용하여 쓰레드 간 안전한 접근을 제공한다.&#x20;

### 4-2. 단점&#x20;

* 성능 저하
  * `synchonized` 는 락을 획득하고 해제하는 오버헤드가 있으며, 이로 인해 성능이 저하될 수 있다.&#x20;
  * 특히, 락 경쟁이 심한 경우 더욱 두드러진다. \
    \-> 락 경쟁 : 쓰레드가 락을 얻기 위해 대기하게 되어 성능 저하를 초래하는 상황을 말한다.&#x20;
* 교착 상태(Deadlock)&#x20;
  * 여러 쓰레드가 서로의 락을 기다리는 상황에서 교착 상태가 발생할 수 있다.&#x20;
  * 이는 프로그램의 실행을 멈추는 결과를 초래할 수 있다.&#x20;
* 직관적이지 않다.&#x20;
  * `synchonized` 를 사용할 때 코드의 흐름이 복잡해 질 수 있으며, 올바른 사용법을 이해하는데 시간이 걸리 수 있다.&#x20;
