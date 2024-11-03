# 자바 NIO 의 동작원리 및 IO 모델

> 참고 링크&#x20;
>
> [https://brewagebear.github.io/fundamental-nio-and-io-models](https://brewagebear.github.io/fundamental-nio-and-io-models/)\
> [https://jenkov.com/tutorials/java-nio/buffers.html](https://jenkov.com/tutorials/java-nio/buffers.html)

## 1. 바이트 코드(ByteBuffer)&#x20;

* Buffer 에 대한 사용법은 많은 블로그나 정보가 인터넷에 깔려있다. 그 중 볼만하다고 여겨지는 것은 [Java NIO Buffer](http://tutorials.jenkov.com/java-nio/buffers.html) 이다.&#x20;
* 여기서는 왜 바이트버퍼(`ByteBuffer`) 에 대해서 알아보자.&#x20;
  * 왜 바이트버퍼만 다루려 하는 것일까? \
    \-> 바이트 버퍼가 시스템 메모리를 직접 사용하는 다이렉트 버퍼를 만들 수 있는 버퍼 클래스이기 때문이다.&#x20;
  * 그렇다면 왜? 바이트버퍼만 다이렉트 버퍼를 만들 수 있게 되었을까?\
    \-> 운영체제가 사용하는 가장 기본적인 단위가 바이트이고, 시스템 메모리 또한 순차적인 바이트들의 집합이기 때문이다.&#x20;
* 우리가 해당 메서드를 통해 기존 `allocate()` 통해서 버퍼를 생성하는 것과 같이 다이렉트 버퍼를 만들 수 있다.&#x20;
  * 여기서 리턴 할 때 `DirecByteByffer` 를 생성해주는데, 이 녀석을 잘 보면 `MappedByteBuffer` 를 상속받는 객체임을 알 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 19.36.52.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 19.57.15.png" alt="" width="307"><figcaption></figcaption></figure>

### 일반 버퍼의 동작 과정

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 20.16.49.png" alt="" width="563"><figcaption></figcaption></figure>

* 일반 버퍼의 파일 I/O 에서는 JVM 힙 메모리 -> 네이티브 메모리로의 중간 복사가 발생하여, 데이터가 JVM 에서 운영체제 메모리로 복사 되는 추가 오버헤드가 생겨난다.&#x20;
* 대용량 파일을 다루거나 빈번한 I/O 가 있는 경우, 이 중간 복사 단계가 병목을 초래할 수 있다. &#x20;

#### 일반 버퍼의 파일 읽기

* 애플리케이션이 파일을 읽으려 하면 JVM 은 시스템 콜을 하고 운영체제는 디스크에서 해당 데이터를 읽어 네이티브 메모리(임시 버퍼) 에 임시 저장 한다.&#x20;
* 운영체제는 이 데이터를 JVM 힙 메모리의 버퍼로 복사한다.&#x20;
* 애플리케이션이 JVM 힙 메모리에 저장된 데이터를 읽는다. &#x20;

#### 일반 버퍼의 파일 쓰기&#x20;

* JVM 은 애플리케이션이 보낸 데이터를 JVM 힙 메모리에 버퍼로 저장한다.&#x20;
* JVM 은 이 데이터를 네이티브 메모리로 복사하여 운영체제에 전달한다. (시스템 콜)&#x20;
* 운영체제는 이 데이터를 디스크에 기록한다.&#x20;

### Direct Buffer 의 동작 과정

#### Direct Buffer 생성

* **`ByteBuffer.allocateDirect()` 메서드를 통해 Direct Buffer 를 생성하면 JVM 은 네이티브 메모리에 직접 버퍼를 할당한다.** &#x20;

#### 파일 읽기

* 애플리케이션이 Direct Buffer 를 통해 파일 읽기를 요청하면, 운영체제는 디스크에서 데이터를 읽어 바로 네이티브 메모리(Direct Buffer) 로 로드한다.&#x20;
* **애플리케이션은 Direct Buffer 에 매핑된 데이터를 바로 사용 가능하며, JVM 힙 메모리로의 추가 복사 과정이 필요하지 않다.**&#x20;

#### 파일 쓰기&#x20;

* 애플리케이션이 Direct Buffer 에 데이터를 쓰면, 해당 데이터는 네이티브 메모리 영역에 직접 기록된다.&#x20;
* 운영체제는 이 네이티브 메모리에 있는 데이터를 디스크에 직접 쓸 수 있어, 중간 복사 없이 효율적으로 데이터가 기록된다.&#x20;

### 어떻게 네이티브 메모리를 자바에서 사용이 가능한가?

* 그렇다면 빨라져서 좋은 건 알겠는데, 어떻게 네이티브 메모리를 자바에서 사용이 가능할까?&#x20;
* 자바는 보통 JVM 힙 메모리만을 사용하는 언어로 알려져 있지만, 자바 NIO 의 Direct Buffer 와 같은 기능을 통해 네이티브 메모리도 사용이 가능하다. **이를 가능하게 하는 주요 요인은 JNI(Java Native Interface) 와 Unsafe 클래스의 사용이다.**
* 자바는 이러한 방법을 통해 운영체제의 네이티브 메모리에 접근할 수 있는 기능을 제한적으로 제공하며, Direct Buffer 는 이 메커니즘을 이용해 네이티브 메모리에 접근한다.&#x20;

#### Java Native Interface(JNI)

* JNI 는 자바 애플리케이션이 네이티브 라이브러리(C, C++ 로 작성된 라이브러리) 와 상호작용 할 수 있도록 해주는 인터페이스이다. 자바는 보통 플랫폼 독립적인 언어로 설계되었지만, JNI 는 네이티브 코드와 통신을 가능하게 하여 네이티브 메모리에 직접 접근하는 기능을 제공한다.&#x20;
  * 자바 NIO에서 Direct Buffer는 **JNI를 통해 네이티브 메모리에 접근**하여, 메모리를 할당하고 관리할 수 있습니다.
  * Direct Buffer가 생성되면, JVM은 JNI 호출을 통해 운영체제에서 **메모리를 할당**받아 Direct Buffer에 할당합니다.
  * 이런 방식으로 **JVM 힙 메모리와 독립적인 네이티브 메모리 영역**을 사용하여 데이터를 처리할 수 있게 됩니다.

#### sun.misc.Unsafe 클래스(비공개 API)

* `sun.misc.Unsafe` 클래스는 자바의 비공개 API로, 일반적으로는 사용이 권장되지 않지만, **직접 메모리 할당과 조작**을 지원합니다. JVM 내부에서는 이 클래스를 사용해 네이티브 메모리 접근과 같은 저수준 작업을 수행합니다.
  * Direct Buffer도 내부적으로 `Unsafe` 클래스를 사용하여, 네이티브 메모리에 데이터를 직접 읽고 쓰는 기능을 구현합니다.
  * `Unsafe`를 통해 네이티브 메모리를 할당하거나 해제할 수 있으며, JVM의 가비지 컬렉터의 관리 밖에서 메모리를 관리하게 됩니다.
  * 이를 통해 네이티브 메모리에 접근하여, 파일 I/O와 같은 데이터 전송을 **중간 복사 없이** 수행할 수 있게 됩니다.

#### Direct Buffer의 구현 방식

* 자바 NIO의 `ByteBuffer.allocateDirect()` 메서드를 사용하면, JVM은 **운영체제의 네이티브 메모리에 직접 할당된 버퍼**를 생성합니다. Direct Buffer는 JVM 힙 메모리와 별도로 관리되며, 가비지 컬렉터가 아닌 **운영체제에서 메모리를 해제**합니다.
* Direct Buffer가 생성되는 과정을 요약하면 다음과 같습니다:
  1. **Direct Buffer 요청**: `ByteBuffer.allocateDirect()`가 호출되면, JVM은 Direct Buffer 생성 요청을 받습니다.
  2. **네이티브 메모리 할당**: JVM은 JNI를 사용하여 운영체제에 네이티브 메모리 할당을 요청하고, 이를 통해 메모리를 직접 할당받습니다.
  3. **메모리 접근 및 관리**: Direct Buffer는 `Unsafe` 클래스 또는 JNI를 통해 네이티브 메모리의 주소를 관리하며, 데이터를 직접 읽고 쓸 수 있는 기능을 제공합니다.
  4. **가비지 컬렉션과의 독립성**: Direct Buffer는 JVM 힙 메모리가 아니기 때문에, 가비지 컬렉션의 대상이 아닙니다. 대신, Direct Buffer가 더 이상 참조되지 않으면 JVM은 네이티브 메모리를 **직접 해제**하여 리소스를 반환합니다.
     1. Direct Buffer는 자바 객체로서 GC의 대상이지만, 객체가 참조를 잃어 GC가 수거할 때에만 JVM이 네이티브 메모리를 해제합니다.&#x20;
     2. 그렇기 때문에, **GC 타이밍에 의존하여 네이티브 메모리가 해제**되는 점에서 **메모리 관리의 지연**이 발생할 수 있습니다. 대규모 또는 빈번한 Direct Buffer 사용 시에는, **명시적으로 `ByteBuffer`를 초기화하거나 필요할 경우 강제로 GC를 호출하여 메모리를 정리**하는 방식으로 메모리 누수를 방지할 수 있습니다.

### 결론&#x20;

* 그렇다면 논 다이렉트 버퍼를 사용하지 않아야 할까?&#x20;
* 답은, 상황에 따라 다르다이다!
* 우리는 위에서 다이렉트 버퍼와, 논 다이렉트 버퍼를 비교하면서 힌트를 얻었다.&#x20;

1. **다이렉트 버퍼(Direct Buffer)**
   * 장점 : 읽고 쓰기가 시스템 메모리를 사용하므로 매우 빠르다.
   * 단점 : 시스템 메모리를 사용하기 때문에 할당 / 해제 비용이 다소 비싸다.
2. **논 다이렉트 버퍼(Non-Direct Buffer)**
   * 장점 : 시스템 메모리가 아닌 Heap 영역에 생성되기 때문에 할당 / 해제 비용이 보다 저렴하다.
   * 단점 : 두 번의 버퍼를 거치기 때문에 읽고 쓰기가 느리다.

* **따라서 일반적으로 성능에 민감하고 버퍼를 오랫동안 유지해서 사용할 필요가 있을 경우에는 다이렉트 버퍼를 유지하고, 그 외에는 논 다이렉트 버퍼를 사용하자!**&#x20;

## 2. 채널 (Channel)&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 20.17.15.png" alt="" width="563"><figcaption></figcaption></figure>

* 채널과 스트림은 상당히 유사하지만, 채널이 스트림의 확장이나 발전된 형태는 아니다. 일종의 게이트웨이라 볼 수 있는데, 단기 기존의 파일이나 소켓 등에서 사용하던 스트림을 NIO 기능을 이용할 수 있도록 도와주는 메서드를 제공한다.&#x20;
* 스트림과 차이점을 위주로 설명하면 아래와 같다.&#x20;
  * **데이터를 받기 위한 타겟으로 바이트버퍼(`ByteBuffer`) 를 사용**
  * **채널을 이용하면 운영체제 수준의 네이티브 IO 서비스들을 직간접적으로 사용 가능하다.**&#x20;
    * MMIO / 파일 락킹 등
  * **스트림과 달리 단방향 뿐 아니라, 양방향 통신도 가능하다.**&#x20;
    * 항상 양방향 통신이 가능한 것은 아니다 ;; \
      (소켓 채널은 양방향 통신을 지원하지만, 파일 채널은 지원하지 않는다)&#x20;
* 우리가 볼 채널은 파일채널과 소켓채널이다. 그 전에 `ScatteringByteChannel`, `GatheringByteChannel` 을 보고자 한다.&#x20;

### ScatteringByteChannel, GatheringByteChannel&#x20;

* [undefined.md](undefined.md "mention") 글에서 운영체제에서 지원하는 MMIO(Memory-mapped I/O) 에 대해 알아보았다.&#x20;
* NIO 채널에서는 효율적인 입출력을 위해 운영체제가 지원하는 네이티브 IO 서비스인 Scatter/Gather 를 사용할 수 있도록 위의 인터페이스를 지원해주고 있다.
* 이 인터페이스를 사용하므로, 시스템 콜과 커널 영역에서 프로세스 영역으로 버퍼 복사를 줄여주거나 또는 완전히 없애줄 수 있다.&#x20;

### 파일 채널 (FileChannel)

* 파일 채널은 파일의 관련된 작업들을 지원하는 채널들로 아래의 특징을 가지고 있다.&#x20;
  * `ByteChannel` 인터페이스를 구현한다.&#x20;
    * 이 인터페이스를 구현하므로 양방향성을 가질 수 있으나, 항상 그런 것은 아니다.&#x20;
  * `AbstactInterruptibleChannel` 추상 클래스를 구현하고 있다.&#x20;
    * 따라서, 비동기적인 방식으로 채널을 닫을 수 있게 되어 스레드와 채널 간의 상태 불일치가 발생하지 않도록 도와준다.&#x20;
  * `ScatteringByteChannel`, `GatheringByteChannel` 을 구현한다.&#x20;
    * 따라서 보다, 빠른 I/O 수행이 가능하다.&#x20;
* 위는 구현에 관련된 특징이라면, 실제로 파일 채널이 같는 특징은 다음과 같다.&#x20;
  * **파일 채널은 항상 블로킹 모드이며, 논 블로킹 모드로 설정할 수 없다.**&#x20;
  * **파일 채널 객체를 직접 만들 수 없다.**&#x20;
  * **대부분의 채널처럼 파일 채널도 가능하면 네이티브 I/O 서비스를 이용하려 한다.**&#x20;
  * **파일 채널 객체는 Thread-safe 하다.**

#### **파일 채널의 특징**&#x20;

* **파일 채널은 항상 블로킹 모드이며, 논 블로킹 모드로 설정할 수 없다.**&#x20;
  * **이 이유는 운영체제의 기능과 연관이 있는데, 현대의 운영체제들은 강력한 캐싱과 프리패치 알고리즘으로 디스크의 I/O 를 사용하지만, 논 블로킹 모드로 사용할 경우 처리 루틴이 달라져 이러한 기능들을 사용하는데 제한이 되기 때문이다.**&#x20;
  * 그렇다면, 파일 채널은 항상 블로킹 I/O 만 사용해야 할까?&#x20;
    * 비동기 I/O 모델은 포스팅 결론부에서 다룰 예정이기에, 이런 방식이 있다고만 알아두자. &#x20;
* **파일 채널 객체를 직접 만들 수 없다.**&#x20;
  * 파일 채널 객체는 이미 열려있는 파일 객체의 팩토리 메서드(`getChannel()`) 를 호출해서 생성된다. ~~따라서, `FileIntputStream` 으로 생성된 채널은 **읽기**만, `FileOutputStream` 으로 생성된 채널은 **쓰기**만 가능하다.~~
* **대부분의 채널처럼 파일 채널도 가능하면 네이티브 I/O 서비를 이용하려 한다.**
* **파일 채널 객체는 Thread-safe 하다.**
  * 같은 파일채널 인스턴스에 대해 여러 쓰레드들이 동시에 메서드를 호출해도 동기화 문제가 발생하지 않는다.
  * ~~이게 가능한 이유는 여러 쓰레드가 접근했을 때 만약 한 쓰레드가 파일 크기 또는 파일 채널의 포지션을 변경하는 부분을 수행하는 메서드를 호출하면 다른 쓰레드들은 해당 작업을 마무리할 때까지 기다렸다가 수행하기 때문이다.~~

#### 파일 채널의 속성1 - **파일 락킹(File Locking)**&#x20;

* 파일 락킹의 주요한 특징은 다음과 같다.&#x20;
  * **파일 락킹은 채널이 아닌 파일을 대상으로 하는 것이다.**&#x20;
  * **동일한 JVM 내부의 여러 스레드 사이가 아닌 외부 프로세스 사이에서 파일의 접근을 제어하기 위함이다.**&#x20;
* 파일 락킹의 경우에는 채널의 락이 걸렸을 경우, `FileLock` 객체가 리턴되며, 이 객체 내부의 메서드들을 통해서 공유 락인지 아닌지 (`isShared()`), 락을 해체할 것인지(`release()`) 등을 처리할 수 있다.&#x20;
* 파일 락킹 예시&#x20;
  * 아래 코드를 보면 try-resource 문으로 처리하여, `release()` 를 명시적으로 선언안해줘도 알아서 `release()` 가 된다.
  *   코드 자체는 채널을 가져오고 락을 걸어서 공유락이지 판단하고 릴리즈까지 하는 일련의 예시 코드라고 볼 수 있다.

      ```java
      public class FileLocking {

          public static void main(String[] args) {
              File file = new File("/Users/liquid.bear/Downloads/test.txt");

              try (FileChannel channel = new RandomAccessFile(file, "rw").getChannel()){
                  try (FileLock lock = channel.lock(0, Long.MAX_VALUE, true)) {
                      boolean isShared = lock.isShared();
                      System.out.println("Is Shared Lock? : " + isShared);
                  }
              } catch (IOException e) {
                  e.printStackTrace();
              }
          }
      }
      ```

#### 파일 채널의 속성2 - **MMIO(Memory-mapped I/O)**

* 파일 채널은 MMIO 를 지원한다. 추상 메서드인 `map()` 을 통해서 처리가 된다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 20.45.12.png" alt=""><figcaption></figcaption></figure>

* 이때 인자를 보면 `MapMode` 객체를 받는 것을 확인한 수 있는데, 이 객체는 3개의 상수값을 갖는다.&#x20;
  1. `READ_ONLY` : 버퍼에서 읽기만 가능한 모드
  2. `READ_WRITE` : 버퍼에서 읽기와 쓰기 모두 가능한 모드
  3. `PRIVATE` : 읽기와 쓰기 둘다 가능하지만 쓰기를 할 경우 복사본을 만들어 변경 내역을 별도로 보관하여 원본 파일에는 적용되지 않는다.
* 이렇게 `map()` 에서드를 통해서 MMIO 를 구현할 수 있다. 하지만, 주의할 점은 위에서 파일 락킹 같은 경우 `release()` 를 통해서, 해체되지만  MMIO 는 해제할 수 없고, 한번 생성되면 GC 가 발생할 때까지 남아있게 된다.&#x20;
  * 이렇게 설계 된 이유는 보안문제와 성능문제 때문이라고 한다.&#x20;
* MMIO 예시 코드는 다음과 같다.&#x20;
  *   어떤 파일의 버퍼를 만드는데 이때 빠른 I/O 를 처리하기 위해서 사용할 수 있다.&#x20;

      ```java
      ...
      private void initFileBuffer(int size, File file) throws FileNotFoundException {
          int bufferCount = size / FILE_BLOCK_SIZE;
          size = bufferCount * FILE_BLOCK_SIZE;

          try (RandomAccessFile fileData = new RandomAccessFile(file, "rw")) {
              fileData.setLength(size);
              ByteBuffer fileBuffer = fileData.getChannel()
                  .map(MapMode.READ_WRITE, 0L, size);

              divideBuffer(fileBuffer, FILE_BLOCK_SIZE, fileQueue);

          } catch (IOException e) {
              e.printStackTrace();
          }
      }
      ...
      ```

#### 파일 채널의 속성3 - 채널 간 직접 전송&#x20;

* 채널은 JVM 버퍼를 거쳐서 처리할 수도 있지만, **채널 사이에서 다이렉트로 데이터를 전송 할 수도 있다!**&#x20;
* 이런 기능은 `transTo()`, `transForm()` 메서드를 통해서 가능하다.&#x20;
* **예를 들어, 파일을 네트워크 전송을 해야 하는 상황이라고 할 때, 파일 채널과 소켓 채널을 사용해 JVM 버퍼를 거치지 않고 사용할 수 있다.** \
  **-> 중간에 JVM 버퍼를 거치지 않기 때문에, 속도 향상을 기대할 수 있다.**&#x20;
* 예제 코드

```java
package nio_copy;

import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.channels.FileChannel;

public class DirectTransterChannelIO extends MyTimer {
    private static final String DEST_PATH = "/Users/liquid.bear/Downloads/io_test_out.txt";
    public static void main(String[] args) throws IOException {
        start();
        copy();
        end("Channel TransferTo I/O");
    }

    public static void copy() throws IOException {
        try (FileInputStream fileInputStream = new FileInputStream(MyTimer.PATH);
            FileOutputStream fileOutputStream = new FileOutputStream(DEST_PATH);
            FileChannel fileInputChannel = fileInputStream.getChannel();
            FileChannel fileOutputChannel = fileOutputStream.getChannel()) {

            fileInputChannel.transferTo(0, fileInputChannel.size(), fileOutputChannel);
        }
    }
}
```

### 소켓 채널(SocketChannel)&#x20;
