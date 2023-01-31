---
description: HTTP Client 관련한 지식이 공부해보자
---

# HTTP Client

## TCP/IP 통신

* 인터넷 프로토콜 스위트 : 인터넷에서 서로 정보를 주고받는 데 사용 되는 통신규약의 모음이다.
* TCP/IP 도 그 중 하나이며, TCP/IP 프로토콜 슈트라 불린다.

## TCP와 UDP

* TCP : 연결이 필요함, 순서 보장 (연결지향)
* UDP : 연결하지 않고 데이터를 보냄, 순서를 보장하지 않음 (비연결지향)

## Socket과 Socket API 구분

* Socket : 각 프로세스 간 통신의 종작점으로, Socket API 은 각 Socket 간에 연결되어 통신하는 것을 프로그래밍 하는 것이 Socket API 를 사용하는 것이다.&#x20;
* Socket API : 소켓을 위한 API ex) Berkeley sockets

## Socket&#x20;

* Socket 은 기본적으로 파일과 유사하게 다룰 수 있다.
* 자바에서는 소켓을 키보드 입력, 화면 출력, 파일 입출력 등을 Stream 으로 다룰 수 있다 \
  (자바 8에서 도입된 Stream API 아님! 주의,,)

## TCP 통신 순서

1. 서버는 접속 요청을 받기 위한 소켓을 연다(Listen)\
   \- 서버는 여러 클라이언트와 통신을 해야 하기 때문에 계속해서 Listen 상태를 유지한다.\
   \- 1 : 다
2. 클라이언트는 소켓을 만들고, 서버에 접속을 요청한다(Connect)
3. 서버는 접속 요청을 받아서 클라이언트와 통신할 소켓을 따로 만든다.(Accept)\
   \- 1 : 1
4. 소켓을 통해서 서로 데이터를 주고 받는다(Send && Receive 반복)
5. 통신을 마치면 소켓을 닫는다(Close 상대방은 Receive 로 인지할 수 있다)

## URI와 URL

* URI(Uniform Resource Identifier) : 특정 리소스를 식별하는 통합 자원 식별자를 의미한다.
* URL(Uniform Resource Locator) : 흔히 웹 주소라고도 하며, 네트워크 상에서 리소스가 어디 있는지 알려주기 위한 규약으로, URI 의 서브셋이다.
* URI 는 식별하고 URL 은 위치를 가리킨다.&#x20;

## 호스트(host) - 서버의 정보를 의미

*   ### IP 주소

    \- 클라이언트가 실제적으로 통신해야하는 논리적 주소를 의미한다.
*   ### Domain name

    \- www.naver.com 와 같이 우리가 쉽게 생각하는 웹사이트 주소를 의미하며 내부적으로는 IP 를 가리킨다.
*   ### DNS(Domain Name System)

    \- 도메인 이름과 IP 를 서로 변환하는 역할을 한다.

## 포트(port) - 컴퓨터 내 프로세스를 식별하는 단위

## path(경로)

* ### 절대경로 : 최상위 디렉토리부터 해당 파일까지 경유한 모든 경로를 전부 기입하는 방식이다.
* ### 상대경로 : 현재 파일이 존재하는 디렉토리를 기준으로 해당 파일까지의 위치를 작성한 경로이다.

## Java text blocks

* 자바 15 부터 추가 된 기능으로, 기존에 '+' 를 통해 문자열을 이어 붙히는 것보다 가독성이 높아지는 문법이다.

## Java InputStream과 OutputStream 를 사용한 Socket Client 예제

```java
// 1. connect
Socket socket = new Socket("example.com", 80);

String message = "GET / HTTP/1.1\n" +
                "Host: example.com\n" +
                "\n";
/*
OutputStream outputStream = socket.getOutputStream();
outputStream.write(message.getBytes());*/

// 2. write
OutputStream outputStream = socket.getOutputStream();
Writer writer = new OutputStreamWriter(outputStream);

writer.write(message);
writer.flush();

// 3. read
InputStream inputStream = socket.getInputStream();
Reader reader = new InputStreamReader(inputStream);

CharBuffer charBuffer = CharBuffer.allocate(1_000_000);

reader.read(charBuffer);

charBuffer.flip();

/*byte[] bytes = new byte[1_000_000];
int read_size = inputStream.read(bytes);

byte[] data = Arrays.copyOf(bytes, read_size);
String text = new String(reader.toString());*/
String text = charBuffer.toString();

// 4. close
socket.close();
```

## &#x20;Java try-with-resources

* JDK 1.7 부터 추가된 기능으로, 기존에는 자원을 모두 사용했으면 수동으로 닫아주어야 했으나,&#x20;
* 해당 문법을 통해 try 문이 끝남과 동시에, 자원을 반납한다

```java
// 1. connect
try(Socket socket = new Socket("example.com", 80)){
                System.out.println("Connect!");
                
                String message = "GET / HTTP/1.1\n" +
                                "Host: example.com\n" +
                                "\n";
                /*
                OutputStream outputStream = socket.getOutputStream();
                outputStream.write(message.getBytes());*/
                
                // 2. write
                OutputStream outputStream = socket.getOutputStream();
                Writer writer = new OutputStreamWriter(outputStream);
                
                writer.write(message);
                writer.flush();
                
                // 3. read
                InputStream inputStream = socket.getInputStream();
                Reader reader = new InputStreamReader(inputStream);
                
                CharBuffer charBuffer = CharBuffer.allocate(1_000_000);
                
                reader.read(charBuffer);
                
                charBuffer.flip();
                
                /*byte[] bytes = new byte[1_000_000];
                int read_size = inputStream.read(bytes);
                
                byte[] data = Arrays.copyOf(bytes, read_size);
                String text = new String(reader.toString());*/
                String text = charBuffer.toString();
                
                // 4. close
                socket.close();
}catch{
                //...
}

```
