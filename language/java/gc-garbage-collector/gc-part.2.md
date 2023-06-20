# GC Part.2

## 5. GC 의 실행방식&#x20;

* Mark And Sweep 방식의 두 번째 특징은 애플리케이션과 GC 실행이 병행된다는 것이다. \
  \-> 즉 JVM 에서 애플리케이션과 GC 를 병행하여 실행할 수 있는 여러 옵션을 제공한다.

### 5-1. Serial GC&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.20.18.png" alt="" width="375"><figcaption></figcaption></figure>

* Serial GC 는 하나의 스레드로 GC 를 실행하는 방식인데, 하나의 스레드로 GC 를 실행하다 보니 stop-the-world 시간이 긴 것을 알 수 있다.&#x20;
* 싱글 스레드 환경 및 Heap 영역이 매우 작을 때 사용되는 방식이다.&#x20;
* 참고로 Mark And Sweep 이후 메모리 파편화를 막는 Compaction 과정도 진행된다.&#x20;

### 5-2. Paralled GC

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.22.29.png" alt="" width="375"><figcaption></figcaption></figure>

* Paralled GC 의 기본적인 처리과정은 Serial GC 와 동일하지만, 여러개의 스레드로 GC 를 실행하기 때문에, Serial GC 보다 stop-the-world 시간이 짧아진 것을 알 수 있다.&#x20;
* 멀티 코어 환경에서 애플리케이션 속도를 향상시키기 위해서 사용되며, Java8에서 기본적으로 쓰이는 방식이다.
* 일반적인 Paralled GC 는 minor gc 에 대해서만 멀티스레딩을 사용하고 , major gc 는 싱글 스레딩으로 수행한다.&#x20;

### 5-3. Paralled Old GC

* Paralled GC 가 GC 오버헤드를 상당히 줄여주었지만, stop-the-world 는 피할 수 없다.&#x20;
* 때문에, 더욱 발전된 방식이 Paralled Old GC 방식이다. \
  \-> major gc 도 멀티 스레딩으로 수행하고,\
  \-> 기존 Mark Sweep Compation 의 개선 버전인 Mark Summary Compaction 을 사용한다.&#x20;
* 사실상 Java 7 Update 4 버전부터는 Paralled GC 를 설정해도, Paralled Old GC 가 동작한다. \
  \-> 엄밀히 말하면 Java 8 의 디폴트 버전은 Paralled Old GC 인 셈이다.&#x20;

### 5-4. CMS GC

* Application 의 스레드와 GC 스레드가 동시에 실행되어서 stop-the-world 를 최소화하는 GC 이다.\
  \-> 때문에, 메모리와 CPU 를 많이 사용한다.&#x20;
* 하지만, Compaction 기능을 제공하지 않아 장기적 운영적인 측면에서 단점이 많아 Deprecated.

## 6. G1GC

### 6-1. G1GC 란?

* Heap 영역을 Region 으로 잘게 나누었다.&#x20;
* 빠른 처리 속도를 지원하면서 stop-the-world 최소화하며, Application 의 스레드와 GC 스레드가 동시에 실행된다.
* 메모리 Compaction 작업까지 지원한다.&#x20;
* 자바 9 버전부터 기본 GC 방식으로 채택되었다.&#x20;

### 6-2. G1GC 장단점

* 장점&#x20;
  * 별도의 stop-the-world 없이도 Compaction 기능을 제공한다.&#x20;
  * Old/Young 영역을 나눠서 Compaction 할 필요가 없고, 해당 Generation 의 일부분에 대해서만 Compaction 을 진행한다.&#x20;
  * Heap 크기가 클수록 잘 동작한다.
  * CMS 이 비해서 개선된 알고리즘을 사용하고, 처리속도가 빠르다.&#x20;
  * Garbage 로 가득찬 영역을 빠르게 회수하여 빈 공간을 확보하므로 GC 빈도가 줄어든다.
