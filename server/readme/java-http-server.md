# Java HTTP Server

## Java HTTP Server

* 바로 이전 HTTP Server 를 구축할 때, Request, Response 값을 사용하기 위해서 문자열을 편집해야 하는 불편함이 있었다.
* 이 불편함을 해결한 객체가 HttpServer 이며, 기존과 같은 문자열 편집 없이 간편하게 응답, 요청에 대한 헤더, 바디, 메서드 등 정보를 얻을 수 있다.

```java

// HttpHandler 는 handle 추상클래스 하나만 가진 인터페이스이기에 람다를 통해 익명클래스를 구현할 수 있다.
public abstract HttpContext createContext (String path, HttpHandler handler) ;

public interface HttpHandler {
    /**
     * Handle the given request and generate an appropriate response.
     * See {@link HttpExchange} for a description of the steps
     * involved in handling an exchange.
     * @param exchange the exchange containing the request from the
     *      client and used to send the response
     * @throws NullPointerException if exchange is <code>null</code>
     */
    public abstract void handle (HttpExchange exchange) throws IOException;
}
```

```java
public void run() throws IOException {
    InetSocketAddress address = new InetSocketAddress(8080);
    HttpServer httpServer = HttpServer.create(address, 0);

    httpServer.createContext("/", (exchange) -> {
        // 1. Request
        displayRequest(exchange);

        // 2. Response
        String content = "Hello, localhost!\n";
        sendContent(exchange, content);

    });

    httpServer.createContext("/gugbab2", (exchange) -> {
        // 1. Request
        displayRequest(exchange);

        // 2. Response
        String content = "Hello, gugbab2!\n";
        sendContent(exchange, content);

    });


    httpServer.start();
}

private void sendContent(HttpExchange exchange, String content) throws IOException {

    byte[] bytes = content.getBytes();
    exchange.sendResponseHeaders(200, bytes.length);

    OutputStream outputStream = exchange.getResponseBody();
    Writer writer = new OutputStreamWriter(outputStream);

    writer.write(content);
    writer.flush();
}

private void displayRequest(HttpExchange exchange) throws IOException {
    String method = exchange.getRequestMethod();
    System.out.println("method = " + method);

    URI uri = exchange.getRequestURI();
    String path = uri.getPath();
    System.out.println("path = " + path);

    Headers headers = exchange.getRequestHeaders();

    for (String key : headers.keySet()) {
        System.out.println("key : value = " + key + " : " + headers.get(key));
    }
    System.out.println("headers = " + headers);

    InputStream inputStream = exchange.getRequestBody();
    Reader reader = new InputStreamReader(inputStream);

    CharBuffer charBuffer = CharBuffer.allocate(1_000_000);

    reader.read(charBuffer);
    charBuffer.flip();

    String body = charBuffer.toString();
    System.out.println("body = " + body);
}
```

## Java NIO(Non-Blocking I/O)

* #### Java NIO 등장 배경
  * I/O 작업을 Blocking 방식으로 구현하게 되면 클라이언트가 I/O 를 진행할 때마다 쓰레드가 진행하는 작업을 중지하고 I/O 가 끝나기 만을 기다린다.\
    \-> 각 I/O 는 영향을 미치지 않게 하기 위해서 클라이언트 별로 Thread 를 만들어 연결시켜주어야 한다.
  * 위의 경우 Thread 수는 접속자 수가 많아질 수록 Thread 수도 많아지게 된다. 이는 CPU 의 부하(오버헤드)를 초래하게 되고, 성능의 악영향을 주게 된다.
  * 이를 해결하기 위한 방법이 Non-Blocking I/O 이다.
* #### NIO
  * 기본적으로 **NIO 는 IO 를 진행하는 동안 쓰레드의 작업을 중단시키지 않는다.**(제어권을 IO에 넘기지 않는다..)\
    \-> **호출된 함수는 바로 결과를 반환한다.**
  * 또한, IO 는 스트림으로 단 방향으로만 흐름이 연결되지만, NIO 는 Channels/Buffers/Selectors 를 통해 양방향 흐름의 연결이 가능하다.
