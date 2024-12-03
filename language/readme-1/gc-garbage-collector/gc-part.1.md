# GC Part.1

> #### 참고 링크&#x20;
>
> [https://d2.naver.com/helloworld/1329](https://d2.naver.com/helloworld/1329)\
> [https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98GC-%EB%8F%99%EC%9E%91-%EC%9B%90%EB%A6%AC-%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98-%F0%9F%92%AF-%EC%B4%9D%EC%A0%95%EB%A6%AC)

## 1. GC 과정 - Garbage Collection(HotSpot JVM 기반)

-> **HotSpot JVM 은 오라클에서 개발한 JVM 구현체 중 하나이다.**&#x20;

### 1.1 'stop-the-world' 란?

* GC 를 실행하기 위해서 JVM 이 애플리케이션 실행을 멈추는 것을 의미한다.
* stop-the-world 가 발생하면 GC 를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다.\
  -> GC 작업이 완료된 이후 중단했던 작업을 다시 시작한다.
* 어떤 알고리즘을 사용해도 stop-the-world 는 발생하고, GC 튜닝이란 stop-the world 시간을 줄이는 것이다.

### 1.2. GC 란?

* Java 에서는 개발자가 프로그램 코드로 메모리를 명시적으로 해제하지 않기 때문에, GC 가 더 이상 필요 없는 객체를 찾아 지우는 작업을 한다.\
  &#xNAN;**-> heap 영역에 해당!**\
  &#xNAN;**-> JVM 이 담당하는 작업으로 메모리 해제 타이밍을 정확하게 알지 못한다.**
* GC 는 다음 두 가설에 의해서 만들어졌다.
  * **대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.**\
    &#xNAN;**-> 대부분의 객체가 일시적으로 사용되고 GC 에 대상이 된다는 관찰 결과에 따른 내용**
  * **오래된 객체에서 젊은 객체로의 참조는 아주 드물게 존재한다.**
* **결국 GC 도 비용이 드는 작업인데, 메모리 전체 부분이 아닌 특정 부분만을 탐색(Mark)하여 해제(Sweep)해야 효율적이다.**

### 1.3 GC 흐름

* 이러한 가설의 장점을 최대한 잘 살리기 위해서 **HotSpot VM(JVM 구현체 중 하나**) 에서는 **Heap 영역**을 크게 2개로 물리적 공간을 나누었다.
  * **Young 영역** : **새롭게 생성한 객체 대부분이 위치하는 공간으로 대부분의 객체가 금방 사용하지 않는 상태가 되기 때문에, 매우 많은 객체가 Young 영역에 생성되었다가 사라진다.**\
    -> 해당 영역에서 일어나는 GC 를 Minor GC 가 발생한다고 말한다.
  * **Old 영역 : Young 영역에서 살아남은(객체를 계속 사용하는 상태) 객체가 복사되는 공간이다.** \
    -> 대부분 Young 영역보다 크게 할당하고, 크기가 큰 만큼 Young 영역보다 GC 는 적게 발생한다.\
    -> 해당 영역에서 일어나는 GC 를 Major GC가 일어난다 말한다.

<figure><img src="../../../.gitbook/assets/image (46).png" alt=""><figcaption></figcaption></figure>

* 위 그림의 Permanent Generation(자바8 부터 metaspace 라고 부른다) 영역은 Method Area(클래스 수준의 정보를 저장하는 영역) 이라고 한다.
* Permanent Generation 영역은 Old 영역에서살아남은 객체가 영원히 남아있는 곳은 아니다.\
  이 영역에서도 GC 가 발생할 수 있는데, 여기서 GC 가 발생해도 Major GC 횟수에 포함된다.
* "Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우가 있을 때 어떻게 처리가 될까?" 라는\
  상황을 처리하기 위해서 Old 영역에는 512 바이트에 덩어리(chunk) 로 되어 있는 카드 테이블(card table) 이 존재한다.
* 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 Young 영역 객체에 해당하는 카드테이블이 체크된다. Young 영역의 GC 를 실행할 때는 Old 영역에 있는 모든 객체의 참조를 확인하지 않고 이 카드 테이블만 뒤져서 체크 되어있을 경우, GC 대상에서 제외한다.

<figure><img src="../../../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

## 2. Young 영역에 대한 GC

### 2.1 Young 영역의 구분

* Young 영역은 다음 3개의 영역으로 나뉜다.
  * Eden 영역
  * Survivor 영역

### 2.2 Young 영역의 동작

* 각 영역의 처리 절차 순서에 따라서 기술하면 다음과 같다.
  * 새로 생성한 대부분의 객체는 Eden 영역에 위치한다.
  * Eden 영역에서 GC 가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다.\
    -> **Eden 영역이 가득 차야만 minor GC 가 실행된다.**
  * Eden 영역에서 GC 가 발생하면 이미 살아남은 객체가 존재하는 Survivor 영역에 객체가 계속해서 쌓인다.
  * 하나의 Survivor 영역이 가득 차게 되면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동한다. 그리고 가득 찬 Survivor 영역은 아무 데이터도 없는 상태로 된다.
  * 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 Old 영역으로 이동하게 된다.
* **Young 영역의 동작에서 하나의 Survivor 영역은 반드시 비워져 있어야 한다.**\
  -> 만약 비워져 있지 않는다면, 비정상적인 상황으로 인지하면 된다.

<figure><img src="../../../.gitbook/assets/image (66).png" alt=""><figcaption></figcaption></figure>

> ### HotSpot VM 에서 빠른 메모리 할당을 위해서 사용하는 두가지 기술&#x20;
>
> #### 1. bump-the-pointer
>
> * Eden 영역에 할당된 마지막 개체를 추적한다. 마지막 객체는 Eden 영역의 맨 위에 있다. 그리고 다음에 생성되는 개체가 있으면, 해당 개체의 크기가 Eden 영역에 넣기 적당하지만 확인한다.&#x20;
> * 만약 해당 객체의 크기가 적당하다고 판정되면 Eden 영역에 넣게되고, 새로 생성된 객체가 맨 위에 있게된다.&#x20;
> * 따라서 새로운 객체를 생성할 때 마지막에 추가된 객체만 점검하면 되므로 매우 빠르게 메모리 할당이 이루어진다.&#x20;
>
> #### 2. TLABs
>
> * bump-the-pointer 는 멀티 스레드 환경에서는 문제가 발생한다.&#x20;
> * Thread-Safe 하기 위해서 만약 여러 스레드에서 사용하는 객체를 Eden 영역에 저장하려면 Lock 이 발생할 수 밖에 없고, lock-contention 때문에 성능은 매우 떨어지게 될 것이다.&#x20;
> * HotSpot VM 에서 이를 해결한 것이 TLABs 이다.&#x20;
> * 각각의 스레드가 각각의 몫에 해당하는 Eden 영역의 작은 덩어리를 가질 수 있도록 하는 것이다. 각 쓰레드에서 자기가 갖고 있는 TLAB 에만 접근할 수 있기 때문에, bump-the-pointer 라는 기술을 사용하더라도 아무런 Lock 없이 메모리 할당이 가능하다.
