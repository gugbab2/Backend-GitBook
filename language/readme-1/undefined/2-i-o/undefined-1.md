# 채팅 프로그램

## 채팅 프로그램 - 설계

지금까지 학습한 네트워크를 활용해서 간단한 채팅 프로그램을 만들어보자.

요구사항은 다음과 같다.

* 서버에 접속한 사용자는 모두 대화할 수 있어야 한다.&#x20;
* 다음과 같은 채팅 명령어가 있어야 한다.
  * 입장 `/join|{name}`
    * 처음 채팅 서버에 접속할 때 사용자의 이름을 입력해야 한다.
  * 메시지 `/message|{내용}`
    * 모든 사용자에게 메시지를 전달한다.
  * 이름 변경 `/change|{name}`&#x20;
    * 사용자의 이름을 변경한다.
  * 전체 사용자 `/users`
    * 채팅 서버에 접속한 전체 사용자 목록을 출력한다.
  * 종료 `/exit`
    * 채팅 서버의 접속을 종료한다.

### 채팅 프로그램 설계 - 클라이언트&#x20;

기존에 작성한 네트워크 프로그램과 기본 뼈대는 비슷하지만, 어느정도 기본 설계가 필요하다.&#x20;

#### 클라이언트 설계&#x20;

채팅은 실시간으로 대화를 주고받아야 한다. 그런데 기존에 작성한 네트워크 클라이언트 프로그램은 사용자의 콘솔 입력이 있을 때까지 무한정 대기하는 문제가 있다.

기존 클라이언트 프로그램의 문제&#x20;

```java
System.out.print("전송 문자: ");
String toSend = scanner.nextLine(); // 블로킹

// 서버에게 문자 보내기 
output.writeUTF(toSend); 
log("client -> server: " + toSend);
```

* 스레드는 사용자의 콘솔 입력을 대기하기 때문에, 실시간으로 다른 사용자가 보낸 메세지를 콘솔에 출력할 수 없다.&#x20;

콘솔의 입력을 기다리는 부분도 블로킹 되지만, 서버로부터 메시지를 받는 다음 코드도 블로킹 된다.&#x20;

```java
// 서버로부터 문자 받기
String received = input.readUTF(); // 블로킹
```

따라서 사용자의 콘솔 입력과 서버로부터 메시지를 받는 부분을 별도의 스레드로 분리해야 한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.27.38.png" alt=""><figcaption></figcaption></figure>

### 채팅 프로그램 설계 - 서버&#x20;

채팅 프로그램의 핵심은 한 명이 이야기하면 그 이야기를 모두가 들을 수 있어야 한다.&#x20;

따라서 하나의 클라이언트가 보낸 메시지를 서버가 받은 다음에, 서버에서 모든 클라이언트에게 메시지를 다시 전송해야 한다.&#x20;

이렇게 하려면 서버에서 모든 세션을 관리해야 한다. 그렇게 해야 모든 세션에 메시지를 전달할 수 있다.&#x20;

참고로 우리는 앞서 세션을 관리하는 세션 매니저를 만들어두었다.&#x20;

따라서 기존 구조를 잘 활용하면 채팅 서버를 쉽게 구축할 수 있다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.29.25.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.29.32.png" alt=""><figcaption></figcaption></figure>

* 클라이언트1이 서버로 "Hello" 라는 메시지를 전송한다.&#x20;
* 서버는 `SessionManager` 를 통해 연결된 모든 세션에 "Hello" 채팅 메시지를 전달한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.30.38.png" alt=""><figcaption></figcaption></figure>

* 각 세션은 자신의 클라이언트에게 "Hello" 메시지를 전송한다.&#x20;
* 모든 클라이언트는 서버로 부터 "Hello" 메시지를 전송 받는다.&#x20;

## 채팅 프로그램 - 클라이언트&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.31.48.png" alt=""><figcaption></figcaption></figure>

클라이언트는 다음 두 기능을 별도의 스레드에서 실행해야 한다.&#x20;

* 콘솔의 입력을 받아서 서버로 전송한다.&#x20;
* 서버로부터 오는 메시지를 콘솔에 출력한다.&#x20;

먼저 서버로부터 오는 메시지를 콘솔에 출력하는 기능을 개발해보자.&#x20;