* 단점
  * 공간이 부족한 상태를 조심해야 한다. \
    \-> 이 때 Full GC 가 발생하는데, 이 GC 는 Single Thread 로 동작한다.\
    \-> Full GC 는 Heap 전반적으로 GC 가 발생하는 것을 뜻한다.&#x20;
  * 작은 Heap 공간을 가지는 Application 에서는 제 성능을 발휘하지 못하고 Full GC 가 발생한다.&#x20;

### 6-3. G1GC Heap 구조

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.41.26.png" alt="" width="286"><figcaption></figcaption></figure>

* G1GC 는 기존 힙 구조와 다르게, Young/Old 영역을 명확하게 구분하지 않는다.
* G1GC 는 개념적으로 그들이 존재하나, 일정 크기의 논리적 단위인 region 으로 구분한다.&#x20;

### 6-4. G1GC 동작과정

#### 6-4-1. Minor GC

* Minor GC 기존 GC 와 원리가 비슷하나, 멀티 스레드에서 병렬로 동작한다.&#x20;
* 연속되지 않은 공간에, Young 영역이 Region 단위로 메모리에 할당되어 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.44.56.png" alt="" width="375"><figcaption></figcaption></figure>

* Eden 영역이 다 차게되면 GC 가 발생하고, Young 영역에 있는 유효객체를 Survivor 영역이나, Old 영역으로 이동한다.&#x20;
* Minor GC 를 모두 마친 후 모습이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.46.03.png" alt="" width="375"><figcaption></figcaption></figure>

#### 6-4-2. Major GC

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.52.45.png" alt="" width="375"><figcaption></figcaption></figure>

* Initial Mark
  * Initial Mark 단계는 Old Region 에 존재하는 객체들이 참조하는 Survivor Region 이 있는지 파악해서 Suvivor Region 에 마킹하는 단계이다.
  * Survivor Region 에 의존적인 상태이고 때문에, Minor GC 가 전부 끝난 상태여야 한다. \
    \-> 따라서 Initial Mark 는 Minor GC 에 의존적이며, stop-the-world 를 발생시킨다.
*   &#x20;Root Region Scan

    * Initial Mark 단계에서 마킹된 Survivor Region 에서 Old Region 에 대해 참조하고 있는 객체를 마킹한다.&#x20;
    * 멀티 스레드로 동작하며, 다음 Minor GC 가 발생하기 전에 동작을 완료한다.

    <figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.59.19.png" alt="" width="375"><figcaption></figcaption></figure>
* Concurrent Marking&#x20;
  * Old 영역 내 생존해 있는 모든 객체를 마킹한다.&#x20;
  * stop-the-wolrd 가 발생하지 않으므로, Application 스레드와 동시에 동작하고,
  * Minor GC 와 같이 진행되므로 종종 Minor GC 에 의해서 stop-the-world 가 발생될 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-09 00.05.35.png" alt="" width="375"><figcaption></figcaption></figure>

* Remark&#x20;
  * Concurrent Marking 에서 X 표시한 영역을 회수하며 stop-the-world 가 발생한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-09 00.06.57.png" alt="" width="375"><figcaption></figcaption></figure>

*   Copying/Cleanup&#x20;

    * Copying/Cleanup 단계에서 Live Object 비율이 낮은 영역 순으로 순차적으로 GC 가 수행되며, \
      stop-the-world 가 발생한다.
    * GC 수행 시 해당 영역의 Live Object 를 다른 영역으로 이동 후 Garbage 를 수집한다. &#x20;

    <figure><img src="../../../.gitbook/assets/스크린샷 2023-06-09 00.16.30.png" alt="" width="375"><figcaption></figcaption></figure>
* Compaction&#x20;
  * Major GC 가 끝난 후  Live Object 가 새로운 Region 으로 이동하고 메모리 Compaction 이 일어난다.&#x20;
