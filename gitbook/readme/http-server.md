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

## Blocking vs Non-Blocking
