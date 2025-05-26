# 동시성 컬렉션

## 동시성 컬렉션이 필요한 이유1 - 시작&#x20;

`java.util` 패키지에 소속되어 있는 컬렉션 프레임워크는 원자적인 연산을 제공할까?&#x20;

* 예를 들어, 하나의 `ArrayList` 인스턴스에 여러 스레드가 동시에 접근해도 괜찮을까?&#x20;
* 참고로 여러 스레드가 동시에 접근해도 괜찮을 경우를 스레드 세이프(thread safe) 하다고 한다.&#x20;

컬렉션에 데이터를 추가하는 `add()` 메서드를 생각해보면, 단순히 컬렉션에 데이터를 하나 추가하는 것 뿐이다. 따라서 이것은 원자적인 연산처럼 느껴지지만 결과적으로 컬렉션 프레임워크가 제공하는 대부분의 연산은 원자적인 연산이 아니다.&#x20;

### 컬렉션 직접 만들기&#x20;

단일 스레드로 실행하기 때문에 정상적으로 동작한다.&#x20;

```java
package thread.collection.simple.list;

public interface SimpleList {

    int size();
    void add(Object e);
    Object get(int index);
}
```

```java
package thread.collection.simple.list;

import java.util.Arrays;

import static util.ThreadUtils.sleep;

public class BasicList implements SimpleList {

    private static final int DEFALUT_CAPACITY = 5;

    private Object[] elementData;
    private int size = 0;

    public BasicList() {
        this.elementData = new Object[DEFALUT_CAPACITY];
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public void add(Object e) {
        elementData[size] = e;
        sleep(100); // 멀티스레드 문제를 쉽게 확인하는 코드
        size++;
    }

    @Override
    public Object get(int index) {
        return elementData[index];
    }

    @Override
    public String toString() {
        return Arrays.toString(Arrays.copyOf(elementData, size)) + " size = " + size
                + ", capacity = " + elementData.length;
    }
}
```

```java
package thread.collection.simple.list;

public class SimpleListMainV1 {

    public static void main(String[] args) {
        SimpleList list = new BasicList();
        list.add("A");
        list.add("B");
        System.out.println("list = " + list);
    }
}
```

## 동시성 컬렉션이 필요한 이유2 - 동시성 문제&#x20;

### 멀티스레드 문제 확인

#### add() - 원자적이지 않은 연산

이 메서드는 단순히 데이터를 추가하는 것으로 끝나지 않는다. 내부에 있는 배열에 데이터를 추가해야 하고, `size` 도 함께 하나 증가시켜야 한다. 심지어 `size++` 연산 자체도 원자적이지 않다. `size++` 연산은 `size = size + 1` 연산과 같다.

이렇게 원자적이지 않은 연산을 멀티스레드 상황에 안전하게 사용하려면 `synchronized`,  `Lock` 등을 사용해서 동 기화를 해야한다.

```java
@Override
public void add(Object e) {
    elementData[size] = e;
    sleep(100); // 멀티스레드 문제를 쉽게 확인하는 코드
    size++;
}
```

### 문제점

#### 컬렉션 프레임워크 대부분은 스레드 세이프하지 않다.&#x20;

우리가 일반적으로 자주 사용하는 `ArrayList` , `LinkedList` , `HashSet` , `HashMap` 등 수 많은 자료 구조들은 \
단순한 연산을 제공하는 것 처럼 보인다.

* 예를 들어, 데이터를 추가하는 `add()` 와 같은 연산은 마치 원자적인 연산처럼 느껴진다.
* 하지만 그 내부에서는 수 많은 연산들이 함께 사용된다. 배열에 데이터를 추가하고, 사이즈를 변경하고, 배열을 새로 \
  만들어서 배열의 크기도 늘리고, 노드를 만들어서 링크에 연결하는 등 수 많은 복잡한 연산이 함께 사용된다.

**따라서 일반적인 컬렉션들은 절대로!! 스레드 세이프 하지 않다!!**

단일 스레드가 컬렉션에 접근하는 경우라면 아무런 문제가 없지만, 멀티스레드 상황에서 여러 스레드가 동시에 하나의 컬렉션에 접근하는 경우라면 `java.util` 패키지가 제공하는 일반적인 컬렉션들은 사용하면 안된다!\
(물론 일부 예외도 있다. 뒤에서 설명한다.)

## 동시성 컬렉션이 필요한 이유3 - 동기화&#x20;

컬렉션 프레임워크의 동시성 문제를 락을 통해서 해결할 수 있다.&#x20;

