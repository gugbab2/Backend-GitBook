# GC(Garbage Collector)

* GC(Garbage Collection)는 자바 애플리케이션에서 사용하지 않는 메모리를 자동으로 수거하는 기능을 말한다.
* `C/C++` 같은 언어는 메모리를 할당하고 직접 해제해야했지만, 자바에서는 `GC`를 이용하여 개발자들이 메모리 관리를 비교적 신경쓰지 않아도 된다.

## 1. JVM 메모리 구조

* 자바에서는 크게 두 영역으로 메모리 영역을 구분한다.
  * young 영역 : 생성된지 얼마 안된 객체를 저장\
    \-> 이 영역에 저장된 객체들이 시간이 지나게 된다면 우선순위가 낮아지면 old 영역으로 옮겨간다.&#x20;
  * old 영역 : 생성된지 오래된 객체를 저장
  * perm 영역 : 클래스, 메소드 등의 코드가 저장되는 영역

## 2. JVM 메모리 관리 방식

### 2-1. Minor GC&#x20;

* 먼저 `New/Young` 영역을 `Minor GC` 라고 부른다. `New/Young` 영역은 `Eden / Survivor` 이라는 두 영역으로 또 나뉘게 된다.
* `Eden` 영역은 자바 객체가 생성되자마자 저장되는 곳이다. 이렇게 생성된 객체는 `Minor GC`가 발생할 때 `Survivor` 영역으로 이동하게 된다.
* `Survivor` 영역은 `Survivor1`과 `Survivor2` 두 영역으로 나뉘는데, `Minor GC`가 발생하면 `Eden`과 `Survivor1`에 활성 객체를 `Survivor2`로 복사한다.
* 활성이 아닌 객체는 자연스럽게 `Survivor1`에 남아있게 되고, `Survivor1`과 `Eden` 영역을 클리어 한다. (결과적으로 활성 객체만 `Survivor2`)로 이동하게 된 것이다.
* 다음번 `Minor GC`가 발생하면 같은 원리로 `Eden`과 `Survivor2` 영역에서 활성 객체를 `Survivor1`으로 이동시키게 된다. 계속 이런 방식을 반복하면서 `Minor GC`를 수행한다.
* 이렇게 `Minor GC`를 수행하다가 `Survivor` 영역에서 오래된 객체는 `Old` 영역으로 옮기게 된다.
* 이러한 방식의 `GC` 알고리즘을 `Copy & Scavenge` 라고 한다. 이 방식은 속도가 매우 빠르며 작은 크기의 메모리를 콜렉팅하는데 매우 효과적이다.
* `Minor GC`의 경우에는 자주 일어나기 때문에 `GC`에 걸리는 시간이 짧은 알고리즘을이 적합하다.

### 2-2. Full GC

\-> 이 과정에서 stop the world 가 일어난다. \
\-> STW 은 객체를 효율적으로 제거하기 위해서 JVM 을 잠시 멈추는 것을 의미한다.&#x20;

* `Old` 영역의 가비지 컬렉션을 `Full GC` 라고 부르며 `Full GC`에 사용되는 알고리즘을 `Mark & Compact`라고 한다.
* `Mark & Compact` 알고리즘은 객체들의 참조를 확인하면서 참조가 연결되지 않은 객체를 표시한다. 이 작업이 끝나면 사용되지 않는 객체를 모두 표시하고 이 표시된 객체를 삭제한다.
* `Full GC`는 속도가 매우 느리며, `Full GC`가 일어나는 도중에는 순간적으로 자바 애플리케이션이 멈춰버리기 때문에, `Full GC`가 일어나는 정도와 시간은 애플리케이션의 성능과 안정성에 아주 큰 영향을 미친다.

## 3. GC 가 중요한 이유

* 가비지 컬렉션 중에서 마이너 GC의 경우에는 보통 0.5 이내에 끝나기 때문에 큰 문제가 되지 않지만, 그러나 FULL GC의 경우에는 자바 애플리케이션이 멈춰 버리기 때문에, 문제가 될 수 있다.
* 멈추는 동안 사용자의 요청이 큐에 들어있다가, 순간적으로 요청이 한꺼번에 들어오기 때문에 과부하에 의한 여러 장애를 만들 수 있다.
* 따라서 원활한 서비스를 위해서는 `GC`가 어떻게 일어나게 하느냐가 시스템의 안정성과 성능에 큰 변수로 작용할 수 있다.

## 4.G1GC

* 자바 9부터 기본 GC 로 자리를 잡은 G1GC 는 기존처럼 일일이 객체를 뒤져서 객체를 제거하지 않는다.&#x20;
* 기본적으로 힙영역을 young generation / old generation 으로 구분하여 사용한다.&#x20;
* 대신 메모리가 많이 차 있는 영역(region) 을 우선적으로 확인해 GC 한다.&#x20;
* 이전의 GC 들은 객체가 살아남으면 에덴 -> 서바이버1 -> 서바이버2 의 영역을 순차적으로 이동했지만, G1GC 는 아니다. G1GC 는 더욱 효율적이라 생각되는 위치에 재할당 시킨다. (더 이상 영역에서 순서가 보장되지 않는다)
*