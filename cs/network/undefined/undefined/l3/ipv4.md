# IPv4 주소의 기본 구조

## IPv4

<figure><img src="../../../../../.gitbook/assets/image (27).png" alt=""><figcaption></figcaption></figure>

* IPv4 는 32bit 의 주소체계를 갖는다.
* 8bit \* 4 로 볼 수 있는데, 8bit 는 2의 8제곱으로 256 가지의 수를 표현할 수 있다. (0 \~ 255)

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

* IP 는 8bit 씩 '.' 으로 구분하면 위와 같이 표기한다.
* 앞에 24bit 를 Network ID, 뒤 8bit 를 Host ID 라 부른다.\
  (Host : 컴퓨터 한대)\
  -> Network ID : 주소\
  -> Host ID : 상세주소
* 결국, 인터넷은 IP 주소로 목적지 주소로 찾아가야 한다.
* **예를 들어, 인터넷을 택배로 예를 들면 아래와 같이 생각할 수 있다.**
  * **택배들이 도착하는 물류센터 : Network ID 로 판별**
  * **구체적인 주소로 배달 : Host ID 로 판별**