```java
package chat.client;

import java.io.DataInputStream;
import java.io.IOException;

import static util.MyLogger.log;

public class ReadHandler implements Runnable{

    private final DataInputStream input;
    private final Client client;
    public boolean closed = false;

    public ReadHandler(DataInputStream input, Client client) {
        this.input = input;
        this.client = client;
    }

    @Override
    public void run() {
        try {
            while (true) {
                String receive = input.readUTF();
                System.out.println(receive);
            }
        } catch (IOException e) {
            log(e);
        } finally {
            client.close();
        }
    }

    public synchronized void close() {
        if (closed) {
            return;
        }

        // 종료 로직 필요시 작성
        closed = true;
        log("readHandler 종료");
    }
}
```

* `ReadHandler` 는 `Runnable` 인터페이스를 구현하고, 별도의 스레드에서 실행한다.&#x20;
* 서버의 메시지를 반복해서 받고, 콘솔에 출력하는 단순한 기능을 제공한다.&#x20;
* 클라이언트 종료시 `ReadHandler` 도 종료된다. 중복 종료를 막기 위해 동기화 코드와 `closed` 플래그를 사용한다.&#x20;
  * 참고로 예제 코드는 단순하므로 중요한 종료 로직이 없다.&#x20;
* `IOException` 예외가 발생하면, `client.close()` 를 통해 클라이언트를 종료하고 전체 자원을 종료한다.&#x20;

```java
package chat.client;

import java.io.DataOutputStream;
import java.io.IOException;
import java.util.NoSuchElementException;
import java.util.Scanner;

import static util.MyLogger.log;

public class WriteHandler implements Runnable {

    private static final String DELEMITER = "|";

    private final DataOutputStream output;
    private final Client client;

    private boolean closed = false;

    public WriteHandler(DataOutputStream output, Client client) {
        this.output = output;
        this.client = client;
    }

    @Override
    public void run() {
        Scanner scanner = new Scanner(System.in);
        try {
            String userName = inputUsername(scanner);
            output.writeUTF("/join" + DELEMITER + userName);

            while (true) {
                String toSend = scanner.nextLine(); // 블로킹
                if (toSend.isEmpty()) {
                    continue;
                }

                if (toSend.equals("/exit")) {
                    output.writeUTF(toSend);
                    break;
                }

                // "/" 로 시작하면 명령어, 나머지는 메시지
                if (toSend.startsWith("/")) {
                    output.writeUTF(toSend);
                } else {
                    output.writeUTF("/message" + DELEMITER + toSend);
                }
            }

        } catch (IOException | NoSuchElementException e) {
            log(e);
        } finally {
            client.close();
        }
    }

    private static String inputUsername(Scanner scanner) {
        System.out.println("이름을 입력하세요");
        String userName;
        do {
            userName = scanner.nextLine();
        } while (userName.isEmpty());
        return userName;
    }

    public synchronized void close() {
        if (closed) {
            return;
        }
        try {
            System.in.close();  // Scanner 입력 중지 (사용자의 입력을 닫음)
        } catch (IOException e) {
            log(e);
        }
        closed = true;
        log("writeHandler 종료");
    }
}
```

* WriteHandler 는 사용자 콘솔의 입력을 받아서 서버로 메시지를 전송한다.&#x20;
* 처음 시작시 InputUsername() 을 통해 사용자의 이름을 입력 받는다.&#x20;
* 처음 채팅 서버에 접속하면 /join|{name} 을 전송한다. 이 메시지를 통해 입장했다는 정보와 사용자의 이름을 서버에 전달한다.&#x20;
* 메시지는 다음과 같이 설계된다.&#x20;
  * 입장 `/join|{name}`&#x20;
  * 메시지 `/message|{내용}`&#x20;
  * 종료 `/exit`
* 만약 콘솔 입력시 `/` 로 시작하면 `/join`, `/exit` 같은 특정 명령어를 수행한다고 가정한다.&#x20;
* `/` 를 입력하지 않는다면 일반 메시지로 보고 `/message` 에 내용을 추가해서 서버에 전달한다.

`close()` 를 호출하면 `System.in.close()` 를 통해 사용자의 입력을 닫는다. 이렇게 하면 `Scanner` 를 통한 콘솔 입력인 `scanner.nextLine()` 코드에서 대기하는 스레드에 다음 예외가 발생하면 대기 상태에서 빠져나올 수 있다.&#x20;

`java.util.NoSuchElementException: No line found`

서버가 연결을 끊은 경우 클라이언트 자원이 정리되는데, 이때 유용하게 사용된다.

