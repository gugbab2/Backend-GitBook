# GC Part.2

> #### 참고 링크&#x20;
>
> [https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)\
> [https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)

## 3. GC 알고리즘 종류&#x20;

* Old 영역은 기본적으로 데이터가 가득 차면 GC 를 실행한다. GC 방식에 따라서 처리 절차가 달라지므로 어떤 GC 방식이 있는지 살펴보자.&#x20;
* GC 방식은 JDK 7을 기준으로 5가지 방식이 있다.&#x20;
  * Serial GC
  * Parallel GC
  * Parallel Old GC(Parallel Compaction GC)
  * Concurrent Mark & Sweep GC(이하 CMS)
  * G1(Garbage First) GC
* 위 GC 방식 중 운영 서버에서 절대 사용하면 안되는 방식이 Serial GC 방식이다. Serial GC 는 데스크톱의 CPU 코어가 하나만 있을 때 사용하기 위해서 만든 방식이다. (Serial GC 를 사용하면 애플리케이션 성능이 많이 떨어진다)

### 3-1. Serial GC

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.20.18.png" alt="" width="375"><figcaption></figcaption></figure>

* **mark-sweep-compact** 알고리즘을 사용한다.&#x20;
  1. **mark단계에서는 Root Space 로부터 순회를 통해 사용하고 있지 않은 객체를 마킹한다.**&#x20;
  2. **sweep단계에서는 사용하고 있지 않은 객체의 메모리를 해제한다.(Heap 영역)**&#x20;
  3. **compact단계에서는 메모리 단편화를 방지하기위해 힙의 앞부분부터 객체를 채워 넣는다.**&#x20;
* Minor GC 에서는 mark-sweep, Major GC 에서는 mark-sweep-compact 를 사용한다.&#x20;
* Serial GC 는 서버의 CPU 코어가 1개일때 사용하는 방식
* 싱글쓰레드에서 GC 가 일어나기 때문에, 다른 GC 알고리즘보다 stop-the-world 가 가장 길다..&#x20;
* 보통 실무에서 사용하는 경우는 없다..&#x20;

### 3-2. Parallel GC

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 23.22.29.png" alt="" width="375"><figcaption></figcaption></figure>

* Java 8 Default GC 로 사용&#x20;
* Parallel GC 는 Serial GC 와 기본적인 알고리즘은 같다.&#x20;
* Minor GC 에서는 멀티쓰레드로 동작하지만, Major GC 에서는 싱글쓰레드로 동작한다.&#x20;
* Serial GC 에 비해서 stop-the-world 시간이 감소했다.&#x20;

### 3-3. Paralled Old GC

<figure><img src="../../../.gitbook/assets/스크린샷 2024-09-21 16.34.37.png" alt=""><figcaption></figcaption></figure>

* Paralled Old GC 는 JDK 1.6 부터 제공한 GC 방식이다.&#x20;
* Paralled Old GC 는 Major GC 에서도 멀티쓰레드를 사용한다.&#x20;
* 또한, 추가적으로 GC 알고리즘(Mark-Summary-Compaction) 도 다르다.&#x20;
* mark-summary-compact 방식은 여러 스레드를 사용해서 Old 영역을 탐색한다.&#x20;
  1. **mark단계에서는 Root Space 로부터 순회를 통해 사용하고 있지 않은 객체를 마킹한다.**&#x20;
  2. **summary 단계에서는 메모리를 바로 해제하지 않고, 가비지 메모리의 정보를 요약한다.** \
     **-> 즉, 이 단계에서는 메모리 해제가 직접적으로 일어나지 않고 compact 단계에서 메모리를 해제하기 위한 정보를 수집한다.**&#x20;
  3. **compact단계에서는 메모리 단편화를 방지하기위해 힙의 앞부분부터 객체를 채워 넣는다.** \
     **-> 이 단계에서, 메모리를 해제하는 것이 아닌, 덮어쓰어지면서 간접적으로 메모리가 해제된다.**&#x20;

### 3-4.  Concurrent Mark & Sweep GC(이하 CMS)

<figure><img src="../../../.gitbook/assets/image (123).png" alt=""><figcaption><p>Serial GC, CMS GC 비교</p></figcaption></figure>

* 어플리케이션 쓰레드와 GC 쓰레드가 함께 실행되어, stop-the-world 시간을 최대한 줄이기 위해 고안된 GC&#x20;
* 단, GC 과정이 매우 복잡해졌다.&#x20;
* GC 대상을 파악하는 과정이 다른 GC 에 비해서 매우 복잡하기 때문에, CPU 사용량이 높다는 단점을 가지고 있다.&#x20;
* 메모리 파편화 문제&#x20;
* Java 9 부터는 deprecated 되었고, Java 14 부터는 사용이 중지 되었다.&#x20;

### 3-5. G1 GC

<figure><img src="../../../.gitbook/assets/스크린샷 2024-09-21 16.38.32.png" alt=""><figcaption><p>기 ㄴㅁㅇ</p></figcaption></figure>

* CMS GC 를 대체하기 위해서 JDK 7 부터 release 된 GC&#x20;
* JDK 9 부터 Default GC&#x20;
* 4GB 이상의 힙 메모리, stop-the-world 시간이 0.5초 이상이 경우에 사용 \
  \-> Heap 메모리가 너무 작은 경우 미사용 권장..&#x20;
* 기존 GC 알고리즘에서는 Heap 영역을 물리적으로 고정된 Young / Old 영역으로 구분했지만, \
  **G1GC 는 Region 이라는 새로운 개념을 도입**&#x20;
* **전체 Heap 영역을 Region 으로 분할하여 상황에 따라 Eden, Servivor, Old 등의 역할을 동적으로 부여**
* **Garbage 로 가득찬 region 의 GC 를 수행하므로, 결국 Full GC 빈도가 줄어드는 효과를 얻게 되는 원리** \
  **-> region 별 GC 는 늘어날 수 있다.**&#x20;
