# G1GC

#### 참고 링크&#x20;

{% embed url="https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html" %}

## 1. Garbage-First(G1) 가비지 컬렉터 소개 <a href="#jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42" id="jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42"></a>

JDK 9 부터 G1 은 Default GC 이다.

### 1.1 G1GC 의 목표

1. **예측 가능한 짧은 멈춤 시간 (Pause Time Goals)**&#x20;
   1. G1GC 는 "사용자가 요청한 최대 멈춤 시간" 을 지키는 게 최우선 목표이다. \
      (기본값 : `-XX:MaxGCPauseMills=200`)
2. **대용량 Heap 효율적 관리**&#x20;
   1. 과거의 GC(Parallel GC, CMS) 는 Heap 이 클수록 멈춤 시간이 길어진다. \
      (전체 메모리를 관리하기 때문)&#x20;
   2. G1GC 는 Heap 을 **작은 Region(1\~32MB)** 으로 잘라 관리 \
      (전체 메모리를 작은 리전 단위로 관리하면 된다)&#x20;
3. **단편화(Fragmentation) 방지**
   1. CMS 는 Compaction 이 없어서 오래 실행하면 메모리 단편화가 발생.
   2. G1 은 객체를 재배치(Evacuation) 해 Heap 을 연속된 공간으로 유지 -> 단편화(Fragmentation) 방지&#x20;
4. **Throughput 과 Pause Time 의 균형**&#x20;
   1. 멈춤 시간을 줄이면서도 애플리케이션 처리량(Throughput) 을 크게 희생하지 않도록 최적화&#x20;
   2. GC 작업을 나누어 점진적으로 수행해 애플리케이션 성능 저하 최소화

### 1.2 G1GC 트레이드 오프&#x20;

#### ✅ G1GC 는 다른 처리량 중심 GC 보다 처리량이 떨어진다.&#x20;

* G1GC 는 STW 를 짧고 예측 가능하게 만들기 위해서 Heap 을 작은 Region 단위로 나누어 관리한다. 이로 이해 GC 작업이 더 자주 발생할 수 있고, GC 마다 관리해야 할 메타데이터 오버헤드도 증가한다.
* 반면 Parallel GC 같은 처리량 중심 GC 는 한 번에 Heap 전체를 회수하여 GC 호출 횟수를 줄이고 처리량을 극대화한다.&#x20;
* 결론 : G1GC 는 처리량 측면에서는 처리량 중심 GC 에 비해서 낮을 수 있지만, STW 이 짧아 안정적인 서비스 제공에 초점을 맞춘다. (STW 가 짧아야, 사용자 경험이 좋아진다)&#x20;

#### ✅ 처리량이 줄어들면 결국 OOM 이 발생할 수 있는데, 처리량을 줄이는 것이 의미가 있는가?&#x20;

* 처리보다 할당이 빨라 GC 가 메모리를 충분히 회수하지 못하면 OOM 이 발생할 위험이 있다. &#x20;
* G1GC 는 이를 방지하기 위해서 다음과 같은 대응 전략을 가지고 있다.&#x20;
  * Pause Time 목표 유지 (최우선)&#x20;
  * Heap Pressure 증가 시 Pause Time 목표를 일부 깨고 추가 Region 회수&#x20;
  * 그래도 부족하면 Full GC(STW) 수행으로 메모리 확보 \
    (Full GC 발생 시 STW 시간이 상당히 길기 때문에, 튜닝을 통해 Full GC 가 발생하지 않도록 해야한다)&#x20;
* 결론 : 처리량을 줄여 STW 가 짧아지면 사용자 경험이 좋아지기 때문에, 의미가 있다! 하지만 Full GC 가 발생할 수 있기 때문에, 이 경우 튜닝을 통해 할당보다 처리량이 많아지도록 조정해야 한다.&#x20;

## 2. 기본 개념&#x20;

G1 은 각 STW 시 목표 중지 시간을 모니터링 한다.