`IOException` 예외가 발생하면 `client.close()` 를 통해 클라이언트를 종료하고, 전체 자원을 정리한다.

```java
package chat.client;

import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.net.Socket;

import static network.tcp.SocketCloseUtil.closeAll;
import static util.MyLogger.log;

public class Client {

    private final String host;
    private final int port;

    private Socket socket;
    private DataInputStream input;
    private DataOutputStream output;

    private ReadHandler readHandler;
    private WriteHandler writeHandler;
    private boolean closed = false;

    public Client(String host, int port) {
        this.host = host;
        this.port = port;
    }

    public void start() throws IOException {
        log("클라이언트 시작");
        socket = new Socket(host, port);
        input = new DataInputStream(socket.getInputStream());
        output = new DataOutputStream(socket.getOutputStream());

        readHandler = new ReadHandler(input, this);
        writeHandler = new WriteHandler(output, this);
        Thread readThread = new Thread(readHandler, "readHandler");
        Thread writeThread = new Thread(writeHandler, "writeHandler");
        readThread.start();
        writeThread.start();
    }

    public synchronized void close() {
        if (closed) {
            return;
        }
        writeHandler.close();
        readHandler.close();
        closeAll(socket, input, output);
        closed = true;
        log("연결 종료 : " + socket);
    }
}
```

* 클라이언트 전반을 관리하는 클래스이다.&#x20;
* `Socket`, `ReadHandler`, `WriteHandler` 를 모두 생성하고 관리한다.&#x20;
* `close()` 메서드를 통해 전체 자원을 정리하는 기능도 제공한다.

```java
package chat.client;

import java.io.IOException;

public class ClientMain {

    private static final int PORT = 12345;

    public static void main(String[] args) throws IOException {
        Client client = new Client("localhost", PORT);
        client.start();
    }
}
```

## 채팅 프로그램 - 서버1&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-07 20.49.42.png" alt=""><figcaption></figcaption></figure>

채팅 프로그램 서버의 경우 기존에 작성한 네트워크 프로그램의 서버에서 필요한 기능을 추가하면 된다.

```java
package chat.server;

import java.io.*;
import java.net.Socket;

import static network.tcp.SocketCloseUtil.closeAll;
import static util.MyLogger.log;

public class Session implements Runnable{

    private final Socket socket;
    private final DataInputStream input;
    private final DataOutputStream output;
    private final CommandManager commandManager;
    private final SessionManager sessionManager;

    private boolean closed = false;
    private String username;

    public Session(Socket socket, CommandManager commandManager, SessionManager sessionManager) throws IOException {
        this.socket = socket;
        this.input = new DataInputStream(socket.getInputStream());
        this.output = new DataOutputStream(socket.getOutputStream());
        
        this.commandManager = commandManager;
        this.sessionManager = sessionManager;
        this.sessionManager.add(this);
    }

    @Override
    public void run() {
        try {
            while (true) {
                String received = input.readUTF();
                log("client -> server : " + received);

                commandManager.execute(received, this);
            }
        } catch (IOException e) {
            log(e);
        } finally {
            sessionManager.remove(this);
            sessionManager.sendAll(username + "님이 퇴장했습니다");
            close();
        }
    }

    public void send(String message) throws IOException {
        log("server -> client : " + message);
        output.writeUTF(message);
    }

    public synchronized void close() {
        if (closed) {
            return;
        }
        closeAll(socket, input, output);
        closed = true;
        log("연결 종료 : " + socket);
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }
}
```

* `CommandManager` 는 명령어를 처리하는 기능을 제공한다. 바로 뒤에서 설명한다.&#x20;
* `Session` 의 생성 시점에 `SessionManager` 에 `Session` 을 등록한다.&#x20;
* `username` 을 통해 클라이언트의 이름을 등록할 수 있다. 사용자의 이름을 사용하는 기능은 뒤에서 추가하겠다. 지금은 값이 없으니 `null` 로 사용된다.&#x20;

#### run()&#x20;

* 클라이언트로부터 메시지를 전송받는다.&#x20;
* 전송 받은 메시지를 `commandManager.execute()` 를 사용해서 실행한다.&#x20;
* 예외가 발생하면 세션 매니저에서 세션을 제거하고, 나머지 클라이언트에게 퇴장 소식을 알린다. 그리고 자원을 정리한다.

#### send(String message)

* 이 메서드를 호출하면 해당 세션의 클라이언트에게 메시지를 보낸다.&#x20;
