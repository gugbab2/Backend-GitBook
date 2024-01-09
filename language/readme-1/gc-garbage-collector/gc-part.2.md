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

## 6. G1GC

### 6-1. G1GC 란?

* 대규모 힙 영역을 효율적으로 관리하기 위해서 설계되었다.&#x20;
* 힙을 영역으로 구분했다.
  * 전체 힙을 논리적인 작은 크기의 영역으로 분할한다.&#x20;
  * 각 영역은 young, old, Humongous(큰 객체나, 배열을 관리하는 영역) 영역으로 구분된다.&#x20;
* **영역 기반 수집**
  * &#x20;G1GC는 전체 힙을 작은 영역으로 분할합니다. 이렇게 분할된 영역들 중에서 **가장 가비지가 많은 영역부터 수집을 진행합니다.**&#x20;
  * **이 방식을 통해서 힙 전체에 일어나는 GC 를 분산시켜 stop the world 를 최소화했다.**&#x20;
* **병렬 처리**
  * &#x20;G1GC는 다중 스레드를 사용하여 가비지 컬렉션 작업을 병렬로 처리합니다. 이를 통해 가비지 컬렉션 작업을 빠르게 수행하고 일시 중지 시간을 최소화합니다.
* **Mixed GC**
  * &#x20;G1GC에서는 Mixed GC라고 불리는 통합된 가비지 컬렉션을 수행합니다. Mixed GC는 Young 영역과 Old 영역을 동시에 수집하는 방식으로, 전체 힙을 한 번에 처리하므로 일시 중지 시간을 최소화할 수 있습니다.
* **일시 중지 시간 조절**
  * &#x20;G1GC는 일시 중지 시간을 조절하기 위해 목표 일시 중지 시간을 설정할 수 있습니다. G1GC는 설정된 목표 일시 중지 시간을 지키면서 가비지 컬렉션을 수행하며, 일시 중지 시간이 넘어가지 않도록 작업을 조절합니다.
* **자바 9 버전부터 기본 GC 방식으로 채택되었다.**&#x20;

### 6-2. G1GC Heap 구조

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.41.26.png" alt="" width="286"><figcaption></figcaption></figure>

* G1GC 는 기존 힙 구조와 다르게, Young/Old 영역을 명확하게 구분하지 않는다.
* G1GC 는 개념적으로 그들이 존재하나, 일정 크기의 논리적 단위인 region 으로 구분한다.&#x20;
