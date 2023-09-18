# UDP 통신

* **UDP 는 비연결형 통신이기에, 서버를 실행시키지 않고 클라이언트만 실행시켜도 문제가 없다!**\
  **-> 반면 TCP 에러가 난다.**

## UDP 통신을 위해 알아야 하는 Datagram 관련 클래스

* 자바에서 UDP 통신은 클래스 하나에서 클라이언트, 서버의 역할을 모두 할 수 있다. \
  \-> 그 역할을 하는 클래스가 **DatagramSocket** 클래스이다.&#x20;

### **DatagramSocket** 클래스 생성자

* DatagramSocket() : 소켓 생성 후 사용 가능한 포트로 대기
* DatagramSocket(DatagramSocketImpl impl) : 사용자가 지정한 SocketImpl 객체를 사용하여 소켓 객체만 생성
* DatagramSocket(int port) : 소켓 객체 생성 후 지정된 port 로 대기
* DatagramSocket(int port, InetAddress address) : 소켓 객체 생성 후 address 와 port 를 사용하는 서버에 연결
* DatagramSocket(SocketAddress address) : 소켓 객체 생성 후 address 에 지정된 서버로 연결

### DatagramSocket 클래스에서 사용하는 메서드

* void receive(DatagramPacket packet) : 메서드 호출 시 요청을 대기하고, 만약 데이터를 받았을 때, packet 객체에 데이터를 저장&#x20;
* void send(DatagramPacket packet) : packet 객체에 있는 데이터를 전송

### DatagramPacket 클래스 생성자

* 단 하나의 생성자를 제외하고는 모두 데이터를 받기 위한 생성자이다. \

* DatagramPacket(byte\[] buf, int length) : length 만큼의 데이터를 받기 위한 객체 생성
* DatagramPacket(byte\[] buf, int length, InetAddress address, int port) :&#x20;
* DatagramPacket(byte\[] buf, int offset, int length) : 버퍼에 offset 이 할당되어 있는 데이터를 전송하기 위한 객체 생성
* DatagramPacket(byte\[] buf, int offset, int length, InetAddress address, int port) :&#x20;
* DatagramPacket(byte\[] buf, int offset, int length, SocketAddress address) :&#x20;
* DatagramPacket(byte\[] buf, int length, SocketAddress address) : \
  \
  \-> byte 배열은 전송되는 데이터이다. \
  \-> offset 은 전송되는 byte 배열의 첫 위치이다. \
  \-> length 는 전송되는 데이터의 크기이다. \
  \-> DatagramPacket.getData() : byte\[] 로 전송받은 데이터를 리턴\
  \-> DatagramPacket.getLength() : 전송받은 데이터의 길이를 리턴&#x20;