다른 컬렉터와 마찬가지로 G1 은 힙의 리전을 신세대(에덴, 서바이버1,2)와 구세대로 나눈다. 공간 회수 작업은 가장 효율적인 신세대에 집중되며, 구세대에서도 간헐적으로 공간 회수가 수행된다.

일부 작업은 처리량 향상을 위해서 STW 에서 수행한다. 자원을 많이 사용하는 작업에 대해서는 애플리케이션과 병렬적으로 진행한다. 공간 회수를 위한 STW 을 짧게 하기 위해서 G1 은 공간 회수를 단계적으로 병렬로 수행한다.&#x20;

G1 은 이전 애플리케이션의 동작 및 STW 에 대한 정보를 추적하여 관리 비용 모델을 구축함으로 예측 가능성을 확보한다. 이 정보를 사용해 STW 시간 목표를 달성하기 위한 작업의 크기를 조정한다.&#x20;

### 2.1 힙 레이아웃&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-07-20 22.38.37.png" alt="" width="563"><figcaption></figcaption></figure>

G1은 힙을 크기가 동일한 여러 개의 힙 영역으로 분할한다. 각 영역은 연속적인 가상 메모리 범위이며, 할당 및 회수의 단위이다. 특정 시점에 이러한 각 영역은 비어 있거나 특정 세대(신세대, 구세대) 에 할당될 수 있다.&#x20;

애플리케이션은 기본적으로 항상 신세대에 객체를 할당한다. 거대(Humongous) 객체의 경우 구세대에 직접 할당한다.&#x20;

### 2.2 가비지 수거 사이클

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-07-21 09.26.18.png" alt="" width="563"><figcaption></figcaption></figure>

