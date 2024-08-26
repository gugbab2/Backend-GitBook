# Out of path 구조와 DPI 그리고 망중립

## Out of path 구조(Sensor)

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-12 16.23.28.png" alt=""><figcaption></figcaption></figure>

* Out of path 구조는 쉽게 Sensor(Port Mirroring) 로 생각할 수 있다.
* Out of path 센서와 연결되는 대표적인 장치는 L2 스위치이다. \
  **-> L2 스위치는 미러링 용 포트를 가지고 있고, 포트간의 통신 정보들을 미러링 용 포트를 통해 센서에 전달한다.**\
  **-> 해당 작업은 부하가 심한 작업이기 때문에, 높은 성능을 필요로 한다.**\
  \-> 미러링 전용 스위치(Tab Switch) 도 존재한다. (가격이 비싸다 ;;)

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-12 17.09.01.png" alt=""><figcaption></figcaption></figure>

* L2 스위치를 통해서 센터로 L2 Frame 이 전달되면, 프로세스(L7)로 정보가 전달되고 수집하게되며, 수집된 정보를 바탕으로 분석하게 된다.&#x20;
* 이 수집의 과정을 보조기억장치에 Write 한다고 생각할 수 있는데,
* **무조건적으로 네트워크에서 데이터가 전달되는 속도가 보조기억장치에 Write 속도보다 빠르다!**\
  **-> 속도차로 인한 손실이 발생할 수 있고,** \
  **-> 무손실 스캐너 장비의 가격은 무시무시하다 ;;**

## DPI 그리고 망중립

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-12 16.43.07.png" alt=""><figcaption></figcaption></figure>

* **우리나라에서는 유해사이트를 국가 차원에서 막고 있는데, 어떻게 막는 것일까?**
  * **유해사이트에 접근하려하면, ISP(Internet Service Provider) 단 Sensor 에서 응답을 유해사이트보다 먼저 응답을 주게된다.** \
    **-> Sensor 는 Only Read 만 가능하기에, 응답을 빨리 주는 방법밖에 없다.**&#x20;
  * **국가가 지정하는 유해사이트(포르노, 마약 ..)를 제외하고는 접근 제한을 해서는 안된다. (망중립의 원칙)**
* 유해사이트에 대한 응답으로 주고 있는 것들은 HTTP 이다. 이것을 한번 더 생각해보면 L7 HTTP 통신이 이루어지고 있다는 말이 된다.
* HTTP 통신 단위의 흐름을 보면 다음과 같다.\
  \-> Socket Stream > Segment > Packet&#x20;
* 결국 네트워크 내에서는 Packet 의 단위로 통신이 이루어지게 되는데, Packet 에는 IP, TCP, HTTP 헤더 정보를 담고 있다. (TCP/IP 통신이라면 HTTP 헤더 정보는 가지고 있지 않는다)

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-01-12 16.58.26.png" alt=""><figcaption></figcaption></figure>

* 다음과 같이 IP, TCP 헤더를 Packet 의 헤더,&#x20;
* HTTP 헤더(SPI), 그 외 부분(DPI)을 Packet 의 Payload 라 부른다.&#x20;
* **대한민국은 SPI 의 정보수집이 허용된 나라이며, DPI 정보수집(저장)은 도감청에 해당되는 범죄이다.** \
  **-> DPI 가 허용된다는 것은, 개인정보가 모두 노출되는다는 것을 의미하기에 상당히 민감한 사회문제이다.**&#x20;
