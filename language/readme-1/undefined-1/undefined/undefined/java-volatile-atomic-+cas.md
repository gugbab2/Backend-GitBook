# Java volatile 과 Atomic 변수(+CAS)

> #### 참고 링크
>
> [https://inkyu-yoon.github.io/docs/Language/Java/Volatile](https://inkyu-yoon.github.io/docs/Language/Java/Volatile)

#### volatile 핵심!

* 쓰레드는 성능 향상을 위해 매번 Main Memory 를 사용하는 것이 아닌 CPU Cache 를 사용해 작업을 진행한다.&#x20;
* 예를 들어 쓰레드 A,B 가 있는 멀티 쓰레드 환경이라고 가정했을 때 A 쓰레드에서  static 변수를 변경하고, 이 변경이 캐시에만 반영되고 메모리에 반영되지 않을 수 있는데, 이 때 B 쓰레드 자신의 캐시에서 이전 값을 계속 사용하게 되면, B 쓰레드는 A 쓰레드의 변경 사항을 모르게 된다. \
  **-> 이를 가시성 문제라고 한다.**&#x20;
* 자바에서는 Cache 를 사용하지 않고 Main Memory 만을 사용하도록 하는 `volatile` 키워드로 이 문제를 해결한다.&#x20;
* 하지만, `volatile` 로 원자성 문제를 해결할 수 없어 결과적으로 동시성 문제도 해결할 수 없게 되는데, 이때 사용할 수 있는 것이 non-blocking 기반 동기화 작업인 CAS 알고리즘을 사용하는 Atomic 변수이다.&#x20;

## 가시성 문제란?&#x20;

* 가시성 문제란, 하나의 스레드에서 공유자원을 수정한 결과가 다른 스레드에게 보이지 않는 문제를 말한다.&#x20;

```java
public class Test{
    public static boolean flag = false; 
    
    private static class A extneds Thread {
        @Override 
        public void run() {
            while(!flag){}
            System.out.println("success");
        }
    }
    
    public static void main(String[] args) InterruptedException {
        final A a = new A(); 
        a.start();
        
        Thread.sleep(200);
        flag = true; 
        a.join(); 
    }
}
```

* 위 코드에서 A 스레드를 실행시키면, 공유 자원인 flag 가 false 값이기 때문에, while 문을 순회하게 된다. 그러다가 메인 스레드에서 flag 를 true 로 변경하게 되면 while 문을 탈출할 것이라고 생각하지만, 실제로는 while 문을 탈출하지 못하는데 이러한 문제를 **가시성 문제**라고 한다.&#x20;
* **멀티 쓰레드에서 쓰레드는 성능 향상을 위해 Main Memory 에서 읽은 변수 값을 CPU Cache 에 저장하게 된다.**&#x20;
  * **즉, 위 예시에서 A 쓰레드는 처음에 메인 메모리에서 flag 값을 false 로 읽어온 뒤, 캐싱한 상태이기 떄문에, 다른 쓰레드가 flag 값을 true 로 변경해도 알아채지 못하고 while 문을 빠져나오지 못하게 되는 것이다.**

#### **이러한 가시성 문제를 해결해 줄 수 있는 키워드가 `volatile` 이다.**

* **공유 자원에 `volatile` 키워드를 사용하면 항상 Main Memory 에 읽고 쓰겠다는 것을 명시하는 것이다.** &#x20;
* `public volatile static boolean flag = false;` 로 코드를 변경후 실행시키면 while 문을 빠져나오는 것을 확인할 수 있다.&#x20;
* **단, `volatile` 키워드를 통해 가시성 문제를 해결하지만, 동시성 문제는 해결하지 못한다.**&#x20;

## Atomic 변수와 CAS 알고리즘

### CAS 알고리즘 동작 방식&#x20;

#### CAS 알고리즘은 다음과 같은 방식으로 동작한다.&#x20;

1. **메모리 위치의 현재 값이 예상 값과 동일한지 확인한다.**&#x20;
2. **동일하다(true)면 메모리 위치의 값을 새 값으로 변경한다.**&#x20;
3. **동일하지 않다면(false) 아무 작업도 수행하지 않고 현재 메모리 위치의 값을 읽고서 1번 과정으로 돌아간다.**  \
   **-> 동일할 때까지 1\~3 과정을 반복한다. (결국에는 동시성 이슈를 해결하고 값을 변경하게 된다)**

### AtomicInteger 살펴보기

```java
public class AtomicIntegerTest {

    private static int count;

    public static void main(String[] args) throws InterruptedException {
        AtomicInteger atomicCount = new AtomicInteger(0);
        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
                atomicCount.incrementAndGet();
            }
        });

        Thread thread2 = new Thread(() -> {
            for (int i = 0; i < 100000; i++) {
                count++;
                atomicCount.incrementAndGet();
            }
        });

        thread1.start();
        thread2.start();

        Thread.sleep(5000);
        System.out.println("atomic 결과 : " + atomicCount.get());    // 200000
        System.out.println("int 결과 : " + count);                   // 152298
    }
}
```

* `AtomicInteger` 타입인 `atomicCount` 는 의도대로 200000 이 출력된 것을 볼 수 있고, `int` 타입인 `count` 는 동기화가 지켜지지 않아 잘못된 값을 출력하는 것을 볼 수 있다.&#x20;

#### 동기화가 어떻게 이루어지는지 `AtomicInteger` 클래스를 살펴보자&#x20;

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-09-04 21.58.23.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-09-04 21.58.43.png" alt="" width="563"><figcaption></figcaption></figure>

* `volatile` 변수를 `value` 로 갖는 것을 볼 수 있다.&#x20;
* 그리고 `compareAndSet` 메서드를 살펴보면 `expectedValue`, `newValue` 가 있는데,&#x20;
  * 인자로 기존값(`expectedValue`)과 변경할 값(`newValue`)을 전달한다.&#x20;
  * 변경할 값을 메모리에 전달하기 전, 메모리에 있는 값과 기존값을 비교해서 일치하는지 확인한다.&#x20;
  * 만약 일치하면 값을 반영하고, 다르다면 후처리를 한다.(후처리는 개발자가 결정)
