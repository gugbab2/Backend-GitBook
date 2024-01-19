# 대규모 부하분산을 위한 GSLB

## GSLB(Global Service Load Balancing)

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-14 09.50.02.png" alt=""><figcaption></figcaption></figure>

* 기존의 부하분산 시스템은 웹서버 아래 클론 된 웹서버를 여러대 두어 관리하는 시스템이었다.&#x20;
* 하지만, 서비스의 규모가 커지면서 외국에서 트래픽이 들어온 경우, **속도향상을 위해서 나라별로 서버를 분산시킬 필요성이 생겨났는데, 이때 사용되는 기술이 GSLB 이다.**&#x20;
* **GSLB 는 DNS 체계를 이용해, 클라이언트가 DNS 서버에 질의를 하게되면 해당 ISP 내 해당하는 도메인을 다시 알려줘 재질의를 할 있도록 하는 프로세스이다.**&#x20;
* **ISP 별로 다른 서버를 가지고 있기 때문에, 이 서버들을 동기화 하는 것이 가장 큰 이슈인데, 이때 CDN 기술을 사용해 서버들을 동기화해준다.** \
  \-> CDN 서비스만을 제공하는 회사들이 존재한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-01-14 09.55.06.png" alt=""><figcaption></figcaption></figure>