> #### G1GC 핵심 데이터 구조&#x20;
>
> * **Region (영역)**&#x20;
>   * 힙을 구성하는 가장 기본적인 단위로, 기본 크기는 1MB 에서 32 MB 까지 설정될 수 있다.&#x20;
>   * G1GC 는 이 영역들을 독립적으로 GC 대상으로 삼고, Young, Old, Humongus(매우 큰 객체) 등의 역할을 동적으로 할당한다.&#x20;
> * **Remembered Set (RSet)**&#x20;
>   * 각 영역(Region)마다 존재하는 데이터 구조로, 다른 영역에서 해당 영역으로 들어오는 참조(Incoming References)를 기록한다.
>     * Old → Young 참조 관리: Old 영역의 객체가 Young 영역의 객체를 참조하는 경우, 해당 Young 영역의 RSet에 이 참조 정보가 기록된다. 이는 Minor GC(Young GC) 시 Old 영역 전체를 탐색하지 않고도 RSet만 확인하여 살아있는 Young 객체를 효율적으로 추적할 수 있도록 돕는다.
>     * Young → Old 참조 관리: Young 영역의 객체가 Old 영역의 객체를 참조하는 경우에도 해당 Old 영역의 RSet에 이 참조가 기록된다. Initial Mark 및 Concurrent Mark 단계에서는 이 RSet을 사용해 Young → Old 참조를 추적하고 Old 영역 객체의 생존 여부를 마킹하여 객체 그래프의 일관성을 유지한다.
>     * **결론: RSet은 Old → Young뿐만 아니라 Young → Old 참조까지 관리하여 Minor GC와 Major GC 모두에서 영역 간 참조를 빠르고 효율적으로 처리할 수 있도록 한다.**
> * **Card Table (카드 테이블)**&#x20;
>   * 힙 전체를 작은 카드(Card) 단위(일반적으로 512바이트)로 나눈 비트맵이다.
>     * 역할: 각 카드는 특정 메모리 범위를 나타내며, 해당 카드 내의 객체에서 다른 영역으로 참조가 수정될 때 카드를 더티(Dirty) 상태로 표시한다.
>     * RSet 업데이트와의 연관성: RSet 업데이트는 이 Card Table을 통해 간접적으로 이루어진다. 사용자 스레드에서 Write Barrier 코드를 감지하면 해당 카드를 더티 상태로 바꾸고, 이후 GC 스레드가 GC 이전에 더티 카드를 참고해 RSet 을 업데이트 한다.&#x20;
> * **Collection Set (CSet)**&#x20;
>   * Mixed GC(Minor 또는 Major GC) 시 실제로 GC 대상이 되는 영역(Region)들의 집합이다.
>     * 역할: G1GC는 CSet에 포함된 영역들만 GC를 수행하며, 이는 G1GC가 설정된 STW 목표를 준수하기 위해 어떤 영역을 GC할지 선택하는 기준이 된다.
>     * 처리: 살아있는 객체는 CSet 외부의 다른 영역으로 복사되고, CSet에 포함된 영역들은 완전히 비워져 재사용된다.
> * **Free List (프리 리스트)** : GC 후에 완전히 비워진 영역(Region) 들을 관리하는 리스트이다. 새로운 객체를 할당하거나 GC 과정에서 살아있는 객체를 복사할 때, 이 Free List 에서 빈 영역을 가져와 사용한다.&#x20;
> * **Top-At-Mark-Start (TAMS)**&#x20;
>   * Concurrent Marking이 시작될 때 각 영역(Region)의 할당 포인터 위치를 나타내는 값이다.
>     * 역할: G1GC는 SATB 알고리즘을 사용하여 마킹을 수행하는데, TAMS는 마킹 시작 지점 이후에 새로 할당된 객체들(즉, TAMS 상단에 있는 객체들)은 기본적으로 살아있는 것으로 간주하고 GC 대상에서 제외한다. 이를 통해 동시성 마킹 중에도 애플리케이션 스레드가 자유롭게 객체를 할당할 수 있도록 돕는다.
> * **Snapshot-At-The-Beginning (SATB)**&#x20;
>   * G1GC의 Concurrent Mark 단계에서 사용하는 핵심 알고리즘이다.
>     * 역할: GC 마킹이 시작되는 시점에 객체 그래프 "스냅샷"을 찍고, 이 스냅샷을 기준으로 살아있는 객체를 판단한다. 마킹 진행 중에 애플리케이션 스레드가 객체 참조를 변경하더라도, SATB는 변경되기 전의 스냅샷을 기반으로 살아있는 객체를 간주한다.
>     * Write Barrier와의 연관성: 이를 위해 Write Barrier를 사용해 마킹 주기 동안 변경되는 참조들(특히 변경되기 전의 참조)을 버퍼에 기록하고, Remark 단계에서 이 버퍼들을 처리한다.
> * **Write Barrier**&#x20;
>   * 애플리케이션 스레드가 객체의 참조를 변경할 때 실행되는 특별한 코드로, GC가 참조 변경을 추적할 수 있도록 돕는다.
>     * Card Table 업데이트: 참조가 변경된 메모리 블록(카드)을 Dirty로 표시한다.
>     * RSet 업데이트:
>       * Old → Young 참조가 새롭게 생기면 해당 Young Region의 RSet에 등록(Minor GC 시 사용).
>       * Young → Old 참조가 새롭게 생기면 해당 Old Region의 RSet에 등록(Initial Mark/Concurrent Mark 시 사용).
>     * SATB(Snapshot-At-The-Beginning): Concurrent Mark 중 참조 변경 시 변경 전(old reference)를 버퍼에 기록한다.

#### 1. Young-Only Phase (Minor GC)&#x20;

