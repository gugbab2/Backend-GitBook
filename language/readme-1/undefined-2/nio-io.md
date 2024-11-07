# 자바 NIO 의 동작원리 및 IO 모델

> 참고 링크&#x20;
>
> [https://brewagebear.github.io/fundamental-nio-and-io-models](https://brewagebear.github.io/fundamental-nio-and-io-models/)\
> [https://jenkov.com/tutorials/java-nio/buffers.html](https://jenkov.com/tutorials/java-nio/buffers.html)
>
> [https://medium.com/dtevangelist/event-driven-microservice-%EB%9E%80-54b4eaf7cc4a](https://medium.com/dtevangelist/event-driven-microservice-%EB%9E%80-54b4eaf7cc4a)

## 1. 바이트 코드(ByteBuffer)&#x20;

* Buffer 에 대한 사용법은 많은 블로그나 정보가 인터넷에 깔려있다. 그 중 볼만하다고 여겨지는 것은 [Java NIO Buffer](http://tutorials.jenkov.com/java-nio/buffers.html) 이다.&#x20;
* 여기서는 왜 바이트버퍼(`ByteBuffer`) 에 대해서 알아보자.&#x20;
  * 왜 바이트버퍼만 다루려 하는 것일까? \
    \-> 바이트 버퍼가 시스템 메모리를 직접 사용하는 다이렉트 버퍼를 만들 수 있는 버퍼 클래스이기 때문이다.&#x20;
  * 그렇다면 왜? 바이트 버퍼만 다이렉트 버퍼를 만들 수 있게 되었을까?\
    \-> **운영체제가 사용하는 가장 기본적인 단위가 바이트이고, 시스템 메모리 또한 순차적인 바이트들의 집합이기 때문이다.**&#x20;
* 우리가 해당 메서드를 통해 기존 `allocate()` 통해서 버퍼를 생성하는 것과 같이 다이렉트 버퍼를 만들 수 있다.&#x20;
  * 여기서 리턴 할 때 `DirecByteByffer` 를 생성해주는데, 이 녀석을 잘 보면 `MappedByteBuffer` 를 상속받는 객체임을 알 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 19.36.52.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 19.57.15.png" alt="" width="307"><figcaption></figcaption></figure>

### 일반 버퍼의 동작 과정

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 20.16.49.png" alt="" width="563"><figcaption></figcaption></figure>

* 일반 버퍼의 파일 I/O 에서는 JVM 힙 메모리 -> 네이티브 메모리로의 중간 복사가 발생하여, 데이터가 JVM 에서 운영체제 메모리로 복사 되는 추가 오버헤드가 생겨난다.&#x20;
* **대용량 파일을 다루거나 빈번한 I/O 가 있는 경우, 이 중간 복사 단계가 병목을 초래할 수 있다.** &#x20;

#### 일반 버퍼의 파일 읽기

* 애플리케이션이 파일을 읽으려 하면 JVM 은 시스템 콜을 하고 운영체제는 디스크에서 해당 데이터를 읽어 네이티브 메모리(임시 버퍼) 에 임시 저장 한다.
* 운영체제는 이 데이터를 JVM 힙 메모리의 버퍼로 복사한다.&#x20;
* 애플리케이션이 JVM 힙 메모리에 저장된 데이터를 읽는다. &#x20;

#### 일반 버퍼의 파일 쓰기&#x20;

* JVM 은 애플리케이션이 보낸 데이터를 JVM 힙 메모리에 버퍼로 저장한다.&#x20;
* JVM 은 이 데이터를 네이티브 메모리로 복사하여 운영체제에 전달한다. (시스템 콜)&#x20;
* 운영체제는 이 데이터를 디스크에 기록한다.&#x20;

### Direct Buffer 의 동작 과정

#### Direct Buffer 생성

* **`ByteBuffer.allocateDirect()` 메서드를 통해 Direct Buffer 를 생성하면 JVM 은 네이티브 메모리에 직접 버퍼를 할당한다.** &#x20;
  * **네이티브 메모리에 할당된 Direct Buffer 는 GC 대상이 아니기 때문에, 메모리 해제를 코드 내에서 직접 해야하는데, 이로 인해 메모리 누수 위험이 증가한다.**&#x20;
  * **시스템 콜을 통해서 네이티브 메모리에 객체를 생성해야 하기 때문에, 초기화 비용이 비교적 크다!**&#x20;

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
* Direct Buffer가 생성되고 해제되는 과정을 요약하면 다음과 같습니다:
  1. **Direct Buffer 요청**: `ByteBuffer.allocateDirect()`가 호출되면, JVM은 Direct Buffer 생성 요청을 받습니다.
  2. **네이티브 메모리 할당**: JVM은 JNI를 사용하여 운영체제에 네이티브 메모리 할당을 요청하고, 이를 통해 메모리를 직접 할당받습니다.
  3. **메모리 접근 및 관리**: Direct Buffer는 `Unsafe` 클래스 또는 JNI를 통해 네이티브 메모리의 주소를 관리하며, 데이터를 직접 읽고 쓸 수 있는 기능을 제공합니다.
  4. **메모리 해제 (가비지 컬렉션과의 독립성)** : Direct Buffer는 JVM 힙 메모리가 아니기 때문에, 가비지 컬렉션의 대상이 아니다.
     * **그렇다면 어떻게 메모리를 해제할 수 있을까? (직접적인 방법을 권장!)**&#x20;
       * **간접적인 방법**&#x20;
         * **Direct Buffer 는 GC 의 관리 대상은 아니지만, 참조하는 객체가 더 이상 필요하지 않을 경우, 해당 객체에 대한 참조를 제거하면 GC 가 메모리를 해제하기 위한 신호를 받을 수 있다.**&#x20;
         * **그러나 이는 완전한 해제를 보장하지 않으며, GC 가 직접적으로 네이티브 메모리를 해제하지는 않는다**
       * **직접적인 방법**&#x20;
         * **Direct Buffer 를 생성하면 내부적으로 `Cleaner` 객체가 생성되며, `Cleaner` 객체를 사용해 직접적으로 네이티브 메모리를 해제할 수 있다.**&#x20;

### 결론&#x20;

* 그렇다면 논 다이렉트 버퍼를 사용하지 않아야 할까?&#x20;
* 답은, 상황에 따라 다르다이다!
* 우리는 위에서 다이렉트 버퍼와, 논 다이렉트 버퍼를 비교하면서 힌트를 얻었다.&#x20;

1. **다이렉트 버퍼(Direct Buffer)**
   * 장점 : 읽고 쓰기가 네이티브 메모리를 사용하므로 매우 빠르다.
   * 단점 : 네이티브 메모리를 사용하기 때문에 할당 / 해제 비용이 다소 비싸다.
2. **논 다이렉트 버퍼(Non-Direct Buffer)**
   * 장점 : 네이티브 메모리가 아닌 Heap 영역에 생성되기 때문에 할당 / 해제 비용이 보다 저렴하다.
   * 단점 : 두 번의 버퍼를 거치기 때문에 읽고 쓰기가 느리다.

* **따라서 일반적으로 성능에 민감하고 버퍼를 오랫동안 유지해서 사용할 필요가 있을 경우(대용량 파일)에는 다이렉트 버퍼를 유지하고, 그 외에는 논 다이렉트 버퍼를 사용하자!**&#x20;

## 2. 채널 (Channel)&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2024-11-03 20.17.15.png" alt="" width="563"><figcaption></figcaption></figure>

* 채널과 스트림은 상당히 유사하지만, 채널이 스트림의 확장이나 발전된 형태는 아니다. 일종의 게이트웨이라 볼 수 있는데, 단기 기존의 파일이나 소켓 등에서 사용하던 스트림을 NIO 기능을 이용할 수 있도록 도와주는 메서드를 제공한다.&#x20;
* 스트림과 차이점을 위주로 설명하면 아래와 같다.&#x20;
  * **데이터를 받기 위한 타겟으로 바이트버퍼(`ByteBuffer`) 를 사용**
    * 다양한 래핑 클래스들(`CharBuffer`, `ShortBuffer`, `IntBuffer`, `LongBuffer`, `FloatBuffer`, `DoubleBuffer`)이 존재한다.&#x20;
  * **채널을 이용하면 운영체제 수준의 네이티브 IO 서비스들을 직간접적으로 사용 가능하다.**&#x20;
    * MMIO / 파일 락킹 등
  * **스트림과 달리 단방향 뿐 아니라, 양방향 통신도 가능하다.**&#x20;
    * 항상 양방향 통신이 가능한 것은 아니다 ;; \
      (소켓 채널은 양방향 통신을 지원하지만, 파일 채널은 지원하지 않는다)&#x20;
* 우리가 볼 채널은 파일채널과 소켓채널이다. 그 전에 `ScatteringByteChannel`, `GatheringByteChannel` 을 보고자 한다.&#x20;

### ScatteringByteChannel, GatheringByteChannel&#x20;

* [undefined.md](undefined.md "mention") 글에서 운영체제에서 지원하는 MMIO(Memory-mapped I/O) 에 대해 알아보았다.&#x20;
* NIO 채널에서는 효율적인 입출력을 위해 운영체제가 지원하는 네이티브 IO 서비스인 Scatter/Gather 를 사용할 수 있도록 위의 인터페이스를 지원해주고 있다.
* 이 인터페이스를 사용하므로, **시스템 콜과 커널 영역에서 프로세스 영역으로 버퍼 복사를 줄여주거나 또는 완전히 없애줄 수 있다.**&#x20;

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
* **대부분의 채널처럼 파일 채널도 가능하면 네이티브 I/O 서비스를 이용하려 한다.**
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
* 이렇게 `map()` 메서드를 통해서 MMIO 를 구현할 수 있다. 하지만, 주의할 점은 위에서 파일 락킹 같은 경우 `release()` 를 통해서, 해체되지만  MMIO 는 해제할 수 없고, 한번 생성되면 GC 가 발생할 때까지 남아있게 된다.&#x20;
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

* 이제는 파일 채널과 양대산맥인 소켓 채널에 대해서 알아보자.&#x20;
* 소켓 채널은 파일 채널과 다르게 비교하여 몇 가지 다른 특징이 존재한다.&#x20;
  * **논 블로킹 모드 지원**&#x20;
  * **`SelectableChannel` 을 상속해서 `Selector` 와 함께 멀티플레스 I/O 가 가능하다.**&#x20;
* 기존 소켓, I/O 를 통한 네트워크 프로그래밍의 문제점이 있다.&#x20;
  * **블로킹 모드만 지원된다..**&#x20;
  * **이로 인해, 각 클라이언트 요청에 대해 하나의 스레드를 생성해야 하는 멀티스레드 모델을 사용하는 경우, 클라이언트가 많아질수록 생성해야 할 스레드 수가 기하급수적으로 증가합니다. 이는 자원(메모리, CPU 등)의 비효율적인 사용을 초래합니다.**
  * **스레드가 많아질수록 운영체제는 스레드 간의 컨텍스트 스위칭을 수행해야 하며, 이는 CPU의 작업 효율성을 떨어뜨립니다. 컨텍스트 스위칭은 스레드의 상태를 저장하고 복원하는 과정으로, 이 과정에서 CPU 시간이 소모됩니다.**
* 하지만 논 블로킹 소켓 채널이 도입됨에 따라 멀리플렉스 I/O 를 지원하는 셀렉터가 도입되어 기존의 문제가 해결되었다.&#x20;
* 참고로, 소켓 채널은 별 다른 설정을 하지 않으면 기본적으로 블로킹 모드로 설정된다. 따라서 아래와 같이 논 블로킹 모드로 바꿔주어야 한다.&#x20;

```java
SocketChannel socketChannel = SocketChannel.open(new InetSocketAddress("host ip", port));
socketChannel.configureBlocking(false);
```

* 소케 채널을 사용한 간단한 논 블로킹 채팅 프로그램은 결론 부분에서 확인하자.&#x20;

## 3. 셀렉터(Selector)&#x20;

* 우리가 셀렉터를 보기 전에 [이벤트 주도 아키텍처](https://en.wikipedia.org/wiki/Event-driven\_architecture)와 [리엑터 패턴](https://i5on9i.blogspot.com/2013/11/reactor-pattern.html)을 한번 쯤 볼 필요가 있다.&#x20;
* 하지만 간단하게 살펴보면, 이벤트 주도 아키텍처에서 리액터 패턴이라는 것이 존재하고, 셀렉터는 바로 이 리액터 패턴을 구성하는 요소 중에 리액터를 담당하는 놈이라고 이해하면 된다.&#x20;
  * **즉, 여러 채널의 셀렉션 키를 자신에게 등록하게 하고 등록된 채널의 이벤트 요청들을 나누어서 적절한 서비스 제공자에게 보내 처리하는 것이다 .**
  * **이를 통해 I/O 멀티플렉싱을 가능하게 한다.**&#x20;

### 3-1. 기존의 네트워크 프로그래밍(ThreadPool) 모델의 단점&#x20;

* 스레드 풀과 같은 기존 모델은 단순하고 널리 사용되었지만, 대규모 네트워크 서버나 실시간 처리가 필요한 애플리에키션 환경에서 몇가지 한계를 드러냈다.&#x20;
* 특히 클라이언트가 늘어나면서 스레드 풀 방식의 문제점이 더 두드러졌고, 이를 보완하기 위해서 Selector 와 같은 비동기 논블로킹 기술이 등장하게 되었다.&#x20;

#### 스레드 풀 모델의 단점&#x20;

1. 클라이언트 수 증가에 따른 스레드 수 증가&#x20;
   1. 스레드 풀은 요청마다 스레드를 할당하여 처리하는데, 클라이언트가 많아지면 스레드 수가 그에 비례해 증가한다.&#x20;
   2. 스레드가 많아질수록 메모리 사용량은 늘고, 스레드 간 컨텍스트 스위칭이 빈번해지면서 CPU 오버헤드가 증가한다.&#x20;
   3. 수천, 수만 개의 동시 연결이 필요한 **대규모 서버에서는 이 방식으로는 효율적으로 확장하기가 어렵다 ..**&#x20;
2. 스레드가 비효율적으로 대기하는 구조&#x20;
   1. 블로킹 I/O 사용할 때 스레드는 I/O 작업이 완료될 때까지 대기 상태로 전환된다. 응답이 느리거나, 지연이 발생하면 해당 스레드는 응답 대기 동안 다른 작업을 할 수 없게 되며, 이는 다중 클라이언트 환경에서 비효율 적이다.
   2. 대기 상태일 때는 CPU 자원을 반납하고, 대기가 풀리면 다시 CPU 자원을 할당 받아 사용한다.&#x20;

* 이러한 단점을 해결하기 위해서 I/O 멀티플렉싱 모델이 탄생했다!

#### 이를 해결하게 위해 I/O 멀티플렉싱 모델이 탄생하게 되었다!

### 3-2. 논 블로킹 모델과 셀렉터 동작 원리&#x20;

<figure><img src="../../../.gitbook/assets/image (149).png" alt=""><figcaption></figcaption></figure>

* I/O 멀티플렉싱 모델의 핵심적인 기능은 크게 세가지로 볼 수 있다.&#x20;

1. 셀렉터(Selector) : 리액터 패턴에서 리액터 역할을 해주는 객체&#x20;
2. 셀렉터블채널(SelectableChannel) : 셀렉터에 등록할 수 있는 채널들은 이 클래스를 상속받는다. 우리가 볼 예제는 소켓 채널 클래스이므로, 셀렉터에 등록할 수 있다.&#x20;
3. 셀렉션키(SelectionKey) : 특정 채널과 셀렉터 사이에서 해당 이벤트에 대한 내용에 정보를 들고 있는데, 이 값을 토대로 이벤트 요청을 처리한다.&#x20;

* 위 내용을 기반으로 전체적인 흐름을 보자면 다음과 같다.&#x20;

1. 채널을 셀렉터에 등록하면 이 등록에 관련된 채널과 셀렉터와 연관 정보를 갖는 셀렉션키가 셀렉터에 저장되고 리턴된다.&#x20;
2. 위의 셀렉션키를 토대로 어떤 채널이 자신이 등록한 모드에 대해 동작할 준비가 되면 셀렉션키는 그 준비상태를 내부적으로 저장한다.&#x20;
3. 소켓 서버의 예시를 들자면&#x20;
   1. 클라이언트를 `accept` 할 준비가 되면 셀렉션키는 준비상태가 된 것이고,&#x20;
   2. 이 때 셀렉터가 `select()` 메서드를 호출해서 자신에게 등록된 모든 셀렉션키의 상태를 체크하여&#x20;
   3. 상태가 준비상태라면 하나씩 순서대로 꺼내서 요청한 이벤트에 대해서 적절하게 처리한다.&#x20;

* 이제 이 동작을 기반으로 하나씩 살펴보자.&#x20;

#### 3-2-1. SelectobleChannel

* 위에서 이 클래스를 상속받은 클래스만이 셀렉터에 등록될 수 있다고 하였다.&#x20;
* 우리가 살펴볼 `SelectableChannel` 의 기능은 크기 2가지이다.&#x20;
  * **첫번째, 소켓채널에서 본 논블로킹 모드 활성화 기능** \
    **(해당  기능은 소켓 채널에서 다루었다)**
  * **두번째, 어떻게 셀렉터에 등록하는가?**
* 아래 `register()` 메서드를 통해서 채널을 셀렉터에등록할 수 있다. \
  (세번째 인자인 Object att 는 셀렉션키에서 설명하겠다)
* 여기서 ops 는 이벤트의 모드라고 볼 수 있다. 셀렉터에 등록할 수 있는 이벤트 모드들은 4가지가 있다.  (해당 이벤트들은 상수로 등록되어 있다)
  1. **OP\_READ** : 서버가 클라이언트의 요청을 `read` 할 수 있을 때 발생하는 이벤트
  2. **OP\_WRITE** : 서버가 클라이언트의 응답을 `write` 할 수 있을 때 발생하는 이벤트
  3. **OP\_CONNECT** : 서버가 클라이언트의 접속을 허락했을 때 발생하는 이벤트
  4. **OP\_ACCEPT** : 클라이언트가 서버에 접속했을 때 발생하는 이벤트

<figure><img src="../../../.gitbook/assets/image (150).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/image (151).png" alt=""><figcaption></figcaption></figure>

* 아래와 같은 코드로 채널을 셀렉터에 등록할 수 있다 .

<pre class="language-java"><code class="lang-java">// 셀렉터 생성
Selector selector = Selector.open();
// 서버소켓 채널 생성
ServerSocketChannel server = ServerSocketChannel.open();
<strong>server.configureBlocking(false); // 논블록킹 모드 활성화
</strong>
ServerSocket socket = server.socket();
SocketAddress addr = new InetSocketAddress(port);
socket.bind(addr); // 소켓 생성 후 해당 주소에 바인드

// 셀렉터에 생성된 ServerSocketChannel과 ACCEPT 이벤트 등록
server.register(selector, SelectionKey.OP_ACCEPT); 

</code></pre>

* 각 채널 구현체마다 등록될 수 있는 이벤트는 다른데 아래와 같다.&#x20;

| 채널 구현체              | 등록할 수 있는 이벤트                     |
| ------------------- | -------------------------------- |
| ServerSocketChannel | OP\_ACCEPT                       |
| SocketChannel       | OP\_CONNECT, OP\_READ, OP\_WRITE |
| DatagramChannel     | OP\_READ, OP\_WRITE              |
| Pipe.SourceChannel  | OP\_READ                         |
| Pipe.SinkChannel    | OP\_WRITE                        |

* 여러 개의 이벤트를 등록할 수 있는 채널은 아래와 같이 여러개의 이벤트도 등록할 수 있으며, 하나의 셀렉터에 여러개의 채널도 등록할 수 있다.&#x20;

```java
Selector selector = Selector.open();

SocketChannel channel1 = SocketChannel.open();
channel1.configureBlocking(false);
SocketChannel channel2 = SocketChannel.open();
channel2.configureBlocking(false);
ServerSocketChannel server = ServerSocketChannel.open();
server.configureBlocking(false);

server.register(selector, SelectionKey.OP_ACCEPT);
channel1.register(selector, SelectionKey.OP_READ);
channel2.register(selector, SelectionKey.OP_READ | SelectionKey.OP_WRITE);

Set<SelectionKey> keys = selector.keys();

for (SelectionKey key : keys) {
    System.out.println(key.channel().getClass() + " " + key.interestOps());
}
```

<figure><img src="../../../.gitbook/assets/image (152).png" alt=""><figcaption></figcaption></figure>

* 그림으로 보면 다음과 같을 것이다.&#x20;
  * 셀렉터는 이렇게 이벤트가 발생한 채널들만 선택해서 각 이벤트에 맞는 동작을 하도록 모든 이벤트들에 대한 컨트롤러 역할을 한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (153).png" alt=""><figcaption></figcaption></figure>

#### 3-2-2. SelectionKey&#x20;

* 어떤 채널이 어떤 셀렉터에, 어떤 이벤트 모드로 등록되었는지, 그 등록한 이벤트를 수행할 준비가 되었는지에 대한 정보들을 담고 있는 객체이다.&#x20;
  * 즉, 이벤트 처리에 대해서 셀렉터와 채널 사이에서 도와주는 역할을 하는 객체이다.&#x20;
* 셀렉션 키에는 크게 두가지 집합이 존재한다.&#x20;
  * **interest set**
    * 위에서 여러 채널과 이벤트를 등록하는 예시 코드 내 `key.interestOps()` 라는 메서드가 존재한다. 이 정보들은 `register()` 할 때 등록했던 상수들 값이다.&#x20;
    * **따라서 interest set 은 셀렉터에 등록한 이벤트 정보를 담고있는 집합이다.**&#x20;
  * **ready set**
    * **ready set 은 채널에서 이벤트가 발생하면 그 이벤트들을 저장하는 집합이다.**&#x20;
* 즉, 셀렉션키는 interest set, ready set 을 활용하여 이벤트 헨들링을 도와주는 역할을 한다.&#x20;
* `register()` 메서드의 세번째 인자인 `att` 는 셀렉션키에 참조할 객체를 추가하는 메서드이고, 해당 키에 참조할 객체가 있다면 그 객체를 리턴하고, 없다면 `null` 을 리턴한다.&#x20;

<figure><img src="../../../.gitbook/assets/image (154).png" alt=""><figcaption></figcaption></figure>

* 셀렉션키에 참조된 객체는 `attachement()` 메서드로 가져올 수 있으며, `register()` 로 등록이 가능하지만, `attach()` 메서드로도 등록할 수 있다.&#x20;
  * `attach()` 메서드로 등록된 객체는 `SelectionKey` 가 참조하고 있기 때문에, GC 대상이 되지 않는다. 때문에, `SelectionKey` 가 삭제되기 전에 명시적으로 `SelectionKey.attach(null)` 또는 `SelectionKey.cancel()` 을 호출하여 참조를 해제해야 한다.&#x20;
  * 그렇게 해야만 메모리 누수가 발생하지 않는다.&#x20;

#### 3-2-3. Selector

* 셀렉터는 위에서 언급한 바와 같이 등로된 채널들이 발생시킨 이벤트에 대해 적절한 처리 핸들러로 요청을 분기해주는 컨트롤러 역할을 한다.&#x20;
* 위에서 셀렉터에 대한 내용은 많이 언급했으니 중요한 특징만 가지고 설명하고자 한다.
* 셀렉터 또한 등록된 이벤트를 처리하기 위해서는 자신에게 등록된 채널과 연관된 셀렉션키에 대해서 알고 있어야 한다.&#x20;
* 그러므로 셀렉터 내부에는 셀렉션키에 대한 집합을 가지고 있다. 이 집합은 크게 3가지이며, 셀렉터 내부에는 아래의 집합들을 관리한다.&#x20;
  * **등록된 키 집합(Registered Key Set)**
    * 셀렉터에 등록된 모든 셀렉션키의 집합이다. 하지만 이 집합에 있는 모든 키가 유효하지는 않다.&#x20;
    * 메서드 : `Selector.keys()`&#x20;
  * **선택된 키 집합(Selected Key Set)**&#x20;
    * 등록된 키 집합 내 포함되어 있다.&#x20;
    * 셀렉션키가 수행 준비상태가 되어서 **ready set(이벤트가 발생)** 이 비어있지 않은 키들이 `Selector.select()` 메서드에 호출되어서 선택되었을 때 이 집합에 추가된다.&#x20;
  * **취소된 키 집합(Cancelled Key Set)**&#x20;
    * 등록된 키 집합 내 포함되어 있다.&#x20;
    * 등록을 해제하고 싶을 때 `SelectionKey.cancel()` 메서드로 등록을 취소할 수 있는데, 이 키는 바로 유효하지 않은 키로 설정되고 취소된 키 집합에 추가된다.&#x20;
* **주의 사항으로 셀렉터는 스레드 세이프하지만, 세 가지 키 집합은 스레드 세이프 하지 않으므로, 멀티스레드 환경에서는 반드시 동기화처리를 해주어야 한다.**&#x20;
* 이제 셀렉터의 동작 원리에 대해서 살펴보자. 셀렉터는 `select()`, `poll()` 과 같은 시스템 콜을 래핑한 것이다.&#x20;
  * **실제 사용 방식은 `select()` 메서드를 호출하면서 사용되는데 내부적으로 아래와 같은 방식으로 동작한다.**&#x20;
  * **아래 1\~3 동작과정을 반복하면서 진행하는데, 그 실행 시점과 블로킹 여부만 차이가 있다.**&#x20;

1. **취소된 키 집합을 체크한다.**
   * 만약 집합이 비어 있지 않다면
     * 이 집합에 저장된 각각의 키들은 셀렉터가 관리하는 세가지 집합에서 모두 삭제되어 각 키와 연관된 채널이 셀렉터에서 등록이 해제된다.
2. **등록된 키 집합을 체크한다.**
   * 만약 **ready set**이 비어있지 않은 셀렉션키가 존재한다면
     * 등록된 키 집합에 넣는다. (이미 존재한다면 그 키를 업데이트 처리만 한다.)
3. **셀렉터가** `selectedKeys()` **메서드를 호출한다.**
   * 저장된 선택된 키 집합을 가져오고, 그 안에 저장된 셀렉션키의 이벤트 형식에 따라 적절한 핸들러에게 처리를 넘긴다.

* 셀렉터가 제공하는 `select` 함수는 총 세가지이다.&#x20;

1. `select()`&#x20;
   * 블록킹되는 메서드이며, 선택된 키 집합이 비어있다면 키가 추가될 때까지 블록킹 된다. 그러다가 사용할 수 있는 키가 추가되면 ① \~ ③을 실행한다.
2. `select(long timeout)`&#x20;
   * 밀리세컨드마다 `select()` 함수와 동일하게 처리된다. 따라서, 해당 시간마다 블록킹이 된다.
3. `selectNow()`
   * 논블록킹 메서드이다. 따라서 이용 가능한 채널이 없으면 0을 리턴하고, 아니면 마찬가지로 등록된 키 집합안에 들어있는 셀렉션키의 개수를 리턴한다.

* 여기서 추가로, `wakeup()` 메서드는 스레드가 블로킹 되어 있는 경우 이 블로킹 된 스레드를 깨우는데 사용한다.

## 4. I/O 모델 및 간단한 블로킹IO & NIO 예제

### 4-1. I/O 모델&#x20;

* IO / NIO 를 다루면서 자주 했던 말이 블로킹과 논 블로킹이다.&#x20;
* 이것들은 I/O 모델이라는 개념에 속해있다. 이번 포스팅에서는 4가지 I/O 모델을 다루어보자.&#x20;
  * 블로킹(Blocking) I/O && 동기(Synchronous) I/O 모델&#x20;
  * 논 블로킹(Non-Blocking) I/O 모델
  * 비동기(Asynchronous) I/O 모델
  * I/O 다중화(Multiplexing) 모델

#### 4-1-1.  블로킹(Blocking) I/O && 동기(Synchronous) I/O 모델&#x20;

<figure><img src="../../../.gitbook/assets/image (157).png" alt=""><figcaption></figcaption></figure>

* 위 그림을 보면 어플리케이션은 커널에서 응답이 올 때까지 블로킹된다. \
  (다른 작업은 하지 못하고 waiting 상태가 된다)&#x20;
* 당연히 우리의 똑똑한 선배님들은 이러한 응답을 대기하는 대기시간이 발생하기 때문에, 이 시간을 줄일 수 없을까? 고민을 하게 되었고, 그렇게 나온 I/O 모델이 논 블로킹 I/O 모델이다.&#x20;

#### 4-1-2. 논 블로킹(Non-Blocking) I/O 모델

<figure><img src="../../../.gitbook/assets/image (158).png" alt=""><figcaption></figcaption></figure>

* 논 블로킹 모델은 그림과 같이 시스템 콜이 발생한 뒤에 응답이 끝날때까지 기다리는 것이 아니라, 제어권을 어플리케이션이 가지고 있다.&#x20;
* 그렇다면 비동기 통신이랑은 무엇이 다른 것일까?

#### 4-1-3. 비동기(Asynchronous) I/O 모델

<figure><img src="../../../.gitbook/assets/image (159).png" alt=""><figcaption></figcaption></figure>

* 가장 큰 차이점은 논 블로킹 I/O 모델처럼 주기적으로 처리 여부를 응답하는 것이 아니라, 커널에 시스템 콜을 한 뒤 어플리케이션은 다른 일을 하다가 커널이 콜백으로 완료 여부를 알려준다.&#x20;
  * 즉, I/O 처리가 완료된 타이밍에 결과를 회신하는 모델이다.&#x20;
* 차이점을 정리하자면 비동기 I/O 모델은 완료 했을 때 통지를 하지만, 논 블로킹 I/O 모델은 처리가 가능한 상태를 판단하면서 처리한다.&#x20;
* 여기서 논 블로킹 I/O 의 단점을 생각해 볼 수 있는데, 시스템 콜이 계속해서 발생할 수 있다는 단점이 있다..&#x20;
* 이러한 단점을 해결하기 위해서, 이벤트 등록(필요한 시점에만 물어보게끔 하는 것이다) 을 통해 이를 처리하는 방법을 고안하였고, 이 방법이 I/O 다중화 모델이다!

#### 4-1-4. I/O 다중화(Multiplexing) 모델

<figure><img src="../../../.gitbook/assets/image (161).png" alt="" width="550"><figcaption></figcaption></figure>

* `select` 시스템 콜은 `Selector.select()` 라고 볼 수 있을 것이며, **data ready** 부분은 셀렉션키의 **ready set** 이 존재하는 경우이다.&#x20;
* 즉, 우리가 공부한 NIO 는 I/O 다중화 모델을 구현할 수 있는 객체들이다. 이러한 개념들을 출발하여 오늘날 I/O 모델의 중심이라 볼 수 있는 이벤트 주도 아키텍처 등이 탄생했다고 볼 수 있다.&#x20;

### 4-2. 간단한 블로킹IO & NIO 예제&#x20;

#### 4-2-1. 블로킹 IO (전통적인 방식)&#x20;

* 블로킹 IO 에서 입출력 작업이 완료될 때까지 스레드를 기다린다. 이 방식은 동기적으로, 각 요청에 대해 스레드가 하나씩 할당되어 작업을 완료할 때가지 해당 스레드가 블로킹 상태로 유지된다.&#x20;
* 요청이 많아질 경우 많은 스레드를 생성해야 하며, 컨텍스트 스위칭 비용이 증가하여 성능에 영향을 끼칠 수 있다.&#x20;

```java
import java.io.*;
import java.net.*;

public class BlockingIOServer {
    public static void main(String[] args) {
        try (ServerSocket serverSocket = new ServerSocket(8080)) {
            System.out.println("Blocking I/O Server started on port 8080");
            while (true) {
                Socket clientSocket = serverSocket.accept(); // 클라이언트가 연결될 때까지 대기
                handleRequest(clientSocket); // 클라이언트 요청 처리
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static void handleRequest(Socket clientSocket) {
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(clientSocket.getInputStream()));
             PrintWriter writer = new PrintWriter(clientSocket.getOutputStream(), true)) {

            String request = reader.readLine(); // 요청이 들어올 때까지 블로킹
            System.out.println("Received: " + request);
            writer.println("Echo: " + request); // 응답 전송
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

* 블로킹 IO 동작&#x20;
  * `serverSocket.accept()` 가 호출되면 클라이언트의 연결 요청이 들어올 때까지 스레드가 블로킹된다.&#x20;
  * `clientSocket.getInputStream().readLine()` 도 클라이언트가 데이터를 보낼 때까지 블로킹된다.&#x20;
  * **즉, 모든 입출력 작업은 완료될 때까지 스레드가 대기 상태에 있어야 한다.**&#x20;

#### 4-2-2. NIO

* NIO 에서는 하나의 스레드로 여러 채널을 관리할 수 있다.&#x20;
* 각 채널은 논 블로킹 모드로 동작하여 준비된 작업만 처리한다.&#x20;
* 이를 위해서 `Selector` 를 이용해 이벤트가 발생한 채널을 감지하고 작업을 수행한다.&#x20;

```java
import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.ByteBuffer;
import java.nio.channels.*;
import java.util.Iterator;

public class NonBlockingIOServer {
    public static void main(String[] args) {
        try (Selector selector = Selector.open();
             ServerSocketChannel serverSocketChannel = ServerSocketChannel.open()) {
            
            serverSocketChannel.bind(new InetSocketAddress(8080));
            serverSocketChannel.configureBlocking(false); // 논블로킹 모드로 설정
            serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
            System.out.println("Non-blocking I/O Server started on port 8080");
            
            while (true) {
                selector.select(); // 이벤트 발생할 때까지 대기
                Iterator<SelectionKey> keyIterator = selector.selectedKeys().iterator();
                
                while (keyIterator.hasNext()) {
                    SelectionKey key = keyIterator.next();
                    keyIterator.remove();
                    
                    if (key.isAcceptable()) {
                        // 새로운 연결 수락
                        ServerSocketChannel server = (ServerSocketChannel) key.channel();
                        SocketChannel client = server.accept();
                        client.configureBlocking(false);
                        client.register(selector, SelectionKey.OP_READ);
                        System.out.println("Accepted connection from client");
                    } else if (key.isReadable()) {
                        // 데이터 읽기
                        SocketChannel client = (SocketChannel) key.channel();
                        ByteBuffer buffer = ByteBuffer.allocate(256);
                        int bytesRead = client.read(buffer);
                        
                        if (bytesRead == -1) {
                            client.close();
                        } else {
                            buffer.flip();
                            client.write(buffer); // Echo back to client
                            buffer.clear();
                        }
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}

```

* 논 블로킹 IO 동작&#x20;
  * `selector.select()` 메서드를 하나 이상의 채널에 이벤트가 발생할 때까지 대기한다. 이 호출은 블로킹 상태이지만, 이벤트가 발생한 채널의 작업만 처리하므로, CPU 와 메모리를 효율적으로 사용할 수 있다.&#x20;
  * `SelectionKey.OP_ACCEPT` 를 통해 클라이언트 연결이 수락되었을 때 이벤트를 받고,&#x20;
  * `SelectionKey.OP_READ` 로 읽기 가능 상태인 채널에서 데이터를 읽는다.&#x20;
  * **`SocketChannel.configureBlocking(false)` 로 설정했기 때문에, IO 작업을 수행할 때도 스레드가 블로킹 되지 않으며 다른 작업을 처리할 수 있다.**&#x20;
* I/O 멀티플렉싱을 위한 핵심적인 기능들을 크게 세가지로 볼 수 있다.&#x20;

1. 셀렉터(Selector) : 리액터 패턴에서 리액터 역할을 해주는 객체&#x20;
2. 셀렉터블채널(SelectableChannel) : 셀러터에 등록할 수 있는 채널들은 이 클래스를 상속받는다. 우리가 볼 예제는 소켓 채널 클리스이므로, 셀렉터에 등록할 수 있다.
3. 셀렉션키(SelectionKey) : 특정 채널과 셀렉터 사이에서 해당 이벤트에 대한 내용에 대한 정보를 들고 있는다. 이 값을 토대로 이벤트 요청을 처리한다.&#x20;

* 위 내용을 토대로 전체적인 흐름을 보자면&#x20;

1. 채널을 셀렉터에 등록하면 이 등록에 관련된 채널과 셀렉터와 연관 정보를 갖고 있는 셀렉션키가 셀렉터에 저장되고, 리턴된다.&#x20;
2. 위의 셀렉션키를 토대로 어떤 채널이 자신이 등록한 모드에 대해 동작할 준비가 되면 셀렉션키는 그 준비상태를 내부적으로 저장한다.&#x20;
3. 소켓 서버의 예시를 들자면 클라이언트 `accept` 할 준비가 되면 **셀렉션키는 준비 상태가 된 것**이고, 이 때 셀렉터가 `select()` 메서드를 호출해서 자신에게 등록된 모든 셀렉션키를 검사하여 준비상태면, 하나씩 순서대로 꺼내서 요청한 이벤트에 대해 적절하게 처리한다.&#x20;
