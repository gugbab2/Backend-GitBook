# STL 컨테이너 - Queue, Stack, Set, List

## Queue

* 선입 선출 (First-in first-out, FIFO) 자료구조\
  \-> 배열이기 때문에, Vector 와 장단점이 비슷하다.&#x20;

<figure><img src="../../../.gitbook/assets/image (6) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

### Queue 만들기&#x20;

<figure><img src="../../../.gitbook/assets/image (7) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (8) (1).png" alt=""><figcaption></figcaption></figure>

## Stack&#x20;

* 후입 선출(Last-in first-out, LIFO) 자료구조

<figure><img src="../../../.gitbook/assets/image (9) (1).png" alt=""><figcaption></figcaption></figure>

### Stack 만들기&#x20;

<figure><img src="../../../.gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

## Set&#x20;

* 맵의 키와 같이 정렬되는 컨테이너이다. \
  \-> **맵에서 사용되는 키와 동일하다.**
* 중복되지 않는 키를 요소로 저장한다.&#x20;
* 역시 이진 탐색 트리 기반이다.\
  \-> 오름차순
* 장단점 또한 Map 과 같다.

## List

<figure><img src="../../../.gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

### List 만들기

<figure><img src="../../../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (13).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (14).png" alt=""><figcaption></figcaption></figure>

### List 장단점&#x20;

#### 장점&#x20;

* 삽입과 제거에 걸리는 시간이 O(1)
* 어느 위치든 삽입 / 제거 가능&#x20;

#### 단점

* 탐색이 느린 편&#x20;
* 임의적으로 접근 불가&#x20;
* **메모리가 불연속적**\
  **-> CPU 캐시와 잘 동작하지 못한다.** \
  **-> 대부분의 경우 Vector 가 더 효과적으로 동작한다.**&#x20;