* **GC 시점 :** Young 영역(대부분은 Eden 영역)이 가득 차서 새로운 객체를 할당할 공간이 부족할 때 발생한다. 이 과정은 애플리케이션 스레드가 일시 중지되는 STW 상태에서 진행된다.
* **GC 순서 :**&#x20;
  * **1. CSet 에 대상 Region 추가** :&#x20;
    * Minor GC 의 경우, **Young 영역 전체가 대상이 된다.**&#x20;
    * G1GC 는 설정된 목표 STW 시간을 맞추기 위해 가장 효율적으로 가비지를 수집할 수 있는 Region 들을 CSet 에 포함시킨다.
    * **Young 영역의 특성: Young 영역은 대부분의 객체가 금방 죽는다는 "약한 세대 가설"에 기반합니다. 따라서 `Minor GC`가 발생하면 Young 영역 전체를 스캔하고 수집하는 것이 기본 전제입니다.**
  * **2. 루트(GC Root) 스캔** :&#x20;
    * 스레드 스택의 로컬 변수, 메서드 영역의 정적 변수, JNI 참조 들과 같은 **GC 루트에서 직접 접근 가능한 객체들을 찾아내고 살아있는 객체로 마킹한다.**&#x20;
    * 단순히 GC 루트 객체만 보는 것이 아니라, **도달 가능한 모든 객체 그래프를 재귀적으로 탐색해 살아있는 객체들을 마킹한다.**
  * **3. RSet 활용**:&#x20;
    * Old 영역의 객체가 Young 영역의 객체를 참조하는 경우를 처리하기 위해 각 Young 영역의 RSet 을 확인한다.&#x20;
    * RSet 에는 Young 영역 을 참조하는 Old 영역의 정보가 담겨있다.&#x20;
    * 이 정보를 통해 G1GC 는 Old 영역 전체를 스캔하는 대신, **RSet 에 기록된 참조만 확인해 Old -> Young 참조에 의해 살아있는 Young 객체들을 찾아내고 마킹한다.**&#x20;
  * **4. Evacuation (복사)**:&#x20;
    * 앞서 마킹된(살아있는) 객체들은 새로운 Survivor 영역이나 Old 영역으로 복사된다.&#x20;
    * 복사 후, 기존의 Young 영역은 지우고, **해당 영역들은 Free List 에 추가되어 다음 객체 할당에 재사용된다.**&#x20;

2. Space-Reclamation Phase (Old 영역 마킹 및 Mixed GC)&#x20;

