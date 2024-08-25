# 새로운 STL 컨테이너

## unordered\_map

### unordered\_map

#### 기존의 map

* 기존의 map 은 key 값을 기반으로 정렬이 되었기 때문에,&#x20;
* **요소 삽입 / 제거가 빈번할 경우 성능이 저하된다.**\
  **-> 트리 균형을 맞추기 위해서, 트리를 재조합 해야 한다. (O(logN))**\
  **->  대부분의 경우 정렬을 원하지 않는 경우가 많고, 정렬을 하지 않는 unordered\_map 이 나왔다.**&#x20;

<figure><img src="../../../../.gitbook/assets/image (89).png" alt=""><figcaption></figcaption></figure>

#### 새로운 unordered\_map

* unordered\_map 은 자동으로 정렬되지 않는 컨테이너이고,&#x20;
* 요소는 해쉬 함수가 생성하는 index 기반의 bucket 안에 저장된다. \
  \-> 해쉬맵이라고도 한다. (O(1))\
  \-> 버킷은 배열이다.&#x20;
* 버킷을 사용하기 때문에, 기존의 map 보다 메모리 사용량이 증가한다.

<figure><img src="../../../../.gitbook/assets/image (91).png" alt=""><figcaption></figcaption></figure>

#### unordered\_map 의 동작 방식&#x20;

1. **입력값에 대해서 언제나 동일한 정수를 반환해주는 해쉬 함수를 통해서 해쉬값을 반환한다.** \
   \-> 해쉬 충돌의 문제도 있지만, 일단은 논외.. \
   \-> 해쉬값을 구해주는 해쉬 함수의 종류는 매우 다양하다.&#x20;
2. **그 해쉬값을 버킷에 저장한다.** \
   \-> 해쉬값을 저장하는 경우도 매우 다양하다. 그림은 나머지 연산을 사용하는 예제이다.

<figure><img src="../../../../.gitbook/assets/image (92).png" alt=""><figcaption></figcaption></figure>

3. 요소에 접근할 때도, 해쉬 함수를 해쉬값을 확인하고 해당 버킷에 저장된 요소를 가져온다. \
   **-> (O(1))**

<figure><img src="../../../../.gitbook/assets/image (93).png" alt=""><figcaption></figcaption></figure>

#### 해쉬 충돌

* **가끔 키가 다른데도 같은 해쉬 값이 나오는 경우가 있다..**&#x20;
* **이럴 경우, 하나의 버킷에 둘 이상의 데이터가 들어간다..**
* 해쉬 충돌을 해결하는 방법은 매우 다양하다!

<figure><img src="../../../../.gitbook/assets/image (94).png" alt=""><figcaption></figcaption></figure>

* **아래 예제에서는 버킷의 내용을 확인할 수 있는데, 해쉬 충돌이 매우 빈번하게 일어나는 것을 볼 수 있다.** \
  **-> 이런 경우 내부에서 버킷마다 vector 와 같은 자료구조를 사용해 저장할 수 있다.**

<figure><img src="../../../../.gitbook/assets/image (95).png" alt=""><figcaption></figcaption></figure>

## unordered\_set

* unordered\_map 과 동작하는 방식이 동일하다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (96).png" alt=""><figcaption></figcaption></figure>

## 범위 기반 for 반복문

### 기존 for 반복문

<figure><img src="../../../../.gitbook/assets/image (97).png" alt=""><figcaption></figcaption></figure>

### 범위 기반 for 반복문 (가독성이 좋은 방법)

* 배열과 STL 컨테이너에서 사용 가능하다. \
  \-> 값, 포인터, 래퍼런스 모두 사용이 가능하다.
* 컨테이너, 배열을 역순회할 수 없음&#x20;

<figure><img src="../../../../.gitbook/assets/image (98).png" alt=""><figcaption></figcaption></figure>
