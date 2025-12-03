# Gradle 멀티 모듈

#### 참고 링크&#x20;

[https://www.youtube.com/watch?v=ipDzLJK-7Kc](https://www.youtube.com/watch?v=ipDzLJK-7Kc)

## 1. 왜? 멀티 모듈 프로젝트 구조가 중요할까?&#x20;

* 멀티모듈 프로젝트는 프로젝트 초기에 이루어져야 하는 일련의 설계 결정이다.&#x20;
* 멀티모듈 프로젝트는 요소의 구조(structure) 와 그 관계(relationship) 에 관한 것이다.&#x20;

#### 결국은 나중에 변경하기가 매우 어렵다. 때문에 리스크를 줄이기 위한 시작이다!

### 계층 기준 멀티 모듈

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.03.59.png" alt=""><figcaption></figcaption></figure>

* 프로젝트 규모가 커지면서, 서비스 스펙이 늘어나고 공통 기능이 만들어지면서 발생하는 자연스러운 구조다.&#x20;
* 개발자 머리에서 강박관념으로 박혀있는, 중복을 줄여야 한다는 생각이 다음과 같은 상황을 만들어낸다.&#x20;

### 계층 기준 멀티 모듈의 문제점&#x20;

#### 1. Too many connections

Core & Command 사용하는 의존성 프로젝트는 커넥션 풀을 모두 할당받는다.&#x20;

#### 2. java.lang.NoClassDefFoundError

특정한 모듈이 하위 버전 라이브러리를 참조하는 경우 버전 업그레이드가 난해함

#### 3. Copy & Paste&#x20;

참조하는 곳이 너무 많아 일단 if.. else 분기 처리 copy & paste&#x20;

#### 4. All Build & Deploy&#x20;

배포시 작은 변화에도 전체 프로젝트를 빌드해야하는 생산성이 떨어지는 문제가 발생한다.&#x20;

***

## 2. 무엇? 을 기준으로 멀티 모듈 프로젝트 구조를 나뉘어야 할까?&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.34.47.png" alt=""><figcaption></figcaption></figure>

* 계층 기준 멀티 모듈 구조를 볼 때 역할, 책임, 협력 관계가 올바른지 생각해볼 필요가 있다.&#x20;
* 기술 베이스 지향적이 구조도 대안일 수 있다.&#x20;

### 기술 기반 멀티모듈 문제점

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.35.56.png" alt=""><figcaption></figcaption></figure>

* 기술 베이스 지향적인 구조로 구성하다보면 도메인을 어디 모듈에 연결해야 하는지? 의문점이 들기 시작한다. \
  (도메인과 매핑이 안되는게 어려워지는 포인트이다. )

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.38.45.png" alt=""><figcaption></figcaption></figure>

* 발표자 도메인의 경우 유관부서 및 업체연동 모듈이 추가되고 일정 시점이 지나서 버전 업 요청이 오게 되면 코드 변화가 광범위하다.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.41.24.png" alt=""><figcaption></figcaption></figure>

* 기술 베이스 지향적인 모듈로 시작했던 프로젝트가 유관부서 및 업체연동 모듈이 추가되면서 멀티 모듈 프로젝트의 규모는 2배 이상 커지게 된다.

### 그렇다면 어떻게?&#x20;

#### 우선 Core, Common 모듈은 삭제하고, 그래도 Core, Common 이 정말 필요한지? 한번 더 고민하자.&#x20;

* Core 란? 우린 이미 알고있다.&#x20;
* Common 이란? 우린 이미 잘 알고 있다.&#x20;
* **Core & Common 모듈의 장점보다 단점이 더 크다.**&#x20;

#### 바운디드 컨텍스트 기준으로 모듈을 나누자.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.51.08.png" alt=""><figcaption></figcaption></figure>

1. 서버모듈 (잦은변화)&#x20;
   1. batch
   2. admin
   3. api
2. 데이터모듈 (+도메인)&#x20;
   1. meta
   2. user
   3. chart&#x20;
3. 연동모듈 (큰변화)&#x20;
   1. aod
   2. vod
   3. photo
   4. billing
4. 클라우드(시스템) 모듈 (변화적음)
   1. config
   2. gateway&#x20;
   3. discovery&#x20;
   4. aws, gcp, azure

## 3. 어떻게? 실전 멀티 모듈 프로젝트를 구현해야 할까?

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.53.13.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-21 14.54.37.png" alt=""><figcaption></figcaption></figure>