* **GC 시점 :** Old 영역의 점유율이 Initiation Heap Occupancy Percent (IHOP) 임계값(기본값 45%)에 도달하면 G1GC는 Old 영역 마킹을 위한 마킹 사이클을 시작한다. 이 사이클은 동시성(Concurrent) 작업과 STW 작업이 혼재되어 진행된다.
* **GC 순서 :**&#x20;
  * **1. Initial Mark (초기 마킹 - STW)**&#x20;
    * 목적 : GC 루트에서 **직접적으로 접근 가능한 살아있는 객체들을 식별하고 마킹한다.**&#x20;
  * **2. Root Region Scanning (루트 영역 스캔 - Concurrent)**&#x20;
    * 목적 : Initial Mark 에서 마킹된 객체들을 시작으로, **Young 영역(주로 Survivor Region) 에서 Old 영역으로의 참조(Young -> Old)**&#xB97C; 스캔하여 추가적으로 살아있는 Old 영역 객체를 찾아 마킹한다. 이는 GC 루트 집합을 확장하고 Concurrent Marking 을 위한 정환한 시작점을 제공한다.
    * 특징 :&#x20;
      * RSet(Remembered Set) 활용 : 이 단계에서 Old Region 의 RSet 이 활발하게 사용된다. Young 영역에서 Old 영역으로의 참조가 발생하면 해당 Old Region 의 RSet 에 기록된 정보를 통해 Old 객체를 Root 로 간주하여 마킹할 수 있다.&#x20;
      * Card Table 활용 : Old 영역의 Card Table 은 RSet 을 업데이트하는 데 사용된다.&#x20;
  * **3. Concurrent Marking (동시성 마킹 - Concurrnet)**&#x20;
    * **목적** : 힙 전체를 스캔하며, 이전 단계에서 마킹된 객체들을 시작으로 **도달 가능한(Reachable) 모든 살아있는 객체들을 찾아서 마킹**한다. 이 단계에서 Old 영역의 실제 가비지를 식별하는 작업이 이루어진다.&#x20;
    * **특징 :**&#x20;
      * **SATB(Snapshop-At-The-Beginning) 알고리즘 :** 이 알고리즘을 사용하여 마킹 시작 시점의 힙 스냅샷을 시준으로 살아있는 객체를 판단한다. Concurrent Marking 단계중 어플리케이션 스레드에 의해 객체 참조가 변경된 경우 SATB 버퍼에 변경 이전 참조값을 저장한다. 이렇게 기록된 객체들은 변경되었더라도 일단 살아 있는 것으로 간주되어 마킹한다.
      * **TAMS(Top-At-Mark-Start) 의 중요성** :  각 Region의 `TAMS` 값 이후에 할당된 객체(즉, `Concurrent Mark` 시작 시점 이후에 새로 생성된 객체)는 이 마킹 사이클의 대상이 아니므로 별도의 스캔 없이 자동으로 살아있는 것으로 간주된다. `TAMS` 이전에 존재하던 객체들만 마킹 대상이 된다.
  * **4. Remark (재마킹 - STW)**
    * 목적 : `Concurrent Marking` 단계에서 `Write Barrier`를 통해 `SATB 버퍼`에 기록된 참조 변경 정보들을 최종적으로 처리하고, 애플리케이션 스레드가 동시성 마킹 중에 변경한 참조들 때문에 누락되었을 수 있는 살아있는 객체를 찾아 마킹한다. **이 단계에서 최종적으로 살아있는 객체 집합이 확정된다.**
    * 특징 :&#x20;
      * SATB 버퍼 처리: 이 단계에서 GC는 `Concurrent Marking` 동안 `Write Barrier`가 `SATB 버퍼`에 기록한 객체 참조 변경 목록을 처리한다.&#x20;
  * **5. Cleanup (정리 - STW & Concurrent)**&#x20;
    * 목적 :
      * **완전히 비어있는 영역 회수** : 마킹 정보를 기반으로 완전히 비어있는 영역(Region)들을 식별하고 즉시 회수한다. 회수된 영역들은 Free List에 추가된다.
      * **살아있는 객체 비율 분석** : 각 Old 영역의 살아있는 객체 비율을 분석하고, 이 비율을 기반으로 다음 단계(Evacuation) 에서 **실제로 GC 대상이 될 Old 영역들을 선별하고, CSet 을 구성한다.**
    * 특징 :&#x20;
      * 초기 작업(비어있는 Region 회수 등) 은 짧은 STW 상태에서 이루어진다.&#x20;
      * 이후 남은 작업(Old 영역 분석 및 통계 자료 갱신)은 Concurrnet 하게 이루어진다.&#x20;
  * **6. Evacuation (재배치 - STW, Mixed GC 의 핵심)**
    * **목적** :&#x20;
      * Cleanup에서 선정된 CSet(Collection Set)에 포함된 **GC할 가치가 높은 Old 영역과 현재 Young 영역의 살아있는 객체들을 새로운 빈 영역으로 복사(Evacuation)**&#xD55C;다. 이 과정에서 선택된 Region들의 메모리 조각화가 해소(Compaction)된다.&#x20;
      * 복사가 완료된 CSet 의 Region 들은 통째로 비워져서 Free List 에 반환된다.
    * **특징** :&#x20;
      * 이 단계는 **STW(Stop-The-World)**&#xB85C; 동작하며, G1GC는 Evacuation 과정에서 발생하는 STW 시간이 사용자가 설정한 목표 시간(-XX:MaxGCPauseMillis) 내에 들어오도록 CSet의 크기를 동적으로 조절한다.

## 3. G1GC 의 내부 동작 방식

