# 네트워크 기본 & TCP 통신

## 네트워크 프로그래밍이란?

* OSI 7 계층은 네트워크 전문가들을 위한 네트워크의 레이어이다.\
  \-> 우리는 자바에서 활용하는 대표적인 레이어를 살펴보자

### 애플리케이션 레이어

* HTTP(Hypertext Transfer Protocol), FTP(File Transfer Protocol), Telnet 들은 모두 TCP(Transfer Control Protocol) 통신을 한다.

### 트랜스포트 레이어

* 만약 자바에서 TCP 통신을 한다면 자바에서 제공하는 API 를 활용하면 된다.\
  \-> **애플리케이션 레이어에스 프로그래밍만 하면 트랜스포트 레이어에서의 처리는 자바가 다 알아서 한다.**
* TCP 통신은 연결기반 프로토콜로 상대방이 확실히 데이터를 받았는지, 보장할 수 있다.
* UDP(User Datagram Protocol) 통신은 TCP 와 달리, 상대방이 확실히 데이터를 받았는지 보장하지 않는다.\
  \-> TCP 에서는 연결을 확인하기 위해서 3 way handshake 작업을 하게 되는데, 이 작업은 리소스를 소모하는데, 세상에 모든 작업이 연결이 보장되어야 할 필요는 없다!\
  \-> 연결보다는 속도를 중시하는 실시간 스트리밍 등,, 의 서비스에서 사용하게 된다.

### 네트워크 레이어

* 일반적인 웹 애플리케이션에서는 80 이라는 번호의 포트를 사용한다\
  \-> 이건 정해져 있는것이다.
* 웹으로 SSL 이라는 안전한 통신을 하기 위해서는, 443 포트를 사용하게 된다.\
  \-> 일반적으로 사용되는 HTTPS 프로토콜은, SSL + HTTP 프로토콜을 함께 사용한다고 생각하면 된다.
* 이렇게 용도가 정해져 있는 포트는 다른 용도로 사용하지 않는 것이 좋다.

## Socket 클래스

* Socket 클래스는 클라이언트에서 객체를 생성해 사용한다.
* 서버에서 클라이언트에서 보낸 요청을 받으면, 요청에 대한 Socket 클래스를 생성해 데이터를 처리한다.\
  \-> Socket 클래스는 원격에 있는 장비와의 연결 상태를 보관하고 있다고 생각하면 된다.
* **서버에서 사용하는 클래스 : ServerSocket**

### 서버에서 사용하는 ServerSocket 클래스

* ServerSocket() : 서버 소켓 객체만 생성한다.
* ServerSocket(int port) : 지정된 포트를 사용하는 서버 소켓 객체를 만든다.\
  \-> backlog 를 지정하지 않을 시 50 개를 가지고 있다.
* ServerSocket(int port, int backlog) : 지정된 포트와 backlog 개수를 가지는 서버 소켓을 생성한다.
* ServerSocket(int port, int bakclog, InetAddress bindAddr) : 지정된 포트와 backlog 개수를 가지고, bindAddr 에 있는 주소에서의 접근만을 허용하는 서버 소켓을 생성한다.\
  \-> backlog : 큐의 개수로 생각하면 된다.(객체가 바빠 연결 요청을 처리못하고 대기시키는 최대 개수)
* 매개변수가 없는 ServerSocket 객체를 제외한 나머지 클래스 객체들은 생성되자마자, 연결을 대기할 수 있는 상태가 된다.
* **매개변수가 없는 Socket 객체는 별도의 연결작업을 해야만 대기가 가능하다 다음 메서드를 보자**
  * Socket accept() : 새로운 소켓 연결을 기다리고 연결이 되면 Socket 객체를 리턴한다.
  * void close() : 소켓 연결을 종료한다.\
    \-> 소켓을 다 사용했다면 항상 연결을 종료해야만 한다!

### 클라이언트에서 사용하는 클래스

* Socket() : 소켓 객체만 생성
* Socket(Proxy proxy) : 프록시 관련 설정과 함께 소켓 객체만 생성
* Socket(SocketImpl impl) : 사용자가 지정한 SocketImpl 객체를 사용하여 Socket 객체만 생성
* Socket(InetAddress address, int port) : 소켓 객체 생성 후 address 와 port 를 사용하는 서버에 연결
* Socket(InetAddress address, int port, InetAddress localAddr, int localPort) : 소켓 객체 생성 후 address 와 port 를 사용하는 서버에 연결하며, 지정한 localAddr와 localPort 에 접속
* **Socket(String host, int port) : 소켓 객체 생성 후, host 와 port 를 사용하는 서버에 연결**
* Socket(String host, int port, InetAddress localAddr, int loaclPort) : 소켓 객체 생성 후, host 와 port 를 사용하는 서버에 연결 후, 지정한 localAddr와 localPort 에 접속
* 위 3개 생성자를 제외한 나머지 생성자들은 모두 객체 생성과 함께 지정된 서버에 접속한다.
