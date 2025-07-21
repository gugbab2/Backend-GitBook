# G1GC

#### 참고 링크&#x20;

{% embed url="https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html" %}

## 1. Garbage-First(G1) 가비지 컬렉터 소개 <a href="#jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42" id="jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42"></a>

G1(Garbage-First) 가비지 컬렉터는 **"다중 프로세서 시스템"** & **"대용량 메모리 환경"**&#xC744; 위해 설계되었다. 목표는 높은 처리량을 유지하면서도 STW 를 예측 가능하게, 그리고 짧게 가져가는 것이다. 이를 위해 G1 은 최소한의 설정으로 최적의 성능을 제공하려 한다.&#x20;

G1 이 특히 목표로 하는 환경 및 애플리케이션의 특징은 다음과 같다.&#x20;

* 힙의 크기가 수십 GB 이상으로 매우 크고, 사용중인 데이터가 Java 힙의 50% 이상을 차지하는 경우 (대용량 메모리)&#x20;
* 객체 할당 및 승격 속도가 시간에 따라 크게 변동할 수 있는 경우
* 힙 내부에 파편화가 상당한 경우&#x20;
* 예측 가능한 STW 를 통해 긴 STW 를 피해야 하는 경우&#x20;

G1 은 애플리케이션이 실행되는 **동시에 일부 작업을 수행**하여 짧은 STW 를 달성한다. 이는 애플리케이션에 할당될 수 있는 프로세서 자원을 가비지 컬렉션에 일부 할애하는 방식이다. **결과적으로 G1 은 다른 처리량 중심 GC 보다 STW 가 훨씬 짧지만, 애플리케이션 전체 처리량은 낮아지는 경향이 있다.**&#x20;

JDK 9 부터 G1 은 Default GC 이다.&#x20;

## 2. 기본 개념&#x20;

G1 은 각 STW 시 목표 중지 시간을 모니터링 한다.&#x20;

다른 컬렉터와 마찬가지로 G1 은 힙을 신세대(에덴, 서바이버1,2)와 구세대로 나눈다. 공간 회수 작업은 가장 효율적인 신세대에 집중되며, 구세대에서도 간헐적으로 공간 회수가 수행된다.&#x20;

일부 작업은 처리량 향상을 위해서 STW 에서 수행한다. 자원을 많이 사용하는 작업에 대해서는 애플리케이션과 병렬적으로 진행한다. 공간 회수를 위한 STW 을 짧게 하기 위해서 G1 은 공간 회수를 단계적으로 병렬로 수행한다.&#x20;

G1 은 이전 애플리케이션의 동작 및 STW 에 대한 정보를 추적하여 관리 비용 모델을 구축함으로 예측 가능성을 확보한다. 이 정보를 사용해 STW 시간 목표를 달성하기 위한 작업의 크기를 조정한다.&#x20;

### 2.1 힙 레이아웃&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-07-20 22.38.37.png" alt="" width="563"><figcaption></figcaption></figure>

G1은 힙을 크기가 동일한 여러 개의 힙 영역으로 분할한다. 각 영역은 연속적인 가상 메모리 범위이며, 할당 및 회수의 단위이다. 특정 시점에 이러한 각 영역은 비어 있거나 특정 세대(신세대, 구세대) 에 할당될 수 있다.&#x20;

애플리케이션은 기본적으로 항상 신세대에 객체를 할당한다. 거대(Humongous) 객체의 경우 구세대에 직접 할당한다.&#x20;

### 2.2 가비지 수거 주기

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-07-21 09.26.18.png" alt="" width="563"><figcaption></figcaption></figure>