Oracle 문서의 "Garbage-First Internals"는 G1 가비지 수집기가 어떻게 예측 가능한 일시 정지 시간을 유지하고 대규모 힙을 효율적으로 관리하는지에 대한 핵심적인 내부 메커니즘을 설명한다. 단순히 가비지를 수집하는 것을 넘어, 최적화를 위해 어떤 판단을 내리고 어떤 특별한 상황에 대응하는지 알아볼 수 있다.

### 3.1 Java Heap Sizing (자바 힙 크기 조정)&#x20;

G1GC 는 힙 크기를 관리하는 방식에 있어 몇가지 특징을 가진다.&#x20;

* 동적 조정: G1GC는 힙 크기를 동적으로 조정하여 애플리케이션의 메모리 요구사항에 맞춥니다. 이는 `Xms` (최소 힙 크기)와 `Xmx` (최대 힙 크기) 옵션 내에서 이루어집니다.
* Region 기반: G1GC는 힙을 `Region`이라는 고정 크기(기본 1MB에서 32MB 사이, 힙 크기에 따라 2048개의 Region을 넘지 않게 자동 조정)의 구역으로 나눕니다. 이 Region들은 필요에 따라 Eden, Survivor, Old, Humongous, Free 등 다양한 역할을 합니다.
* Freelist 활용: GC 후 비워진 Region들은 `Freelist`로 반환되어 새로운 객체 할당에 재사용됩니다. 충분한 `Freelist` 공간을 확보하는 것이 G1GC 성능에 중요합니다.

### 3.2 Preiodic Garbage Collections (주기적인 가비지 수집)&#x20;

G1GC는 완전히 주기적으로 작동하는 것은 아니지만, 특정 조건이 충족될 때 GC 사이클을 시작합니다.&#x20;

* Young-only GC: 주로 Eden 영역이 가득 찼을 때 발생합니다. 이는 애플리케이션의 객체 할당 속도에 따라 매우 자주 발생할 수 있습니다.
* Space-reclamation Phase: Old Generation의 힙 점유율이 `IHOP`을 초과할 때 동시 마킹 사이클이 시작되고, \
  이어서 `Mixed GC`가 여러 번 발생하여 Old Generation의 공간을 점진적으로 회수합니다.
* Full GC 회피: G1GC의 목표는 이러한 주기적인 Young GC와 Space-reclamation Phase를 통해 `Full GC`를 가능한 한 피하는 것입니다.

### 3.3 Determining Initiating Heap Occupancy (IHO) - 동시 마킹의 시작점

G1GC가 Old Generation에 가비지가 얼마나 쌓였을 때 청소를 시작해야 할지 결정하는 기준점이 바로 Initiating Heap Occupancy (IHO) 입니다.

* 역할: Old Generation의 힙 사용량이 이 임계값(기본적으로 전체 힙의 45%)을 초과하면, G1GC는 이제 `Space-reclamation Phase` (Old Generation 공간 회수 단계)를 시작해야겠다고 판단합니다. 이 단계의 첫 걸음이 바로 `Initial Mark`입니다.
* 결정 방식: G1GC는 과거 GC 사이클의 데이터(가비지 생성 속도, GC 소요 시간 등)를 분석하여 다음 `Initial Mark`를 시작해야 할 가장 적절한 시점을 예측하고 IHO 값을 동적으로 조정할 수 있습니다. 너무 일찍 시작하면 불필요한 GC 오버헤드가 발생하고, 너무 늦게 시작하면 힙이 고갈되어 `Full GC`로 이어질 수 있기 때문에 시점 예측이 매우 중요합니다.
* 장점: 힙이 완전히 가득 차기 전에 미리 동시 마킹을 시작함으로써, 애플리케이션의 멈춤 없이 가비지 대상을 미리 파악할 수 있어 예측 가능한 일시 정지 시간을 유지하는 데 기여합니다.

### 3.4 Marking (마킹)&#x20;

