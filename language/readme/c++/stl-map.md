# STL 컨테이너 - Map

## Map

* **Key 를 통해 Value 를 저장하는 방식이다.**&#x20;
* Key 는 중복될 수 없다.&#x20;
* Key 가 기본적으로 오름차순으로 저장된다.&#x20;

### Map 만들기&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-11 14.11.46.png" alt=""><figcaption></figcaption></figure>

### Map 삽입

* std:pair\<iterator, bool> 을 리턴하게 된다. \
  \-> iterator, bool 쌍으로 리턴한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-11 14.31.37.png" alt=""><figcaption></figcaption></figure>

* \[] 연산자를 통해서 key 에 대응하는 값을 참조로 반환할 수 있다.&#x20;
* **`"Coco"` 라는 키에 대응되는 값이 없을 때, `map["Coco"]` 로 읽는다고 하게 되면 기본값이 들어간다.** \
  **-> 때문에, 읽기 연산을 조심해야 한다..**&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-11 14.33.32.png" alt=""><figcaption></figcaption></figure>

* 기본적으로 key 를 기준으로 자동 정렬한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-11 14.39.02.png" alt=""><figcaption></figcaption></figure>

### Map 요소 찾기

<figure><img src="../../../.gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2024-04-11 14.44.08.png" alt=""><figcaption></figcaption></figure>

### Map 요소 수정

<figure><img src="../../../.gitbook/assets/image (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 개체를 Map 의 키로 사용해보자

#### 개체는 Map 의 키로 사용될 수 없다!

* **이전에 Map 의 키 값은 기본적으로 정렬된다고 했는데, 그렇다면 기본적으로 키 값은 비교될 수 있어야 한다.** \
  **-> 비교 연산자를 사용할 수 없다는 에러가 발생한다.**&#x20;

<figure><img src="../../../.gitbook/assets/image (3) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (4) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* 때문에, 두 키를 비교하는 함수를 만들어야 한다.**('<' 연산자 오버로딩)**\
  **-> 이후에 정상적으로 사용이 가능하다.**

<figure><img src="../../../.gitbook/assets/image (5) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## Map 의 장단점&#x20;

### 장점&#x20;

* **std::list 나 std::vector 보다 탐색 속도가 빠르다.**
  * `std::list, std::vector -> O(N)`\
    **-> 선형탐색 한다.**
  * `std::map -> O(logN)`\
    **-> 이진트리 탐색으로 선형탐색보다 탐색 속도가 빠르다.** \
    **-> 하지만 수정, 삭제의 경우 트리 재조정이 필요하기 때문에 std::vector 에 비해서 느리다.**

### 단점&#x20;

* **자동으로 정렬된다..** \
  **-> 정렬이 된다는 것 자체가 추가적인 작업이 필요하다는 의미 ..**\
  (정렬이 필요한 경우라면 장점이 될 수 있다)\
  **-> 정렬을 위해서 이진트리를 사용하는데, 이진트리는 배열과 같이 메모리가 연속적이지 않아도 동작한다. 이는 캐시메모리의 참조 지역성을 따르지 않기 때문에, CPU 최적화에 도움이 안된다.**&#x20;
* **해쉬맵(hashmap) 이 아님, 따라서 O(1) 이 아니다!**
* C++ 에 해결책이 있다!