아래 예제에서는 간편하게 `synchronized` 키워드를 사용해 동시성 문제를 해결했다.&#x20;

```java
package thread.collection.simple.list;

import java.util.Arrays;

import static util.ThreadUtils.sleep;

public class SyncList implements SimpleList {

    private static final int DEFALUT_CAPACITY = 5;

    private Object[] elementData;
    private int size = 0;

    public SyncList() {
        this.elementData = new Object[DEFALUT_CAPACITY];
    }

    @Override
    public int size() {
        return size;
    }

    @Override
    public synchronized void add(Object e) {
        elementData[size] = e;
        sleep(100); // 멀티스레드 문제를 쉽게 확인하는 코드
        size++;
    }

    @Override
    public synchronized Object get(int index) {
        return elementData[index];
    }

    @Override
    public synchronized String toString() {
        return Arrays.toString(Arrays.copyOf(elementData, size)) + " size = " + size
                + ", capacity = " + elementData.length;
    }
}
```

```java
package thread.collection.simple.list;

import static util.MyLogger.log;

public class SimpleListMainV2 {

    public static void main(String[] args) throws InterruptedException {
//        test(new BasicList());
        test(new SyncList());
    }

    private static void test(SimpleList list) throws InterruptedException {
        log(list.getClass().getSimpleName());

        Runnable addA = () -> {
            list.add("A");
            log("Thread-1 : list.add(A)");
        };

        Runnable addB = () -> {
            list.add("B");
            log("Thread-2 : list.add(B)");
        };

        Thread thread1 = new Thread(addA, "Thread-1");
        Thread thread2 = new Thread(addB, "Thread-2");
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        log(list);
    }
}
```

### 문제점&#x20;

`BasicList` 코드가 있는데, 이 코드를 그대로 복사해서 `synchronized` 기능만 추가한 `SyncList` 를 만들었다.

하지만 이렇게 되면 모든 컬렉션을 다 복사해서 동기화용으로 새로 구현해야 한다. 이것은 매우 비효율적이다.

## 동시성 컬렉션이 필요한 이유4 - 프록시 도입&#x20;

### 프록시 (Proxy)&#x20;

우리말로 대리자, 대신 처리해주는 자라는 뜻이다.

프록시를 쉽게 풀어서 설명하자면 친구에게 대신 음식을 주문해달라고 부탁하는 상황을 생각해 볼 수 있다.

* 예를 들어, 당신이 피자를 먹고 싶은데, 직접 전화하는 게 부담스러워서 친구에게 대신 전화해서 피자를 주문해달라고 \
  부탁한다고 해보자. 친구가 피자 가게에 전화를 걸어 주문하고, 피자가 도착하면 당신에게 가져다주는 것이다. \
  여기서 친구가 프록시 역할을 하는 것이다.
  * 나(클라이언트) 피자 가게(서버)
  * 나(클라이언트) 친구(프록시) 피자 가게(서버)

객체 세상에도 이런 프록시를 만들 수 있다. 여기서는 프록시가 대신 동기화( `synchronized`) 기능을 처리해주는 \
것이다.

```java
package thread.collection.simple.list;

public class SyncProxyList implements SimpleList {

    private SimpleList target;

    public SyncProxyList(SimpleList target) {
        this.target = target;
    }

    @Override
    public synchronized int size() {
        return target.size();
    }

    @Override
    public synchronized void add(Object e) {
        target.add(e);
    }

    @Override
    public synchronized Object get(int index) {
        return target.get(index);
    }

    @Override
    public String toString() {
        return target.toString() + " by " + this.getClass().getSimpleName();
    }
}
```

```java
package thread.collection.simple.list;

import static util.MyLogger.log;

public class SimpleListMainV3 {

    public static void main(String[] args) throws InterruptedException {
//        test(new BasicList());
//        test(new SyncList());
        test(new SyncProxyList(new BasicList()));
    }

    private static void test(SimpleList list) throws InterruptedException {
        log(list.getClass().getSimpleName());

        Runnable addA = () -> {
            list.add("A");
            log("Thread-1 : list.add(A)");
        };

        Runnable addB = () -> {
            list.add("B");
            log("Thread-2 : list.add(B)");
        };

        Thread thread1 = new Thread(addA, "Thread-1");
        Thread thread2 = new Thread(addB, "Thread-2");
        thread1.start();
        thread2.start();
        thread1.join();
        thread2.join();
        log(list);
    }
}
```

### 프록시 구조 분석&#x20;