`Marking`은 G1GC가 살아있는 객체들을 식별하는 과정을 의미하며, `Space-reclamation Phase`의 핵심적인 부분입니다.

* 목표: 힙 내의 모든 살아있는 객체를 정확히 표시하여, 표시되지 않은 객체들을 가비지로 간주하고 회수할 대상을 파악하는 것입니다.
* 주요 단계:
  * Initial Mark (`Concurrent Start`): GC Roots(스택, 정적 필드 등)에서 직접 참조하는 객체를 표시하며 마킹 프로세스를 시작합니다. 이는 짧은 STW를 발생시키며, 보통 Young GC 중에 함께 수행됩니다. 이때 `SATB (Snapshot-At-The-Beginning)` 스냅샷을 찍고 `TAMS (Top-At-Mark-Start)` 포인터를 설정합니다.
  * Root Region Scan: Initial Mark 이후 애플리케이션과 동시에 GC Root에서 연결된 객체들을 스캔합니다.
  * Concurrent Marking: 애플리케이션과 동시에 힙 전체를 탐색하며 살아있는 객체를 표시합니다. 이 과정에서 `RSet`과 `Write Barrier`가 중요한 역할을 하여 객체 참조 변경을 추적합니다.
  * Remark: 동시 마킹이 거의 완료된 후, 잠시 STW를 발생시켜 `SATB` 버퍼에 기록된 변경사항을 처리하고 최종적으로 살아있는 객체들을 확정합니다.
  * Cleanup: Remark 이후 STW를 발생시켜 완전히 비어있는 Region을 회수하고, 가비지 비율이 높은 Old Region들을 `Mixed GC`의 `CSet` 후보로 선정합니다.
* SATB (Snapshot-At-The-Beginning)와 Write Barrier: 동시 마킹 중에 애플리케이션이 객체 참조를 변경할 수 있으므로, G1GC는 `SATB`와 `Write Barrier`를 사용하여 마킹의 정확성을 보장합니다. `Write Barrier`는 참조 변경을 감지하고 `Card Table`과 `RSet`을 업데이트하여 GC가 놓치는 객체가 없도록 합니다.

### 3.5 Behavior in Very Tight Heap Situations (매우 부족한 힙 상황에서의 동작)

G1GC는 대부분의 상황에서 잘 작동하지만, 힙 공간이 극도로 부족해지는 예외적인 상황에 직면할 수 있습니다.

* 문제 발생: 주로 `Evacuation` (살아있는 객체를 다른 Region으로 복사하는 과정) 중에 발생합니다. G1GC가 `Young GC`나 `Mixed GC`를 수행하면서 살아있는 객체를 복사해야 하는데, 힙 내에 충분한 빈 Region 공간이 없어 더 이상 복사할 수 없을 때 이 상황 (`Evacuation Failure`)이 발생합니다.
* 결과: 이러한 `Evacuation Failure`는 매우 심각한 문제입니다. G1GC는 이를 해결하기 위해 최후의 수단으로 Full GC를 수행합니다. Full GC는 힙 전체를 단일 스레드로 정리하는 `Serial Old GC`와 유사하게 작동하며, 매우 긴 STW를 발생시켜 애플리케이션 성능에 치명적인 영향을 줄 수 있습니다.
* 의미: G1GC는 `Full GC`를 피하기 위해 설계되었지만, 예측 가능한 GC를 보장하기 위해 한계에 다다르면 `Full GC`로 전환하여 프로그램의 메모리 부족을 해결하려는 방어적인 메커니즘을 가지고 있습니다. 이는 G1GC 튜닝 시 `Full GC`가 발생하지 않도록 힙 크기나 기타 옵션을 신중하게 설정해야 하는 이유를 보여줍니다.

### 3.6 Humongous Objects (초대형 객체)

일반적인 객체는 Region 내의 작은 공간에 할당되지만, G1GC는 특별히 큰 객체인 Humongous Objects를 다루는 별도의 규칙을 가지고 있습니다.

