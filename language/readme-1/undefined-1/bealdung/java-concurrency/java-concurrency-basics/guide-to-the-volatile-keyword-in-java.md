# Guide to the Volatile Keyword in Java

## 1. 개요&#x20;

* 필요한 동기화가 없다면 컴파일러, 프로세서는 온갖 종류의 최적화를 지원한다. 이러한 최적화는 일반적으로 유익하지만 때로는 미묘한 문제를 일으킬 수 있다.&#x20;
* Java, JVM 은 메모리 순서를 제어하는 여러가지 방법을 제공하며 `Volatile` 또한 그 중 하나이다.&#x20;

## 2. 공유 멀티프로세서와 아키텍처&#x20;

* 프로세서는 프로그램 명령어를 실행하는 역할을 한다. 따라서 RAM 에서 프로그램 명령어와 필요한 데이터를 검색해야 한다.&#x20;
* CPU 는 RAM 에 비해서 초당 많은 명령어를 처리할 수 있으므로, 매번 RAM 에서 데이터를 가져오는 것은 바람직 하지 않다..&#x20;
* 이 상황을 개선하기 위해서 프로세서는 [Out of Order Execution](https://en.wikipedia.org/wiki/Out-of-order\_execution) , [Branch Prediction](https://en.wikipedia.org/wiki/Branch\_predictor) , [Speculative Execution](https://en.wikipedia.org/wiki/Speculative\_execution) , Caching 과 같은 트릭을 사용한다.&#x20;
* 다양한 코어가 더 많은 명령과 데이터를 처리함에 따라서, 그들은 더 관련성 있는 데이터와 명령어로 캐시를 채운다.&#x20;
* 이것은 [**캐시 일관성**](https://en.wikipedia.org/wiki/Cache\_coherence) **문제가 생길 수 있지만, 전반적으로 성능을 향상시킨다.** \
  \-> 캐시 일관성 문제 : 각각의 코어 내 캐시에 담긴 공유 리소스 데이터의 균일성일 깨지는 것을 의미한다.&#x20;
* **하나의 쓰레드가 캐시된 값을 업데이트 할 때 어떤 일이 발생하는지 한번 더 생각해보아야 한다!**

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2024-10-03 17.32.57.png" alt="" width="563"><figcaption></figcaption></figure>

## 3. 캐시 일관성 문제&#x20;

* 아래 `TaskRunner` 는 `number`, `ready` 라는 간단한 변수를 정의한다.&#x20;
* `run()` 메서드 내부에서 `ready` 변수가 `false` 라면 다른 대기중인 쓰레드에게 우선권을 넘겨준다. 그 후 `ready` 변수가 `true` 가 되면 `number` 변수를 출력한다.&#x20;
* **많은 사람들은 짧은 시간 후에 `42` 를 출력할 것이라고 생각할 수 있지만, 지연이 훨씬 길 수도 있다..** \
  **-> 이러한 현상은 적절한 메모리 가시성과 재정렬이 부족하기 때문이다.**&#x20;

```java
public class TaskRunner {

    private static int number;
    private static boolean ready;

    private static class Reader extends Thread {

        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }

            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new Reader().start();
        number = 42;
        ready = true;
    }
}
```