#### 컴파일 타임 의존 관계&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-28 14.20.50.png" alt=""><figcaption></figcaption></figure>

* 그림과 같이 정적인 클래스의 의존 관계를 정적 의존 관계라 한다.
* `test()` 메서드를 클라이언트라고 가정. `test()` 메서드는 `SimpleList` 라는 인터페이스에만 의존한다.
  * 이것을 추상화에 의존한다고 표현한다.
* 덕분에 `SimpleList` 인터페이스의 구현체인 `BasicList`, `SyncList`, `SyncProxyList` 중에 어떤 것을 사용하든, 클라이언트인 `test()` 의 코드는 전혀 변경하지 않아도 된다.\


#### 런타임 의존 관계&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-04-28 14.24.53.png" alt=""><figcaption></figcaption></figure>

* 런타임에는 실제 자신이 사용하는 객체와 의존 관계를 맺는다.&#x20;
* `test()` 입장에서 컴파일 타임에는 정적인 것에 의존하기에 코드 변경이 없지만, 런타임에는 실제 의존 관계를 맺는 객체의 메서드를 호출하는 것으로 생각하면 된다.&#x20;

#### 프록시 정리&#x20;

* 프록시인 `SyncProxyList`는 원본인 `BasicList` 와 똑같은 `SimpleList` 를 구현한다. 따라서 클라이언트 인 `test()` 입장에서는 원본 구현체가 전달되든, 아니면 프록시 구현체가 전달되든 아무런 상관이 없다. 단지 수 많은 `SimpleList` 의 구현체 중의 하나가 전달되었다고 생각할 뿐이다.
* 클라이언트 입장에서 보면 프록시는 원본과 똑같이 생겼고, 호출할 메서드도 똑같다. 단지 `SimpleList` 의 구현체일 뿐이다.
* 프록시는 내부에 원본을 가지고 있다. 그래서 프록시가 필요한 일부의 일을 처리하고, 그다음에 원본을 호출하는 구조를 만들 수 있다. 여기서 프록시는 `synchronized` 를 통한 동기화를 적용한다.
* 프록시가 동기화를 적용하고 원본을 호출하기 때문에 원본 코드도 이미 동기화가 적용된 상태로 호출된다.

### 프록시 패턴

지금까지 우리가 구현한 것이 바로 프록시 패턴이다.

**프록시 패턴(Proxy Pattern)**&#xC740; 객체지향 디자인 패턴 중 하나로, 어떤 객체에 대한 접근을 제어하기 위해 그 객체의 대리인 또는 인터페이스 역할을 하는 객체를 제공하는 패턴이다. 프록시 객체는 실제 객체에 대한 참조를 유지하면서, 그 객체에 접근하거나 행동을 수행하기 전에 추가적인 처리를 할 수 있도록 한다.

**프록시 패턴의 주요 목적**

* **접근 제어**: 실제 객체에 대한 접근을 제한하거나 통제할 수 있다.
* **성능 향상**: 실제 객체의 생성을 지연시키거나 캐싱하여 성능을 최적화할 수 있다.
* **부가 기능 제공**: 실제 객체에 추가적인 기능(로깅, 인증, 동기화 등)을 투명하게 제공할 수 있다.

> **참고** : 실무에서 프록시 패턴은 자주 사용된다. 스프링의 AOP 기능은 사실 이런 프록시 패턴을 극한으로 적용하는 예이다. 참고로 **스프링 핵심 원리 - 고급편**에서 이 부분을 자세히 다룬다.

## 자바 동시성 컬렉션1 - synchronized

자바가 제공하는 `java.util` 패키지에 있는 컬렉션 프레임워크들은 대부분 스레드 안전(Thread Safe)하지 않다!

우리가 일반적으로 사용하는 `ArrayList`, `LinkedList`, `HashSet`, `HashMap` 등 수 많은 기본 자료 구조들은 내부에서 수 많은 연산들이 함께 사용된다. 배열에 데이터를 추가하고 사이즈를 변경하고, 배열을 새로 만들어서 배열의 크기도 늘리고, 노드를 만들어서 링크에 연결하는 등 수 많은 복잡한 연산이 함께 사용된다.

그렇다면 처음부터 모든 자료 구조에 `synchronized` 를 사용해서 동기화를 해두면 어떨까?&#x20;

`synchronized`, `Lock`, `CAS` 등 모든 방식은 정도의 차이는 있지만 성능과 트레이드 오프가 있다.

결국 **동기화를 사용하지 않는 것이 가장 빠르다.**

