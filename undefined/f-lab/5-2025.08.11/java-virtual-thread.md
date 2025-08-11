# Java 의 미래 Virtual Thread

참고 링크&#x20;

* [https://www.youtube.com/watch?v=BZMZIM-n4C0](https://www.youtube.com/watch?v=BZMZIM-n4C0)

## 1. Virtual Thread 소개&#x20;

2018 년 Project Loom 으로 시작된 경량 스레드 모델&#x20;

2023년 JDK21 에 정식 feature 추가&#x20;

* 스레드 생성 및 스케줄링 비용이 기존 스레드보다 저렴&#x20;
* 스레드 스케줄링을 통해서 Non-Blocking I/O 지원&#x20;
* 기존 스레드를 상속하여 코드 호환

### 1.1 스레드 생성 / 스케줄링 속도가 빠르다.&#x20;

기존 자바 스레드는 생성 비용이 크다.

* 스레드 풀의 존재 이유&#x20;
* 사용 메모리 크기가 크다 (공간적 비용, 최대 2MB)
* OS 에 의해 스케줄링 (1:1 매핑)&#x20;

Virtual Thread 는 생성 비용이 작다.&#x20;

* 스레드 풀 개념 X&#x20;
  * 요청이 들어올때마다 생성되고,&#x20;
  * 응답이 나갈때마다 파괴된다.&#x20;
* 사용 메모리 크기가 작다. (최대 50KB)
* OS 가 아닌 JVM 내 스케줄링&#x20;

### 1.2 Non-Blocking I/O 지원&#x20;

증가하는 I/O blocking time \
Thread per request 모델에서 특히 병목이 되는 blocking time&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.33.21.png" alt=""><figcaption></figcaption></figure>

Non-Blocking I/O 는 Blocking time 을 획기적으로 줄여준다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.34.06.png" alt=""><figcaption></figcaption></figure>

Virtual Thread Non-Blocking I/O&#x20;

* JVM 스레드 스케줄링&#x20;
* Continuation 활용

### 1.3 기존 스레드 상속&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.39.01.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.40.33.png" alt=""><figcaption></figcaption></figure>

## 2. Virtual Thread 동작 원리&#x20;

### 2.1 일반 스레드 특징&#x20;

* 플랫폼 스레드&#x20;
* OS 에 의해 스케줄링&#x20;
* 커널 스레드와 1:1 매핑&#x20;
* 작업 단위 `Runnable`

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.43.58.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.44.21.png" alt=""><figcaption></figcaption></figure>

### 2.2 Virtual Thread 특징&#x20;

* 가상 스레드&#x20;
* JVM 에 의해 스케줄링&#x20;
  * Thread 는 생성 및 스케줄링 시 커널 영역 접근&#x20;
  * Virtual Thread 는 커널 영역 접근 없이 단순하게 Java 객체 생성 가능&#x20;
  * 즉, Virtual Thread 는 생성 시 시스템 콜 X&#x20;
* 캐리어 스레드와 1:N 매핑&#x20;
* 작업 단위 Continuation

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.45.55.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.46.34.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.47.55.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.48.33.png" alt=""><figcaption></figcaption></figure>

### 2.3 Continuation 작업 단위&#x20;

#### Kotlin 코루틴 동작방식&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.55.18.png" alt=""><figcaption></figcaption></figure>

#### Continuation (컨텍스트 스위칭 방식이랑 비슷)&#x20;

* 실행 가능한 작업 흐름&#x20;
* 중단 가능&#x20;
* 중단 지점으로부터 재실행 가능&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 18.59.44.png" alt="" width="563"><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 19.01.24.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 19.06.57.png" alt=""><figcaption></figcaption></figure>

#### Continuation 사용 이유&#x20;

* Thread 는 작업 중단을 위해 커널 스레드를 중단&#x20;
* Virtual Thread 는 작업 중단을 위해서 Continuation yield&#x20;
* 작업이 block 되어도 실제 스레드는 중단되지 않고 다른 작업 처리&#x20;
* 커널 스레드 중단이 없으므로 시스템 콜 X -> 컨텍스트 스위칭 비용이 낮다.&#x20;

## 3. 기존 스레드 모델 서버와 비교&#x20;

### 3.1 기존 스레드 모델&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 19.13.57.png" alt=""><figcaption></figcaption></figure>

### 3.2 Virtual Thread 모델&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-11 19.15.19.png" alt=""><figcaption></figcaption></figure>

## 4. 성능 테스트&#x20;

## 5. 서비스 적용 시 주의사항&#x20;

