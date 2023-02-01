---
description: HTTP Server 관한 내용을 공부해보자
---

# HTTP Server

## Java ServerSocket

```java
// 1. Listen
// 해당 지역에서 클라이언트의 요청을 기다리고 있다.
// I/O 에서 기다리는 것을 Blocking 이라고 한다.
ServerSocket listener = new ServerSocket(8080, 0);
System.out.println("Listen!");

while (true) {

    // 2. Accept
    Socket socket = listener.accept();
    System.out.println("Accept!");

    // 3. Request -> 처리 -> Response
    Reader reader = new InputStreamReader(socket.getInputStream());

    CharBuffer charBuffer = CharBuffer.allocate(1_000_000);
    reader.read(charBuffer);

    charBuffer.flip();
    System.out.println(charBuffer.toString());

    // 4. Response
    String body = "Hello, world!";
    byte[] bytes = body.getBytes();
    String message = "" +
            "HTTP/1.1 200 OK\n" +
            "Content-Type: text/html; charset=UTF-8\n" +
            "Content-Length: " + bytes.length + "\n" +
            "\n" +
            body;

    OutputStream outputStream = socket.getOutputStream();
    Writer writer = new OutputStreamWriter(outputStream);

    writer.write(message);
    writer.flush();

    // 5. Close
    socket.close();
```

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
