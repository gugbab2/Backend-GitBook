# Inline 구조

## Inline 구조

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-12 16.00.00.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-12 16.12.04.png" alt=""><figcaption></figcaption></figure>

* Inline 구조의 대표적인 라우터를 쉽게 설명하면, NIC 이 여러개 담긴 호스트(PC) 라고 생각할 수 있다.
  * 인바운드, 아웃바운드 용 NIC 이 존재한다.
* **Inline 구조는 Kernel Mode 에서 모든 작업이 이루어져 아웃바운드 응답이 나가게 된다.**&#x20;
* **User Mode 까지 올라갈 수는 없을까?**&#x20;
  * **할 수 있다! 하지만, 속도 측면에서 손실이 크기 때문에, Kernel Mode 에서 모든 작업이 이루어진다.**&#x20;
  * Inline 장비의 작업은 복잡하지 않다.. 라우터를 예로들면 라우팅 테이블에 따라 스케줄링만 해주면 된다.&#x20;
  * 이 작업을 더욱 빠르게 하기 위해서 가속기가 달린 NIC 을 사용하기도 한다.&#x20;
* 네트워크를 구성할 때, 모든 기기의 속도를 평균치로 맞추어야 한다.&#x20;
  * **네트워크 내 한 기기만 Gbps 가 떨어진다면, 하향평준화 된다.**