> ### 용어 정리&#x20;
>
> * Freelist: GC가 끝나고 비워진 Region들이 모여있는 목록. 새로운 객체가 필요할 때 Freelist에서 비어있는 Region을 가져와 사용
> * IHOP (Initiating Heap Occupancy Percent): Old Generation의 힙 사용량이 이 백분율(기본 45%)을 넘으면, G1GC가 Old Generation을 정리하기 위한 동시 마킹 사이클을 시작해야겠다고 판단하는 기준점.
> * SATB (Snapshot-At-The-Beginning): '시작 시점의 스냅샷'이라는 뜻. 동시 마킹이 시작될 때 힙의 살아있는 객체 상태를 논리적으로 기록.&#x20;
>   * Initial Mark 시점에 GC 루트로부터 도달 가능한 모든 살아있는 객체를 대상으로 스냅샷
> * TAMS (Top-At-Mark-Start) 포인터: 각 Region에 설정되는 포인터. 마킹 시작 이후 새로 할당된 객체와 기존 객체를 구분.
> * RSet (Remembered Set): 각 Region마다 RSet이 있어서, 다른 Region에 있는 객체가 현재 Region의 객체를 참조하는 정보를 기록.
> * Card Table (카드 테이블): RSet을 효율적으로 업데이트하기 위한 자료구조. 힙을 작은 '카드'들로 나누고 각 카드의 상태를 기록.
> * Write Barrier (쓰기 장벽): 애플리케이션이 객체 참조를 수정할 때마다 자동으로 실행되는 작은 코드 조각. \
>   참조 변경 시 Card Table에 표시하고 RSet을 업데이트.
> * CSet (Collection Set): GC가 특정 GC 사이클에서 실제로 가비지 수거 작업을 수행할 대상 Region들의 집합.

#### 2.2.1 Young-only Phase (상단 반원)&#x20;

이 그림의 상단 반원은 신세대(Eden 및 Survivor Region) 을 청소하는 단계를 나타낸다. 이 단계는 항상 STW 가 발생한다.&#x20;

* **일반적인 Young GC (작은 파란색 원들)**&#x20;
  * **트리거** : 애플리케이션이 객체를 만들다 신세대(Eden 및 Survivor Region)가 새로운 객체를 할당할 공간이 부족할 때 시작된다.&#x20;
  * **STW 여부 : O**&#x20;
  * **주요 동작**&#x20;
    * 현재 Eden, Survivor Region 의 살아있는 객체들을 찾아 다른 비어있는 Region 으로 복사한다.&#x20;
    * 복사 후 비워진 Region 들은 Freelist 에 추가되어 재활용된다.&#x20;
* **Concurrent Start (큰 파란색 원 - Initial Mark)**&#x20;
  * **트리거** : "Old Gen Occupancy exceeds threshold" (Old Generation 힙 사용량이 IHOP 임계값 초과) 라는 조건이 충족되면, 이 **Young-only GC 의 STW 구간을 활용해서 Initial Mark 단계가 함께 수행된다.**&#x20;
  * **STW 여부 : O**
  * **주요 동작**&#x20;
    * GC 루트 스캔 : GC 루트에서 직접 참조하는 객체들을 표시한다.&#x20;
    * SATB(Snapshot-At-The-Beginning) 스냅샷 : 이 시점에서 힙의 논리적인 '스냅샷' 을 찍어 마킹의 기준으로 삼는다.&#x20;
    * TAMS(Top-At-Mark-Start) 포인터 설정 : 각 Region 에 TAMS 포인터를 설정한다. 이 포인터 이후에 새로 할당된 객체들은 다음 GC 사이클에서 처리될 대상으로 간주된다.&#x20;
  * 이 단계는 Old Generation 가비지 수거를 위한 Space-reclamation Phase의 첫 걸음이다.&#x20;
* **Remark (주황색 원)**&#x20;
  * **트리거** : Concurrent Start 이후 동시 마킹(Cuncurrent Mark) 이 진행된 후, 그림에 나타난 것처럼 Young-only Phase 흐름 안에서 Remark 단계가 수행될 수 있다. 이는 STW 가 발생하는 단계이기 때문이다. \
    (주로 Young-only GC 의 STW 구간에 편승하여 실행된다)&#x20;
  * **STW 여부 : O**&#x20;
  * **주요 동작**&#x20;
    * 동시 마킹 후 SATB 버퍼에 기록된 내용과 비교해, 혹시 놓쳤을지도 모르는 살아있는 객체들을 찾아내서 최종적으로 마킹한다.&#x20;
