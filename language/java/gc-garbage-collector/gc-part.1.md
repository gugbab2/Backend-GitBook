# GC Part.1

## 1. GC 과정 - Generational Garbage Collection

### 1.1 'stop-the-world' 란?

* GC 를 실행하기 위해서 JVM 이 애플리케이션 실행을 멈추는 것을 의미한다.&#x20;
* stop-the-world 가 발생하면 GC 를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. \
  \-> GC 작업이 완료된 이후 중단했던 작업을 다시 시작한다.&#x20;
* 어떤 알고리즘을 사용해도 stop-the-world 는 발생하고, GC 튜닝이란 stop-the world 시간을 줄이는 것이다.

### 1.2. GC 란?

* Java 에서는 개발자가 프로그램 코드로 메모리를 명시적으로 해제하지 않기 때문에, GC 가 더 이상 필요 없는 객체를 찾아 지우는 작업을 한다. \
  \-> heap 영역에 해당!\
  \-> JVM 이 담당하는 작업으로 메모리 해제 타이밍을 정확하게 알지 못한다.&#x20;
* GC 는 다음 두 가설(weak generational hypothesis)에 의해서 만들어졌다.&#x20;
  * **대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.**\
    **-> 대부분의 객체가 일시적으로 사용되고 GC 에 대상이 된다는 관찰 결과에 따른 내용**
  * **오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.**
* **결국 GC 도 비용이 드는 작업인데, 메모리 전체 부분이 아닌 특정 부분만을 탐색(Mark)하여 해제(Sweep)해야    효율적이다.** \
  **-> 어차피 대다수의 객체가 금방 사라지기에 Young Generation 안에서 최대한 메모리를 해제하도록 설계**

### 1.3 GC 흐름

* 이러한 가설의 장점을 최대한 잘 살리기 위해서 **HotSpot VM(JVM 벤더 중 하나로, 성능이 높은 벤더**) 에서는 **Heap 영역**을 크게 2개로 물리적 공간을 나누었다.&#x20;
  * **Young 영역** : 새롭게 생성한 객체 대부분이 위치하는 공간으로 대부분의 객체가 금방 사용하지 않는 상태가 되기 때문에, 매우 많은 객체가 Young 영역에 생성되었다가 사라진다. \
    \-> 해당 영역에서 일어나는 GC 를 Minor GC 가 발생한다고 말한다.&#x20;
  * **Old 영역** : Young 영역에서 살아남은(객체를 계속 사용하는 상태) 객체가 복사되는 공간이다. 대부분 Young 영역보다 크게 할당하고, 크기가 큰 만큼 Young 영역보다 GC 는 적게 발생한다. \
    \-> 해당 영역에서 일어나는 GC 를 Major GC(혹은 Full GC) 가 일어난다 말한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (1) (1) (2).png" alt=""><figcaption></figcaption></figure>

* 위 그림의 Permanent Generation(자바8 부터 metaspace 라고  부른다) 영역은 Method Area(클래스 수준의 정보를 저장하는 영역) 이라고 한다.
* Permanent Generation 영역은 Old 영역에서살아남은 객체가 영원히 남아있는 곳은 아니다. \
  이 영역에서도 GC 가 발생할 수 있는데, 여기서 GC 가 발생해도 Major GC 횟수에 포함된다.&#x20;
* "Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우가 있을 때 어떻게 처리가 될까?" 라는 \
  상황을 처리하기 위해서 Old 영역에는 512 바이트에 덩어리(chunk) 로 되어 있는 카드 테이블(card table) 이 존재한다.&#x20;
* 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 Young 영역 객체에 해당하는 카드테이블이 체크된다. Young 영역의 GC 를 실행할 때는 Old 영역에 있는 모든 객체의 참조를 확인하지 않고 이 카드 테이블만 뒤져서 체크 되어있을 경우, GC 대상에서 제외한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## 2. Young 영역에 대한 GC

### 2.1 Young 영역의 구분

* Young 영역은 다음 3개의 영역으로 나뉜다.
  * Eden 영역
  * Survivor 영역

### 2.2 Young 영역의 동작