* #### Channels
  * 일반적인 NIO 는 Channel 에서 시작된다.
  * 채널은 읽고 쓸 수 있지만, 스트림은 일반적으로 단뱡향(읽기, 쓰기) 으로만 가능하다.
  * **채널은 항상 버퍼에서 읽거나, 버퍼에 쓴다.**
* #### Buffers
  * NIO 버퍼는 NIO 채널에서 데이터를 주고 받는 단위(?) 이기 때문에, 무조건적으로 사용된다.
  * 버퍼는 기본적으로 데이터를 쓸 수 있는 메모리 블록이다.
  * 버퍼라는 메모리 블록은 NIO Buffer 객체로 래핑(?)되어 메모리 블록으로 작업하기 쉽게 일련의 메서드를 제공한다.
* #### Channels 와 Buffers 를 이용한 Non-Blocking I/O
  * 하나의 스레드는 버퍼에 데이터를 읽도록 채널에 요청할 수 있다.\
    \-> 버퍼에서 데이터를 읽고, 쓰기 위해서 채널에 요청한다
  * 채널이 버퍼로 데이터를 읽는 동안 쓰레드는 다른 작업을 수행 할 수 있다.
  * 데이터가 채널에서 버퍼로 읽어지면, 쓰레드는 해당 버퍼를 이용한 Processing(처리) 를 계속 할 수 있다.(콜백함수??)
* #### Selectors
  * 셀렉터를 사용하면 하나의 스레드가 여러 채널을 처리할 수 있게 된다.
  * 적어도 하나의 채널이 사용할 준비가 되거나 중단 조건이 발생할 때까지 프로그램 흐름을 차단할 수( block) 있다.
  * 메서드가 반환 되면 쓰레드는 채널에 준비 완료 된 이벤트를 처리할 수 있다.\
    \-> 즉 하나의 쓰레드에서 여러 채널을 관리할 수 있다.

## Java Lambda expression(람다식)

* 람다식은 함수를 하나의 식으로 표현한 것으로, 함수를 람다식으로 표현하면 메소드 이름이 필요가 없기 때문에, 익명 함수의 한 종류라고 볼 수 있다.
* 함수형 인터페이스에서는 인스턴스를 만들어 이름이 있는 함수를 구현하는 것이 불필요하다고 생각 될 때 람다를 사용해 아래와 같이 간편하게 구현할 수 있다.
* 하지만 너무 과한 람다식의 사용은 가독성을 저하 시킬 수 있다.

```java
// 기존 함수 방식
private String helloWorld(){
    return "hello, world!";
}

// 람다 방식
() -> "hello, world!";
```

* #### Java Functional interface(함수형 인터페이스)
  * @FunctionalInterface 어노테이션을 통해서 인터페이스에 하나의 추상클래스만 갖도록 제한하는 역할을 한다.
  * 함수형 인터페이스를 통해서 아래와 같은 익명클래스를 람다를 통한 가독성 높은 코드로 변경할 수 있게 되었다.
  * 람다로 만들어진 순수함수는 함수형 인터페이스로만 선언이 가능하다.

<pre class="language-java"><code class="lang-java">// 기존의 익명 클래스
System.out.println(new MyLambdaFunction() {
            public int max(int a, int b) {
                return a > b ? a : b;
            }
        }.max(3, 5));

<strong>// 함수형 인터페이스를 이용한 람다
</strong><strong>@FunctionalInterface
</strong><strong>interface MyLambdaFunction {
</strong><strong>    public int max(int a, int b);
</strong><strong>}
</strong>
MyLambdaFunction lambdaFunction = (int a, int b) -> a>b ? a : b;
<strong>System.out.println(lambdaFunction.max(3, 5));
</strong>
</code></pre>