* **Cleanup (주황색 원)**&#x20;
  * **트리거** : Remark 이후, 그림에 나타난 것처럼 Young-only Phase 흐름 안에서 Cleanup 단계가 수행될 수 있다. 이는 STW 가 발생하는 단계이다. \
    (주로 Young-only GC 의 STW 구간에 편승하여 실행된다)
  * **STW 여부 : O**&#x20;
  * **주요 동작**&#x20;
    * 마킹 결과에 따라 완전히 비어있는 Region 들을 찾아내 즉시 힙 공간으로 돌려준다.&#x20;
    * 이 Region 들은 Freelist 에 추가되어 재활용된다.&#x20;
    * 또한, 가비지 비율이 높은 Old Region 들을 뒤에서 나올 Mixed GC 에서 처리할 목록에 추가한다.&#x20;

#### 2.2.2 Space-reclamation Phase (하단 반원)&#x20;

이 그림의 하단 반원은 Old Generation 에서 사용되지 않는 공간(가비지) 를 회수하는 단계를 나타낸다. 이 단계의 핵심은 대부분의 작업을 STW 없이 애플리케이션과 "동시에(concurrently)" 진행해서 멈춤 시간을 최소화하는 것이다.&#x20;

* **Concurrent Marking (동시 마킹) (하단 반원의 검은색 선을 따라 흐르는 부분)**
  * **트리거** : Concurrent Start (Initial Mark) 이후부터 Remark 전까지의 긴 구간 동안 지속 된다. 이 구간에는 Root Region Scan(GC Root 에서 연결된 객체 탐색) 작업도 포함되어 진행된다.&#x20;
  * **STW 여부 : X**
  * **주요 동작**&#x20;
    * GC 스레드가 힙 전체를 돌아다니며 살아있는 객체를 모두 표시한다.&#x20;
    * 이 과정에서 RSet(Remembered Set) 과 쓰기 장벽(Write Barrier) 이 중요한 역할을 한다.&#x20;
    * 쓰기 장벽이 객체 참조 변경을 감지하면 Card Table 에 표시하고, 이를 바탕으로 RSet 이 업데이트된다.&#x20;
* **Mixed GC (복합 GC) (자주색 작은 원들)**&#x20;
  * **트리거** : Cleanup 단계 이후, Old Generation 여러 차례 Mixed GC 가 발생할 수 있다.
  * **STW 여부 : O**&#x20;
  * **주요 동작**&#x20;
    * CSet (Collection Set) 구성: GC가 실제로 청소할 대상 Region들의 집합인 CSet을 구성한다. 이 CSet에는 항상 현재 Young Generation Region(Eden과 Survivor)이 포함되고, 여기에 Old Generation Region 중에서 동시 마킹 단계에서 가장 가비지가 많다고 식별된 Region들이 추가돼요. G1GC는 설정된 GC 멈춤 시간 목표를 지키면서 가장 효율적으로 공간을 회수할 수 있는 Old Region들을 우선적으로 선택해서 CSet에 포함시켜요.
    * 객체 복사 (Evacuation): CSet에 포함된 모든 Region들 내의 살아있는 객체들을 힙의 다른 비어있는 Region으로 복사(Evacuation)하여 공간을 회수하고 메모리를 압축합니다.
    * Region 재활용: 객체 복사가 끝나면, 원래의 Region들은 완전히 비워져서 다음 객체 할당에 바로 사용될 수 있게 됩니다. 이 비워진 Region들은 다시 Freelist로 돌아가 재활용돼요.
  * 중요한 점 : 이 Mixed GC는 한 번에 길게 멈추는 게 아니라, 설정된 멈춤 시간 목표를 넘기지 않도록 여러 번의 짧은 STW로 나뉘어서 실행될 수 있어요.

#### 2.2.3 예외적인 경우 Full GC

* 트리거 : G1GC가 위 사이클을 통해 충분히 공간을 확보하지 못할 때(예: 너무 빠른 객체 할당 속도), \
  예외적으로 Full GC가 발생할 수 있다.&#x20;
* STW : O, 이때는 힙 전체를 대상으로 매우 길\~게 STW가 발생한다.&#x20;
* 왜 위험한가? : Full GC는 애플리케이션을 오랫동안 멈추기 때문에, 사용자 경험에 치명적인 영향을 줄 수 있다. \
  &#x20;G1GC 튜닝의 가장 큰 목표는 Full GC를 피하는 것이다.&#x20;

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

