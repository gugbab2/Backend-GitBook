# HashMap 이해

#### 참고 링크&#x20;

* [https://strong-park.tistory.com/entry/HashMap%EC%9D%B4-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94-%EC%9B%90%EB%A6%AC%EB%A5%BC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-putVal%EB%A9%94%EC%86%8C%EB%93%9C](https://strong-park.tistory.com/entry/HashMap%EC%9D%B4-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94-%EC%9B%90%EB%A6%AC%EB%A5%BC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-putVal%EB%A9%94%EC%86%8C%EB%93%9C)
* [https://mangkyu.tistory.com/430](https://mangkyu.tistory.com/430)

## 1. HashSet 초기화&#x20;

`HashMap` 은 배열로 되어 있어서 CRUD 작업에 O(1) 이 걸린다고 알고 있다.&#x20;

`HashMap` 의 기본 생성자를 보면 다음과 같이 초기용량(`capacity` = 16) 과 로드팩터(0.75) 를 설정한다는 것을 알 수 있다.&#x20;

* 로드 팩터는 용량 대비 실제 데이터 저장 사이즈의 비율(`size/capacity)` 을 의미하며,&#x20;
* 해당 비율이 지정된 비율을 넘어서면 `capacity` 를 2배씩 증가한다. &#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-25 23.59.13.png" alt=""><figcaption></figcaption></figure>

`HashMap` 은 내부적으로 버킷의 배열로 이루어져 있다.&#x20;

* 실제로는 `Node` 라는 클래스 배열로 선언되어 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.00.51.png" alt=""><figcaption></figcaption></figure>

`Node` 는 `HashMap` 의 내부클래스로 아래처럼 저장되어 있다.&#x20;

* `hash` : 키 값의 해시코드
* `key` : 버킷에 저장된 키&#x20;
* `value` : 키에 매핑된 값&#x20;
* `next` : 해시 충돌과 연관된 필드&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.05.17.png" alt=""><figcaption></figcaption></figure>

## 2. HashMap 데이터 저장&#x20;

`put()` 메서드는 내부적으로 `hash()` 메서드를 호출한 뒤, `putVal()` 메서드를 호출한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.07.21.png" alt=""><figcaption></figcaption></figure>

### hash()&#x20;

`hash()` 메서드를 살펴보면 키 객체의 `hashCode()` 메서드를 호출한 후 비트 연산자를 통해 **새로운 해시 코드를 생성한다.**&#x20;

* 키 객체에서 오버라이딩한 `hashCode()` 그대로 사용하는 것이 아니다!&#x20;
* 그 이유는 해시 코드의 분포를 개선하고, 해시 충돌을 최소화하고자 함이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.08.12.png" alt=""><figcaption></figcaption></figure>

### putVal()

`putVal()` 메서드는 크게 3가지 경우의 조건문으로 나뉜다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.11.47.png" alt=""><figcaption></figcaption></figure>

#### 1. if ((tab = table) == null || (n = tab.length) == 0)

* 현재 `table` 이 할당되지 않았거나, 길이가 0이라면 `resize()` 를 호출해서 새로운 `table` 을 할당&#x20;
* 처음 데이터를 넣을 때 테이블을 생성하기 위해서 실행 \
  (초기화 시점에 `table` 을 할당하지 않는것을 알 수 있다)&#x20;

#### 2. if ((p = tab\[i = (n - 1) & hash]) == null)

* 비트 연산을 사용해 나머지 연산처럼 구현하고(`(n-1) & hash`), 해당 인덱스에 버킷의 데이터가 없다면 새로운 `Node` 를 할당&#x20;
* 현재 버킷에 처음 들어가는 데이터일 경우 실행&#x20;

#### 3. else&#x20;

* value 를 대체하거나, 해시충돌이 발생할 때 실행&#x20;

