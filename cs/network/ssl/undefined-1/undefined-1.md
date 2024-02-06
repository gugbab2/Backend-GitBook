# 비대칭키

## 비대칭키&#x20;

* **한쌍의 키**가 서로 상호작용 하는 구조를 갖는다.&#x20;
* **두 키중 하나로 암호화를 하면 다른 하나의 키로 복호화를 한다.** \
  **-> 각각의 키는 암호화, 복호화 하나의 기능만 수행한다.**&#x20;
* **보통 Public Key(공개키) 와 Private(비공개키) Key 로 구분하며 PKI (Public Key Infrastructure)기술의 근간이 된다.**&#x20;

## 비대칭키 사용방식&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-19 13.53.10.png" alt=""><figcaption></figcaption></figure>

* 예를 들어 평문에 Public Key 제곱값에 정해진 Mod 연산을 하는 것을 암호화라고 한다.&#x20;
* 암호문에 Private Key 제곱값에 정해진 Mod 연산을 하는 것을 복호화라고 한다. \
  **-> 위와 같이 12의 29 제곱은 엄청나게 큰 값이다..** \
  **-> 만약 29 라는 값이 커진다면 어떻게 될까? (상상할 수 없을만한 큰 값이다)**\
  **-> 때문에, Key 값을 키우는 것은 간단하면서도 강력한 보안 강화방법이다.**&#x20;

## 대칭키에 비해 비대칭키는 왜 효율이 떨어질까?

* 예를들어 AES-128(대칭키) vs RSA-2048(비대칭키) 의 성능을 비교한다면 어떨까? \
  \-> 별차이 없다..&#x20;
* **알고리즘에서 뒤에 붙는 숫자는 대부분 키의 자릿수를 의미한다고 했는데, 크면 클수록 더 많은 컴퓨팅 에너지를 소모한다.**&#x20;
* **컴퓨터 프로그램에서는 상관 없지만, 모바일 프로그램에서는 컴퓨팅 에너지를 소모하는 것은 고민해야 할 부분이다..**&#x20;