* 정의: \*\*`Humongous Objects`\*\*는 단일 Region 크기의 절반 이상인 객체를 의미합니다. 예를 들어, Region 크기가 4MB라면 2MB 이상의 객체는 Humongous Objects로 분류됩니다.
* 할당 방식: 일반 객체와 달리 `Eden`이나 `Survivor` 영역에 할당되지 않고, Old Generation의 연속적인 Region들에 직접 할당됩니다. 이는 큰 객체를 위한 충분한 연속 공간을 마련하기 위함입니다. \* 가비지 수집: `Humongous Objects`는 일반적인 Young GC의 대상이 아닙니다.
  * 주로 `Cleanup` 일시 정지(pause) 중에, 마킹 단계에서 가비지로 식별된 `Humongous Objects`가 포함된 Region들이 한 번에 회수됩니다.
  * 또는 `Full GC`가 발생할 때 함께 회수될 수 있습니다.
  * G1은 특정 원시 타입 배열 (primitive type arrays) 에 대해서는 `Concurrent Marking` 중에도 기회적으로 회수할 수 있는 최적화가 적용되어 있습니다.
* GC 트리거: `Humongous Objects`의 할당은 때때로 \*\*`Initial Mark young collection`\*\*을 조기에 트리거할 수 있습니다. 이는 큰 객체 할당으로 인해 Old Generation의 힙 점유율이 급격히 증가하여 IHOP 임계값을 초과하기 때문입니다. 짧은 시간 내에 매우 큰 객체를 많이 생성하는 애플리케이션에서는 이러한 `Humongous Objects` 할당이 GC 빈도와 성능에 영향을 줄 수 있습니다.

## 4. Ergonomic Defaults for G1 GC (G1 GC의 기본 설정)

G1GC는 개발자가 복잡한 튜닝 없이도 좋은 성능을 얻을 수 있도록 "인체공학적인" 기본 설정값들을 제공합니다.

* `MaxGCPauseMillis` (기본 200ms):
  * 이것은 G1GC가 한 번의 STW 일시 정지 동안 애플리케이션을 멈추게 할 수 있는 최대 시간을 밀리초 단위로 설정하는 목표값입니다.
  * G1GC는 이 목표를 달성하기 위해 `CSet`의 크기(한 번에 처리할 Region의 수)를 동적으로 조절합니다. 예를 들어, 200ms 안에 처리하기 어렵다고 판단되면, CSet에 포함될 Region의 수를 줄여서 여러 번의 짧은 GC를 수행합니다.
* `ParallelGCThreads` (기본값은 CPU 코어 수에 따라 다름):
  * GC 작업을 수행할 때 사용할 병렬 스레드의 수를 결정합니다. Initial Mark, Remark, Cleanup, 그리고 Young/Mixed GC의 Evacuation 등 STW가 발생하는 단계에서 이 스레드들이 병렬로 작업을 수행하여 시간을 단축합니다.
  * `Concurrent Marking`과 같이 STW가 없는 동시 작업에는 `ConcGCThreads`라는 별도의 스레드 풀을 사용합니다.
* 자동 조정 (Ergonomics): G1GC는 단순히 이 기본값들을 적용하는 것을 넘어, 실행 시간 동안 애플리케이션의 행동과 힙 사용량을 모니터링하여 이러한 설정들을 동적으로 조정합니다. 예를 들어, GC 일시 정지 시간이 목표를 초과하면 다음 GC에서는 CSet 크기를 줄이는 식입니다.

이러한 "인체공학적" 기본 설정과 자동 조정 능력 덕분에, 대부분의 경우 G1GC는 별다른 튜닝 없이도 합리적인 성능을 제공할 수 있습니다. 그러나 특정 애플리케이션의 요구 사항을 충족하기 위해서는 이러한 옵션들을 이해하고 필요에 따라 수동으로 튜닝하는 것이 중요할 수 있습니다.

