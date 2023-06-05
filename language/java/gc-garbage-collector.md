# GC(Garbage Collector)

## GC 과정 - Generational Garbage Collection

### 'stop-the-world' 란?

* GC 를 실행하기 위해서 JVM 이 애플리케이션 실행을 멈추는 것을 의미한다.&#x20;
* stop-the-world 가 발생하면 GC 를 실행하는 쓰레드를 제외한 나머지 쓰레드는 모두 작업을 멈춘다. \
  \-> GC 작업이 완료된 이후 중단했던 작업을 다시 시작한다.&#x20;
* 어떤 알고리즘을 사용해도 stop-the-world 는 발생하고, GC 튜닝이란 stop-the world 시간을 줄이는 것이다.

### GC 주의사항&#x20;

* Java 는 프로그램 코드에서 메모리를 명시적으로 지정하여 해제하지 않는다.\
  \-> 가끔 명시적으로 해제하려고 해당 객체를 null 로 지정하거나, System.gc() 메서드를 호출하는 개발자가 있다. null 로 지정하는 것은 문제가 안되지만 **System.gc() 메서드는 시스템성능에 매우 큰 영향을 끼치기에 절대로 호출해서는 안된다.**

### GC 흐름

* Java 에서는 개발자가 프로그램 코드로 메모리를 명시적으로 해제하지 않기 때문에, GC 가 더 이상 필요 없는 객체를 찾아 지우는 작업을 한다. \
  \-> heap 영역에 해당!
* GC 는 다음 두 가설(weak generational hypothesis)에 의해서 만들어졌다.&#x20;
  * 대부분의 객체는 금방 접근 불가능 상태(unreachable)가 된다.\
    \-> 대부분의 객체가 일시적으로 사용되고 GC 에 대상이 된다는 관찰 결과에 따른 내용
  * 오래된 객체에서 젊은 객체로의 참조는 아주 적게 존재한다.
* 이러한 가설의 장점을 최대한 잘 살리기 위해서 HotSpot VM(JVM 벤더 중 하나로, 성능이 높은 벤더) 에서는 크게 2개로 물리적 공간을 나누었다.&#x20;
  * Young 영역 : 새롭게 생성한 객체 대부분이 위치하는 공간으로 대부분의 객체가 금방 사용하지 않는 상태가 되기 때문에, 매우 많은 객체가 Young 영역에 생성되었다가 사라진다. \
    \-> 해당 영역에서 일어나는 GC 를 Minor GC 가 발생한다고 말한다.&#x20;
  * Old 영역 : Young 영역에서 살아남은(객체를 계속 사용하는 상태) 객체가 복사되는 공간이다. 대부분 Young 영역보다 크게 할당하고, 크기가 큰 만큼 Young 영역보다 GC 는 적게 발생한다. \
    \-> 해당 영역에서 일어나는 GC 를 Major GC(혹은 Full GC) 가 일어난다 말한다.&#x20;

<figure><img src="../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* 위 그림의 Permanent Generation(자바8 부터 metaspace 라고  부른다) 영역은 Method Area(클래스 수준의 정보를 저장하는 영역) 이라고 한다.
* Old 영역은 살아남은 객체가 영원히 남아있는 곳은 아니다. 이 영역에서도 GC 가 발생할 수 있는데, 여기서 GC 가 발생해도 Major GC 횟수에 포함된다.&#x20;
* "Old 영역에 있는 객체가 Young 영역의 객체를 참조하는 경우가 있을 때 어떻게 처리가 될까?" 라는 상황을 처리하기 위해서 Old 영역에는 512 바이트에 덩어리(chunk) 로 되어 있는 카드 테이블(card table) 이 존재한다.&#x20;
* 카드 테이블에는 Old 영역에 있는 객체가 Young 영역의 객체를 참조할 때마다 Young 영역 객체에 해당하는 카드테이블이 체크된다. Young 영역의 GC 를 실행할 때는 Old 영역에 있는 모든 객체의 참조를 확인하지 않고 이 카드 테이블만 뒤져서 체크 되어있을 경우, GC 대상에서 제외한다.&#x20;

<figure><img src="../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

## Young 영역

### Young 영역의 구분

* Young 영역은 다음 3개의 영역으로 나뉜다.
  * Eden 영역
  * Survivor 영역

### Young 영역의 동작

* 각 영역의 처리 절차 순서에 따라서 기술하면 다음과 같다.
  * 새로 생성한 대부분의 객체는 Eden 영역에 위치한다.
  * Eden 영역에서 GC 가 한 번 발생한 후 살아남은 객체는 Survivor 영역 중 하나로 이동된다.&#x20;
  * Eden 영역에서 GC 가 발생하면 이미 살아남은 객체가 존재하는 Survivor 영역에 객체가 계속해서 쌓인다.&#x20;
  * 하나의 Survivor 영역이 가득 차게 되면 그 중에서 살아남은 객체를 다른 Survivor 영역으로 이동한다. 그리고 가득 찬 Survivor 영역은 아무 데이터도 없는 상태로 된다.&#x20;
  * 이 과정을 반복하다가 계속해서 살아남아 있는 객체는 Old 영역으로 이동하게 된다.
*

## Old 영역