그리고 컬렉션이 항상 멀티스레드에서 사용되는 것도 아니다. 미리 동기화를 해둔다면 단일 스레드에서 사용할 때 동기화로 인해 성능이 저하된다. 따라서 동기화의 필요성을 정확히 판단하고 꼭 필요한 경우에만 동기화를 적용하는 것이 필요하다.

> 참고: 과거에 자바는 이런 실수를 한번 했다. 그것이 바로 `java.util.Vector` 클래스이다. 이 클래스는 지금의 `ArrayList` 와 같은 기능을 제공하는데, 메서드에 `synchronized` 를 통한 동기화가 되어 있다.
>
> 그러나 이에 따라 단일 스레드 환경에서도 불필요한 동기화로 성능이 저하되었고, 결과적으로 `Vector`는 널리 사용되지 않게 되었다. 지금은 하위 호환을 위해서 남겨져 있고 다른 대안이 많기 때문에 사용을 권장하지 않는다.

좋은 대안으로는 우리가 앞서 배운 것처럼 `synchronized` 를 대신 적용해 주는 프록시를 만드는 방법이 있다. `List`, `Set`,  `Map` 등 주요 인터페이스를 구현해서 `synchronized`를 적용할 수 있는 프록시를 만들면 된다.

이 방법을 사용하면 기존 코드를 그대로 유지하면서 필요한 경우에만 동기화를 적용할 수 있다.&#x20;

자바는 컬렉션을 위한 프록시 기능을 제공한다.

### 자바 synchronized 프록시&#x20;

`Collections` 가 제공하는 동기화 프록시 기능 덕분에 스레드 안전하지 않은 수 많은 컬렉션들을 매우 편리하게 스레드 안전한 컬렉션으로 변경해서 사용할 수 있다.

```java
package thread.collection.java;

import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

public class SynchronizedListMain {

    public static void main(String[] args) {
//        List<String> list = new ArrayList<>();
        List<String> list = Collections.synchronizedList(new ArrayList<>());
        list.add("data1");
        list.add("data2");
        list.add("data3");
        System.out.println(list.getClass());
        System.out.println("list = " + list);
    }
}
```

#### **synchronized 프록시 방식의 단점**

하지만 `synchronized` 프록시를 사용하는 방식은 다음과 같은 단점이 있다.

* 첫째, 동기화 오버헤드가 발생한다. 비록 `synchronized` 키워드가 멀티스레드 환경에서 안전한 접근을 보장하지만, 각 메서드 호출시마다 동기화 비용이 추가된다. 이로 인해 성능 저하가 발생할 수 있다.
* 둘째, 전체 컬렉션에 대해 동기화가 이루어지기 때문에, 잠금 범위가 넓어질 수 있다. **동기화가 필요 없는 읽기 메서드에서까지 동기화가 이루어진다.**&#x20;
* 셋째, 정교한 동기화가 불가능하다. `synchronized` 프록시를 사용하면 컬렉션 전체에 대한 동기화가 이루어지지만, **메서드 특정 부분만 동기화 하는 것은 불가능하다..**

## 자바 동시성 컬렉션2 - 동시성 컬렉션&#x20;

#### 동시성 컬렉션&#x20;

자바 1.5부터 동시성에 대한 많은 혁신이 이루어졌다. 그 중에 동시성을 위한 컬렉션도 있다. 여기서 말하는 동시성 컬렉션은 스레드 안전한 컬렉션을 뜻한다.

`java.util.concurrent` 패키지에는 고성능 멀티스레드 환경을 지원하는 다양한 동시성 컬렉션 클래스들을 제공한다

* 예를 들어, `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue` 등이 있다.
* 이 컬렉션들 은 더 정교한 잠금 메커니즘을 사용하여 동시 접근을 효율적으로 처리하며, 필요한 경우 일부 메서드에 대해서만 동기화 를 적용하는 등 유연한 동기화 전략을 제공한다.

여기에 다양한 성능 최적화 기법들이 적용되어 있는데, `synchronized`, `Lock`( `ReentrantLock`), `CAS`, 분할 잠 금 기술(segment lock)등 다양한 방법을 섞어서 매우 정교한 동기화를 구현하면서 동시에 성능도 최적화했다.&#x20;

각각의 최적화는 매우 어렵게 구현되어 있기 때문에, 자세한 구현을 이해하는 것 보다는, 멀티스레드 환경에 필요한 동시성 컬렉션을 잘 선택해서 사용할 수 있으면 충분하다.

### 동시성 컬렉션의 종류&#x20;

많으니 사용할 때 찾아보자 ..&#x20;