* 각 영역의 처리 절차 순서에 따라서 기술하면 다음과 같다.
  * 새로 생성한 대부분의 객체는 Eden 영역에 위치한다.
  * Eden 영역에서 GC 가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다. \
    \-> **Eden 영역이 가득 차야만 minor GC 가 실행된다.**
  * Eden 영역에서 GC 가 발생하면 이미 살아남은 객체가 존재하는 Survivor 영역에 객체가 계속해서 쌓인다.&#x20;
  * 하나의 Survivor 영역이 가득 차게 되면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동한다. 그리고 가득 찬 Survivor 영역은 아무 데이터도 없는 상태로 된다.&#x20;
  * 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 Old 영역으로 이동하게 된다.
* **Young 영역의 동작에서 하나의 Survivor 영역은 반드시 비워져 있어야 한다.** \
  \-> 만약 비워져 있지 않는다면, 비정상적인 상황으로 인지하면 된다.&#x20;

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

## 3. Old 영역에 대한 GC

* 시간이 아주 많이 지나서, 언젠가 Old Generation 도 다 채워지는 날이 올때, major gc 가 발생하면서 Mark And Sweep 방식을 통해서 필요 없는 메모리를 비우는데, minor gc 에 비해서 시간이 오래 걸린다.&#x20;
* 또한, GC 가 실행될 때마다 stop-the-world 현상이 발생하는데, 이때 minor gc 보다 major gc 가 stop-the-world 현상이 더 길다.&#x20;

## 4. GC 에서 사용하는 알고리즘&#x20;

### 4.1 Reference Counting(자바에서 사용되지 않는다...)

<figure><img src="../../../.gitbook/assets/image (1).png" alt="" width="375"><figcaption></figcaption></figure>

* **Root Space** 는 Heap 영역 참조를 가리키고 있는 공간이다.\
  **-> Stack 의 로컬변수** \
  **-> Method Area 의 static 변수**\
  **-> Native Method Stack 의 JNI 참조**
* Reference Counting 은 힙 영역의 객체들이 각각 reference count 라는 숫자를 가지고 있다고 생각하면 좋은데, **여기서 reference count 는 몇가지 방법으로 해당 객체에 접근할 수 있는지를 의미한다.**&#x20;
* 만약 reference count 가 0 에 다다르면 해당 객체에 접근할 수 있는 방법이 없다는 뜻이므로 메모리 해제의 대상이 되는 것이다.&#x20;
* 하지만 Reference Counting 은 순환 참조 문제가 발생할 수 있다. \
  \-> Root Space 가 모든 Heap Space 의 참조를 끊는다면, 순환 참조가 발생하게 되고, 결국 사용하지 않는 메모리 영역이 해제되지 못하고 메모리 누수가 발생하게 된다.&#x20;

### 4.2 Mark And Sweep(자바에서 기본적으로 사용되는 방식)

<figure><img src="../../../.gitbook/assets/스크린샷 2023-06-08 21.28.48.png" alt="" width="563"><figcaption></figcaption></figure>

* Mark And Sweep 알고리즘은 Root Space 부터 해당 객체에 접근이 가능한지, 아닌지를 메모리 해제의 기준으로 삼는다.&#x20;
* **Root Space 부터 그래프 순회를 통해서 연결된 객체를 찾아내고(Mark) 연결이 끊어진 객체는 지운다(Sweep).**
* 그림에 분산되어 있던 메모리가 이쁘게 정리된 것을 볼 수 있는데, **이 것은 메모리 파편화를 방지하는 Compaction 방식이다.** \
  \-> **Mark And Sweep 알고리즘에서 Compaction 이 필수는 아니다.**&#x20;
* Mark And Sweep 방식을 사용하면, Root Space 부터 연결이 끊긴 순환 참조되는 객체들도 지울 수 있다.&#x20;
* 하지만, **reference count 가 0이 되면 지워버리는 Reference Counting 방식과 달리, Mark And Sweep 방식은 의도적으로 특정 순간에 GC 를 실행해야 한다.** \
  **-> 즉 어느 순간에, 실행 중인 애플리케이션이 GC 에게 컴퓨터 리소스를 내어 주어야 한다는 것이다.** \
  **-> **_**애플리케이션과 GC 실행이 병행된다.**_&#x20;
