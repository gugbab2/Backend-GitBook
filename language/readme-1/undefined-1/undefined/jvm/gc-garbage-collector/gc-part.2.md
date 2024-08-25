# GC Part.2

## 3. Old 영역에 대한 GC

* Old 영역은 기본적으로 데이터가 가득 차면 GC 를 실행한다. GC 방식에 따라서 처리 절차가 달라지므로 어떤 GC 방식이 있는지 살펴보자.&#x20;
* GC 방식은 JDK 7을 기준으로 5가지 방식이 있다.&#x20;
  * Serial GC
  * Parallel GC
  * Parallel Old GC(Parallel Compaction GC)
  * Concurrent Mark & Sweep GC(이하 CMS)
  * G1(Garbage First) GC
* 위 GC 방식 중 운영 서버에서 절대 사용하면 안되는 방식이 Serial GC 방식이다. Serial GC 는 데스크톱의 CPU 코어가 하나만 있을 때 사용하기 위해서 만든 방식이다. (Serial GC 를 사용하면 애플리케이션 성능이 많이 떨어진다)

### 3-1. Serial GC

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2023-06-08 23.20.18.png" alt="" width="375"><figcaption></figcaption></figure>

* Young 영역에서의 GC 는 앞 절에서 설명한 방식을 사용한다. Old 영역의 GC 는 mark-sweep-compact 라는 알고리즘을 사용한다.&#x20;
  1. mark단계에서는 old영역에서 살아있는 객체를 확인한다.
  2. sweep단계에서는 heap영역의 **앞부분부터 확인하여** 표시되지 않은객체를 제거한다.
  3. compact단계에서는 메모리 단편화를 방지하기위해 힙의 앞부분부터 객체를 채워 넣는다.&#x20;
* Serial GC 는 적은 메모리와 CPU 코어 개수가 적을 떄 적합한 방식이다.&#x20;

### 3-2. Parallel GC

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2023-06-08 23.22.29.png" alt="" width="375"><figcaption></figcaption></figure>

* Parallel GC 는 Serial GC 와 기본적인 알고리즘은 같다.&#x20;
* 하지만, Serial GC 는 GC 를 처리하는 스레드가 하나인 것에 비해, Parallel GC 는 GC 를 처리하는 스레드가 여러개이다. 때문에, Serial GC 보다 빠르게 객체를 처리할 수 있다.&#x20;
* Parallel GC 는 메모리가 충분하고 코어의 개수가 많을 때 유리하다.&#x20;

### 3-3. Paralled Old GC

* Paralled Old GC 는 JDK 5 update 6 부터 제공한 GC 방식이다.&#x20;
* 앞서 설명한 Parallel GC 와 비교해서 Old 영역의 GC 알고리즘(Mark-Summary-Compaction)만 다르다.&#x20;
* mark-sweep-compact 방식이 단일 스레드가 Old 영역을 검사하는 방식이라면, mark-summary-compact 방식은 여러 스레드를 사용해서 Old 영역을 탐색한다.&#x20;
  1. mark단계에서는 old영역을 region별로 나누고 region별로 살아있는 객체를 식별한다.
  2. sweep단계에서는 **ragion 을 확인하여** 표시되지 않은 객체를 제거한다.
  3. compact단계에서는 메모리 단편화를 방지하기위해 힙의 앞부분부터 객체를 채워 넣는다.&#x20;

### 3-4.  Concurrent Mark & Sweep GC(이하 CMS)

<figure><img src="../../../../../../.gitbook/assets/image (123).png" alt=""><figcaption><p>Serial GC, CMS GC 비교</p></figcaption></figure>

* 실행 순서는 다음과 같다.&#x20;
  1. Initial Mark(stop-the-world) : 이 단계에서는 Old 영역에 있는 모든 객체 중 GC 루트와 직접 연결된 객체를 마킹한다. (살아있는 객체)
  2. Concurrent Mark : 애플리케이션 스레드와 병행하여 Initial Mark 단계에서 마킹된 객체들을 참조하는 객체들을 따라가면서 마킹한다.&#x20;
  3. Remark( stop-the-world) : 이전 단계에서 놓친 살아있는 객체들을 확인해 마킹한다.&#x20;
  4. Concurrent Sweep : 이전 과정을 통해 마킹되지 않은 객체들의 메모리를 해제한다.
  5. Concurrnet Reset : 다음 GC 사이클을 준비하기 위해서 내부 데이터 구조를 초기화한다.&#x20;
* CMS GC 는 stop-the-world 시간이 매우 짧다. 때문에 모든 애플리케이션의 응답 속도가 매우 중요할 때 사용하며, Low Latency GC 라고 부른다.&#x20;
* 하지만, CMS GC 는 stop-the-world 시간이 짧다는 장점에 반해 다음과 같은 단점이 존재한다.&#x20;
  * 다른 GC 보다 메모리와 CPU 를 더 많이 사용한다.&#x20;
  * Compaction 단계가 기본적으로 제공되지 않는다.&#x20;
* 만약 조각난 메모리가 많아, Compaction 작업을 실행하면 다른 GC  방식의 stop-the-world 시간보다 stop-the-world 시간이 더 길기 때문에 Compaction 작업이 얼마나 자주, 오랫동안 수행되는지 확인해야 한다.&#x20;

### 3-5. G1 GC

<figure><img src="../../../../../../.gitbook/assets/image (124).png" alt="" width="360"><figcaption></figcaption></figure>

* G1 GC 를 이해하기 위해서는 지금까지의 Young 영역과 Old 영역에 대해서는 잊는 것이 좋다.&#x20;
* 다음 그림에서 보다시피, G1 GC 는 바둑판의 각 영역에 객체를 할당하고 GC 를 실행한다. 그러다가 해당 영역이 꽉 차면 다른 영역에 객체를 할당하고 GC 를 실행한다.&#x20;
* 즉, 지금까지 설명한 Young 의 세가지 영역에서 데이터가 Old 영역으로 이동하는 단계가 사라진 GC 방식이라고 이해하면 된다.&#x20;
* G1 GC 는 장기적으로 말도 많고, 탈도 많은 CMS GC 를 대체하기 위해서 만들어졌다.&#x20;
* G1 GC 의 가장큰 장점은 성능이다. 지금까지 설명한 어떤 GC 방식보다 빠르다!&#x20;
* JDK 9 부터 기본 GC 로 사용하고 있다.&#x20;
