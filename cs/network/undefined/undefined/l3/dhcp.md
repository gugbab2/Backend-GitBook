# 인터넷 설정 자동화를 위한 DHCP

## 인터넷 사용 전에 해야 할 설정

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-06 10.50.11.png" alt="" width="347"><figcaption></figcaption></figure>

#### 기본적으로 인터넷 설정을 하기 위해 아래의 설정들이 필요하다.

* L3
  * IP 주소 설정
  * Subnet mask
  * Gateway IP주소
* DNS 주소

#### 인터넷 설정을 자동화 해주는 DHCP(Dynamic Host Configuration Protocol)

* DHCP 는 주소를 할당하는 서버, 할당 받으려는 클라이언트로 구성되어 있다.
* 복잡한 인터넷 설정을 자동으로 해준다고 할 수 있다.
* **내가 사용할 IP 주소를 서버가 알려준다는 것이 핵심이다!**

#### DHCP 동작과정

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-06 11.02.21.png" alt=""><figcaption></figcaption></figure>

1. 엔드포인트에서 전원이 켜지는 순간 네트워크로 **Broadcast Packet** 이 나간다.
   1. **우리 네트워크에서 DHCP 가 있니?**
   2. **해당 네트워크 안에 DHCP Server 가 존재해야 한다!**
2. DHCP 에서 요청에 대한 응답을 주게된다.
3. 엔드포인트가 응답을 받고, 이전에 DHCP 에게 IP 를 할당 받은적이 있다면, 그대로 사용하겠다고 DHCP 에게 요청을 보낸다.
   1. 처음이라면 어떤 IP 를 사용해야 하는지 묻는다.
4. DHCP 에서는 인터넷 사용에 필요한 IP, GW, DNS, Subnet Mask 등 다양한 정보를 응답으로 주게된다.
5. 엔드포인트에서는 응답을 받고, 해당 정보로 자동으로 설정하게 된다.
