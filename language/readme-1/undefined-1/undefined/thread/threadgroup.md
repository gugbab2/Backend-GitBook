# ThreadGroup

## ThreadGroup

* `ThreadGroup` 은 쓰레드의 관리를 용이하게 만들기 위한 클래스이다.
  * 하나의 어플리케이션에는 여러 종류의 쓰레드가 있을 수 있으며,&#x20;
  * 만약 `ThreadGroup` 가 없다면, 용도가 다른 여러 쓰레드를 관리하기 어려울 것이다.. \
    (물론 쓰레드를 관리하기위한 쓰레드 풀 있다)
  * `ThreadGroup` 은 기본적으로 트리 구조를 가지며, 하나의 `ThreadGroup` 이 다른 `ThreadGroup` 에 속할 수도 있다.&#x20;

## ThreadGroup 을 관리하기 위한 메서드

* `int activeCount()` : 실행중인 쓰레드이 개수를 리턴한다.
* `int activeGroupCount()` : 실행중인 쓰레드 그룹의 개수를 리턴한다.
* `int enumerate(Thread[] list)`&#x20;
  * 현재 쓰레드 그룹에 있는 모든 쓰레드의 매개 변수로 넘어온 쓰레드 배열에 담는다.
  * **`enumerate()` 메서드는 매개변수에 `ThreadGroup` 을 담거나,`Thread` 를 담는데,**&#x20;
  * **배열을 만들기 이전에 `activeCount()` 메서드 호출을 통해서 크기를 미리 지정해 두면 보다 효율적인 객체 생성을 할 수 있다.**
* `int enumerate(Thread[] list, boolean recurse)`&#x20;
  * 현재 쓰레드 그룹에 있는 모든 쓰레드의 매개 변수로 넘어온 쓰레드 배열에 담는다.&#x20;
  * `recurse` 가 `true` 이면, 하위에 있는 쓰레드 그룹에 있는 쓰레드 목록도 포함된다.
* `int enumerate(ThreadGroup[] list)`&#x20;
  * 현재 그룹에 있는 모든 쓰레드 그룹을 매개변수로 넘어온 쓰레드 그룹 배열에 담는다.
* `int enumerate(ThreadGroup[] list, boolean recurse)`&#x20;
  * 현재 그룹에 있는 모든 쓰레드 그룹을 매개 변수로 넘어온 쓰레드 그룹 배열에 담는다.&#x20;
  * `recurse` 가 `true` 이면, 하위에 있는 쓰레드 그룹에 있는 쓰레드 목록도 포함된다.
* `String getName()` : 쓰레드 그룹의 이름을 리턴한다.
* `ThreadGroup getParent()` : 부모 쓰레드 그룹을 리턴한다.
* `void list()` : 쓰레드 그룹의 상세정보를 출력한다.
* `void setDaemon(boolean daemon)` : 지금 쓰레드 그룹에 속한 쓰레드들을 데몬으로 지정한다.
