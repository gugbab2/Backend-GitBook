---
description: HTTP Server 관한 내용을 공부해보자
---

# HTTP Server

## Stateful, Stateless

* ### stateful
* ### stateless(무상태 프로토콜)
  * HTTP 프로토콜은 상태를 유지하지 않는 stateless 프로토콜이다!
  * **때문에, 쿠키/세션/토큰이 필요하다.**
  * 무상태 프로토콜을 통해서 **무한한 서버 증설**이 가능하다.\
    \-> **마지막에 모든 응답에 필요한 모든 정보를 넘겨주기 때문에, 마지막 요청만으로도 정상적으로 처리가 된다.**

## 비연결성

* 서버가 모든 클라이언트와 연결되어 있다면, 연결을 위한 자원을 지속적으로 소모해야 한다.
* 때문에, **HTTP 는 한번의 통신 후 연결을 끊는다.**\
  \-> 서버의 자원을 효율적으로 사용할 수 있다.
* 하지만, **요청할 때마다 3 way handshake 시간이 추가된다.**\
  \-> Persistent Connections(지속 연결성)\
  \-> HTTP 2.0, 3.0 에서 최적화해 문제를 해결해나가고 있다.

## Java ServerSocket

* ServerSocket listener = new ServerSocket(8080, 0); 과 같은 코드를 Blocking 상태라고 한다.
* 제어권을 호출한 곳에 넘겨준 상태이고 파일 읽기, 쓰기 등도 모두 Blocking 동작으로 볼 수 있다.
* TCP 통신에서는 네트워크 상태 같은 요인에 의해 크게 지연될 수 있고, 만약 요청이 없다면 무작정 기다려야만 한다;;
* 멀티스레드, 비동기, 이벤트 기반 처리가 필요한 이유다.

<pre class="language-java"><code class="lang-java">// 1. Listen
// 해당 지역에서 클라이언트의 요청을 기다리고 있다.
// I/O 에서 기다리는 것을 Blocking 이라고 한다.
ServerSocket listener = new ServerSocket(8080, 0);
System.out.println("Listen!");

while (true) {

    // 2. Accept
    Socket socket = listener.accept();
    System.o<a data-footnote-ref href="#user-content-fn-1">u</a>t.println("Accept!");
    
    // 3. Request
    InputStream inputStream = socket.getInputStream();
    Reader reader = new InputStreamReader(inputStream);

    CharBuffer charBuffer = CharBuffer.allocate(1_000_000);
    
    reader.read(charBuffer);
    charBuffer.flip();
    
    System.out.println(charBuffer.toString());
    
    // Request 값 처리..
    
    // 4. Response
    String body = "Hello, world!";
    byte[] bytes = body.getBytes();
    String message = "" +
            "HTTP/1.1 200 OK\n" +
            "Content-Type: text/html; charset=UTF-8\n" +
            //Content-Length 는 byte 값으로 변경처리해야 한다.
            "Content-Length: " + bytes.length + "\n" +    
            "\n" +
            body;

    OutputStream outputStream = socket.getOutputStream();
    Writer writer = new OutputStreamWriter(outputStream);

    writer.write(message);
    writer.flush();
}

<strong>// 5. Close
</strong>socket.close();
</code></pre>

## Sync vs Async

* 처리해야 할 작업들을 어떠한 흐름으로 처리 할 것인가에 대한 관점
* 즉, 호출되는 함수의 작업 완료 여부를 신경쓰냐에 따라, 함수 실행 / 리턴 순차적인 흐름을 따르냐, 안따르냐 관심사
* Sync&#x20;
  * 호출하는 함수 A 가 호출되는 함수 B 의 작업 완료 후 리턴을 기다리거나,&#x20;
  * 바로 리턴 받더라도 미완료 상태라면 작업 완료 여부를 스스로 확인하며 신경쓰면 Sync 이다.
* Async
  * 함수 A 가 함수 B 를 호출할 때 콜백 함수를 함께 전달해서, 함수 B 작업이 완료되면 함께 보낸 콜백 함수를 실행한다.
  * 함수 A 는 함수 B 를 호출한 후로 함수 B 작업 완료 여부에는 신경쓰지 않는다.

## Blocking vs Non-Blocking

* 처리해야 하는 하나의 작업이, 전체적인 작업 흐름을 막느냐 안막느냐에 대한 관점
* 즉, 제어권이 누구에게 있느냐가 관심사이다.
* Blocking
  * A 함수가 B 함수를 호출하면, 제어권을 A 가 호출한 B 함수에 넘겨준다.
* Non-Blocking&#x20;
  * A 함수가 B 함수를 호출해도 제어권은 그대로 자신이 가지고 있는다.

[^1]: 
