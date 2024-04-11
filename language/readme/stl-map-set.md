# STL 컨테이너 - Map, Set

## Map

* **Key 를 통해 Value 를 저장하는 방식이다.**&#x20;
* Key 는 중복될 수 없다.&#x20;
* Key 가 기본적으로 오름차순으로 저장된다.&#x20;

### Map 만들기&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-11 14.11.46.png" alt=""><figcaption></figcaption></figure>

### Map 삽입

* std:pair\<iterator, bool> 을 리턴하게 된다. \
  \-> iterator, bool 쌍으로 리턴한다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-11 14.31.37.png" alt=""><figcaption></figcaption></figure>

* \[] 연산자를 통해서 key 에 대응하는 값을 참조로 반환할 수 있다.&#x20;
* **`"Coco"` 라는 키에 대응되는 값이 없을 때, `map["Coco"]` 로 읽는다고 하게되면 기본값이 들어간다.** \
  **-> 때문에, 읽기 연산을 조심해야 한다..**&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-11 14.33.32.png" alt=""><figcaption></figcaption></figure>

* 기본적으로 key 를 기준으로 자동 정렬한다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-11 14.39.02.png" alt=""><figcaption></figcaption></figure>

### Vector 요소 찾기

<figure><img src="../../.gitbook/assets/스크린샷 2024-04-11 14.44.08.png" alt=""><figcaption></figcaption></figure>
