# synchronized 키워드 이해도 체크

## 0. 1 - 40억까지 더하는 코드를 n 개의 스레드로 병렬로 돌아가도록 짜 보세요.

* **각 스레드가 서로 다른 범위를 나눠서 합계를 구하고**,
* **main 스레드는 join()으로 모든 스레드가 끝날 때까지 기다리는 방식**을 사용합니다.

```java
Thread t1 = new Thread(() -> { /* 1 ~ 10억 합 */ });
Thread t2 = new Thread(() -> { /* 10억+1 ~ 20억 합 */ });
...
t1.start();
t2.start();

t1.join();  // main 스레드는 여기서 대기
t2.join();
```

* **join()**: 스레드가 끝날 때까지 **main 스레드가 대기**하게 만듭니다.
* 이렇게 하면 **main 스레드가 모든 작업이 끝난 뒤 결과를 처리**할 수 있어요.

## 1. synchronized 메서드의 동기화 대상은 누구인가?&#x20;

* **인스턴스 메서드에 synchronized를 붙이면 → 그 인스턴스(this)의 모니터 락**이 걸립니다.

```java
public synchronized void instanceMethod() { 
    // this 객체에 락이 걸림
}
```

* 같은 인스턴스의 synchronized 메서드는 **동시에 실행되지 않아요.**
* 하지만 **다른 인스턴스끼리는 서로 독립적**으로 실행됩니다.

## 2. static synchronized 메서드의 경우 1과 같은가 다른가?&#x20;

* **static synchronized 메서드**는\
  → **그 클래스의 Class 객체에 락을 걸어요.**

```java
public static synchronized void staticMethod() { 
    // MyClass.class 객체에 락이 걸림
}
```

* 여기서 **MyClass.class**는 **JVM에서 해당 클래스 정보를 담는 객체(java.lang.Class 타입)**&#xC785;니다.

**🔍 인스턴스 메서드와의 차이:**

| 메서드 유형                               | 락 대상                             |
| ------------------------------------ | -------------------------------- |
| **인스턴스 메서드 (synchronized)**          | **this 인스턴스의 모니터 락**             |
| **static 메서드 (static synchronized)** | **클래스 객체(MyClass.class)의 모니터 락** |

* 그래서 **인스턴스 락과 클래스 락은 서로 간섭하지 않습니다.**

## 3. synchronized block 의 괄호 안에는 무엇을 넣어줘야 할까?

* **괄호 안에는 락을 걸 대상(Object)** 을 넣어야 해요.
* 즉, **모니터 락을 걸고 싶은 객체**를 지정하는 거예요.

```java
synchronized (lockObject) {
    // lockObject의 모니터 락을 획득한 후 실행
}
```

* **lockObject**로는:
  * **this** → 인스턴스 락
  * **MyClass.class** → 클래스 락
  * **new Object()** → 별도 락 전용 객체
