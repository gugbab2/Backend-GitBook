# NIO (New IO)

## 자바 NIO ?

* 자바 1.4 부터 추가된 기능으로, 기존 IO 에서 속도 개선을 위해서 만들어진 기능이다.&#x20;
* NIO 는 지금까지 사용한 `Stream` 을 사용하지 않고, `Channel` 과 `Buffer` 를 사용한다.

## NIO 의 Buffer 클래스

* NIO 에서 제공하는 Buffer 클래스에서는 java.nio.buffer 클래스를 확장하여 사용한다.&#x20;

#### Buffer 클래스에서 제공하는 메서드&#x20;

* `int capacity()` : 버퍼에 담을 수 있는 크기 리턴&#x20;
* `int limit()` : 버퍼에서 읽거나 쓸 수 없는 첫 위치 리턴&#x20;
* `int position()` : 현재 버퍼 위치 리턴&#x20;

```java
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;
import java.nio.file.Path;
import java.nio.file.Paths;
import java.nio.file.StandardOpenOption;

public class NioFileExample {
    public static void main(String[] args) {
        // 파일에 쓰기
        String data = "Hello, NIO!";
        Path path = Paths.get("nio_example.txt");

        try (FileChannel fileChannel = FileChannel.open(path, 
                                StandardOpenOption.CREATE, 
                                StandardOpenOption.WRITE)) {
            
            // 데이터를 바이트 버퍼에 넣기
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            buffer.put(data.getBytes());
            
            // 버퍼를 플립 (읽기 모드로 전환)
            buffer.flip();
            
            // 채널을 통해 버퍼를 파일에 쓰기
            fileChannel.write(buffer);
            System.out.println("파일에 데이터가 성공적으로 쓰였습니다.");
        } catch (IOException e) {
            e.printStackTrace();
        }

        // 파일에서 읽기
        try (FileChannel fileChannel = FileChannel.open(path, StandardOpenOption.READ)) {
            // 읽을 데이터를 위한 버퍼 할당
            ByteBuffer buffer = ByteBuffer.allocate(1024);
            
            // 파일에서 데이터를 버퍼로 읽기
            int bytesRead = fileChannel.read(buffer);
            
            // 버퍼를 읽기 모드로 플립
            buffer.flip();
            
            // 버퍼에서 데이터를 읽어서 출력
            byte[] byteArray = new byte[bytesRead];
            buffer.get(byteArray);
            System.out.println("파일에서 읽은 데이터: " + new String(byteArray));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```







...&#x20;
