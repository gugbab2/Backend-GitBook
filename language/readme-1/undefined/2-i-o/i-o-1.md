# I/O 기본1

## 1. 스트림 시작1&#x20;

<figure><img src="../../../../.gitbook/assets/image (132).png" alt=""><figcaption></figcaption></figure>

* 자바가 가진 데이터를 hello.dat 라는 파일에 저장하려면 어떻게 해야할까?&#x20;
* 자바 프로세스가 가지고 있는 데이터를 밖으로 보내려면 출력 스트림을 사용하면 되고, 반대로 외부 데이터를 자바 프로세스 안으로 가져오려면 입력 스트림을 사용하면 된다.&#x20;
* 참고로 각 스트림은 단반향으로 흐른다.&#x20;

### 예제1

* `new FileOutputStream("temp/hello.dat")`
  * 파일에 데이터를 출력하는 스트림이다.&#x20;
  * 파일이 없으면 파일을 자동으로 만들고, 데이터를 해당 파일에 저장한다.&#x20;
  * 폴더를 만들지는 않기 때문에, 폴더는 미리 만들어두어야 한다.&#x20;
* `write()`&#x20;
  * byte 단위로 값을 출력한다. 여기서는 65,66,67을 출력했다.&#x20;
  * 참고로 ASCII 코드 집합에서 65는 A, 66은 B,  67은 C이다.&#x20;
* `new FileInputStream("temp/hello.dat")`
  * 파일에서 데이터를 읽어오는 스트림이다.&#x20;
* `read()`&#x20;
  * 파일에서 데이터를 byte 단위로 하나씩 읽어온다.&#x20;
  * 순서대로 65, 66, 67 을 읽어온다.&#x20;
  * 파일에 끝에 도달해서 더는 읽을 내용이 없다면 -1 을 반환한다.&#x20;
    * 파일의 끝 (EOF, End of File)&#x20;
* `close()`&#x20;
  * 파일에 접근하는 것은 자바 입장에서 외부 자원을 사용하는 것이다.&#x20;
  * 자바에서 내부 객체는 자동으로 GC 가 되지만, 외부 자원은 반드시 사용 후 닫아주어야 한다.&#x20;

```java
 package io.start;
 
 import java.io.FileInputStream;
 import java.io.FileOutputStream;
 import java.io.IOException;
 
 public class StreamStartMain1 {
      public static void main(String[] args) throws IOException {
         FileOutputStream fos = new FileOutputStream("temp/hello.dat");
         fos.write(65);
         fos.write(66);
         fos.write(67);
         fos.close();
         
         FileInputStream fis = new FileInputStream("temp/hello.dat");
         System.out.println(fis.read());
         System.out.println(fis.read());
         System.out.println(fis.read());
         System.out.println(fis.read());
         fis.close();
      }
}
```

### 예제2&#x20;

* 입력 스트림의 read() 메서드는 파일의 끝에 도달하면 -1 을 반환한다. 따라서 -1 을 반환할 때 까지 반복문을 사용하면 파일의 데이터를 모두 읽을 수 있다.&#x20;

```java
 package io.start;
 
 import java.io.FileInputStream;
 import java.io.FileOutputStream;
 import java.io.IOException;
 
 public class StreamStartMain2 {
      public static void main(String[] args) throws IOException {
         FileOutputStream fos = new FileOutputStream("temp/hello.dat");
         fos.write(65);
         fos.write(66);
         fos.write(67);
         fos.close();
         
         FileInputStream fis = new FileInputStream("temp/hello.dat");
         int data; 
         while((data = fis.read()) != -1) {
            System.out.println(data);
         }
         fis.close();
      }
}
```

## 2. 스트림 시작2&#x20;

### 예제3&#x20;

* 이번에는 byte 를 하나씩 다루는 것이 아니라, byte\[] 을 사용해서 데이터를 원하는 크기 만큼 더 편리하게 저장하고 읽는 방법을 살펴보자&#x20;
* 출력 스트림
  * `write(byte[])` : byte\[] 에 원하는 데이터를 담고 write() 에 전달하면 해당 데이터를 한 번에 출력할 수 있다.&#x20;
* 입력 스트림&#x20;
  * `read(byte[], offset, length)` : byte\[] 을 미리 만들어두고, 만들어둔 byte\[] 에 한 번에 데이터를 읽어올 수 있다.&#x20;
    * `byte[]` : 데이터가 읽혀지는 버퍼&#x20;
    * `offset` : 데이터 기록되는 byte\[] 인덱스 시작 위치&#x20;
    * `length` : 읽어올 byte 의 최대 길이&#x20;
    * `반환 값` : 버퍼에 읽은 총 바이트 수. 여기서는 3byte 를 읽었으므로 3이 반환된다. 스트림의 끝에 도달하여 더 이상 데이터가 없는 경우 -1 을 반환&#x20;
  * `read(byte[])` : offset, length 를 생략한 read(byte\[]) 메서드도 있다. 아래와 같이 default 값을 가진다.&#x20;
    * `offset` : 0
    * `length` : byte\[].length

```java
package io.start;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.uril.Arrays;
 
public class StreamStartMain2 {
      public static void main(String[] args) throws IOException {
         FileOutputStream fos = new FileOutputStream("temp/hello.dat");
         byte[] input = {65,66,67};
         fos.write(input); 
         fos.close();
         
         FileInputStream fis = new FileInputStream("temp/hello.dat");
         byte[] buffer = new byte[10];
         int readCount = fis.read(buffer, 0, 10);
         System.out.println("readCount = " + readCount);
         System.out.println(Arrays.toString(buffer));
         fis.close();
      }
}
```

### 예제 4&#x20;

* 모든 byte 한번에 읽기&#x20;

```java
package io.start;
 
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.uril.Arrays;
 
public class StreamStartMain2 {
      public static void main(String[] args) throws IOException {
         FileOutputStream fos = new FileOutputStream("temp/hello.dat");
         byte[] input = {65,66,67};
         fos.write(input); 
         fos.close();
         
         FileInputStream fis = new FileInputStream("temp/hello.dat");
         byte[] readBytes = fis.readAllBytes(); 
         System.out.println(Arrays.toString(buffer));
         fis.close();
      }
}
```

### 부분으로 나누어 읽기 vs 전체 읽기&#x20;

* `read(byte[], offset, length)`
  * 스트림의 내용을 부분적으로 읽거나, 읽은 내용을 처리하면서 스트림을 계속해서 읽어야 할 경우 적합하다.&#x20;
  * 메모리 사용량을 제어할 수 있다 .
  * 예를 들어, 파일이나 스트림에서 일정한 크기의 데이터를 반복적으로 읽어야 할 때 유용하다. 특히, 대용량 파일을 처리할 때, 한 번에 메모리에 로드하기보다는 이 메서드를 사용하여 파일을 조각조각 읽어들일 수 있다 .
  * 100MB 의 파일을 1MB 단위로 나누어 읽고 처리하는 방식을 사용하면 한 번에 최대 1MB 의 메모리만 사용한다.&#x20;
* `readAllBytes()`
  * 한 번의 호출로 모든 데이터를 읽을 수 있어 편리하다.&#x20;
  * 작은 파일이나 메모리에 모든 내용을 올려서 처리해햐 하는 경우에 적합하다.&#x20;
  * 메모리 사용량을 제어할 수 없다.&#x20;
  * 큰 파일의 경우  OutOfMemoryError 가 발생할 수 있다.&#x20;

## 3. InputStream, OutputStream&#x20;

<figure><img src="../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

* 현대의 컴퓨터는 대부분 byte 단위로 데이터를 주고받는다. 참고롤 bit 단위는 너무 작기 때문에, byte 단위를 기본으로 사용한다.&#x20;
* 이렇게 데이터를 주고 받는 것을 Input / Output(I/O) 라 한다.&#x20;
* 자바 내부에 있는 데이터를 외부에 있는 파일에 저장하거나, 네트워크를 통해 전송하거나 콘솔에 출력할 때 모두 byte 단위로 데이터를 주고 받는다.&#x20;
* 만약 파일, 네트워크, 콘솔 등에서 각각 데이터를 주고 받는 방식이 다르다면 상당히 불편할 것이다.&#x20;
* 또한, 파일에 저장하던 내용을 네트워크에 전달하거나 콘솔에 출력하도록 변경할 떄 너무 많은 코드를 변경해야 할 수 있다.&#x20;
* 이런 문제를 해결하기 위해 자바는 `InputStream`, `OutputStream` 이라는 기본 추상 클래스를 제공한다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (131).png" alt=""><figcaption></figcaption></figure>

* 스트림을 사용하면 파일을 사용하든, 소켓을 통해 네트워크를 사용하든 모두 일관된 방식으로 데이터를 주고 받을 수 있다. 그리고 수 많은 기본 구현 클래스들도 제공한다.&#x20;
* 물론 각각의 구현 클래스들은 자신에게 맞는 추가 기능도 함께 제공한다. &#x20;

### 메모리 스트림&#x20;

* `ByteArrayOutputStream`, `ByteArrayInputStream` 을 사용하면 메모리에 스트림을 쓰고 읽을 수 있다.&#x20;
* 이 클래스들은 `OutputStream`, `InputStream` 을 상속받았기 때문에 부모의 기능을 모두 사용할 수 있다.&#x20;
  * 코드를 보면 파일 입출력과 매우 비슷한 것을 알 수 있다.&#x20;
* 참고로 메모리에 어떤 데이터를 저장하고 읽을 때는 컬렉션이나 배열을 사용하면 되기 때문에, 이 기능은 잘 사용하지 않는다. 주로 스트림을 간단하게 테스트 하거나 스트림의 데이터를 확인하는 용도로 사용한다.&#x20;

```java
package io.start;

import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.util.Arrays;
 
public class ByteArrayStreamMain {
       public static void main(String[] args) throws IOException {
              byte[] input = {1, 2, 3};
                  
              // 메모리에 쓰기
              ByteArrayOutputStream baos = new ByteArrayOutputStream();
              baos.write(input);
                  
              // 메모리에서 읽기
              ByteArrayInputStream bais = new ByteArrayInputStream(baos.toByteArray());
              byte[] bytes = bais.readAllBytes();
              System.out.println(Arrays.toString(bytes));
       }
 }
```

### 콘솔 스트림&#x20;

* 우리가 자주 사용했던 `System.out` 이 사실은 `PrintStream` 이다.&#x20;
* 이 스트림은 `OutputStream` 을 상속받는다.&#x20;
* 이 스트림은 자바가 시작될 때 자동으로 만들어진다. 따라서 우리가 직접 생성하지 않는다.&#x20;
  * `write(byte[])` : `OutputStream` 부모 클래스가 제공하는 기능이다.&#x20;
  * `println(String)` : `PrintStream` 이 자체적으로 제공하는 추가 기능이다.&#x20;

<pre class="language-java"><code class="lang-java">package io.start;

import java.io.IOException;
import java.io.PrintStream;

import static java.nio.charset.StandardCharsets.UTF_8;

public class PrintStreamMain {
        public static void main(String[] args) throws IOException {
<strong>                PrintStream printStream = System.out;
</strong><strong>                
</strong>                byte[] bytes = "Hello!\n".getBytes(UTF_8);
                printStream.write(bytes);
                printStream.println("Print!");
        }
}
</code></pre>

## 4. 파일 입출력과 성능 최적화1 - 하나씩 쓰기&#x20;

```java
package io.buffered;

public class BufferedConst {
    public static final String FILE_NAME = "temp/buffered.dat";
    public static final int FILE_SIZE = 10 * 1024 * 1024; // 10MB
    public static final int BUFFER_SIZE = 8192; // 8KB
}
```

### 예제1 - 쓰기&#x20;

* 먼저 가장 단순한 `FileOutputStream` 의 `write()` 를 사용해 1byte 씩 파일을 저장해보자.&#x20;
* 그리고 10MB 파일을 만드는데 걸리는 시간을 확인해보자.&#x20;

```java
package io.buffered;

import java.io.FileOutputStream;
import java.io.IOException;

import static io.buffered.BufferedConst.FILE_NAME;
import static io.buffered.BufferedConst.FILE_SIZE;

public class CreateFileV1 {
    public static void main(String[] args) throws IOException {
        FileOutputStream fos = new FileOutputStream(FILE_NAME);
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < FILE_SIZE; i++) {
            fos.write(1);
        }
        fos.close();
        
        long endTime = System.currentTimeMillis();
        System.out.println("File created: " + FILE_NAME);
        System.out.println("File size: " + FILE_SIZE / 1024 / 1024 + "MB");
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
    }
}
```

* 실행하면 M2 맥북 프로 기준으로 약 14초 걸린다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (133).png" alt=""><figcaption></figcaption></figure>

### 예제1 - 읽기&#x20;

* fis.read() 를 사용해서 앞서 만든 파일에서 1byte 씩 데이터를 읽는다.&#x20;
* 파일의 크기가 10MB 이므로 fis.read() 메서드를 약 1000만번(10 \* 1024 \* 1024) 호출한다.&#x20;

<pre class="language-java"><code class="lang-java">package io.buffered;

import java.io.FileInputStream;
import java.io.IOException;
import static io.buffered.BufferedConst.FILE_NAME;

public class ReadFileV1 {
       public static void main(String[] args) throws IOException {
       FileInputStream fis = new FileInputStream(FILE_NAME);
       long startTime = System.currentTimeMillis();
<strong>
</strong><strong>       int fileSize = 0;
</strong>       int data;
    while ((data = fis.read()) != -1) {
           fileSize++;
    }
    fis.close();

    long endTime = System.currentTimeMillis();
    System.out.println("File name: " + FILE_NAME);
    System.out.println("File size: " + (fileSize / 1024 / 1024) + "MB");
    System.out.println("Time taken: " + (endTime - startTime) + "ms");
   }
}
</code></pre>

* 실행 결과를 보는데 상당히 많은 시간이 소요된다.&#x20;

<figure><img src="../../../../.gitbook/assets/image (134).png" alt=""><figcaption></figcaption></figure>

### 정리&#x20;

* 10MB 파일 하나 쓰는데 14초, 읽는데 5초라는 매우 오랜 시간이 걸렸다.&#x20;
* 이렇게 오래 걸린 이유는 자바에서 1byte 씩 디스크에  데이터를 전달하기 때문이다. 디스크는 1byte 의 데이터를 받아서 1byte 의 데이터를 쓴다. 이 과정을 1000만번 반복하는 것이다.&#x20;
* 더 자세하게 설명하면 다음 2가지 이유로 느려진다.&#x20;
  * `write()` 나 `read()` 를 호출할 때마다 OS 에 시스템 콜을 통해 파일을 읽거나 쓰는 명령어를 전달한다. 이러한 시스템 콜은 상대적으로 무거운 작업이다.&#x20;
  * HDD, SSD 같은 장치들도 하나의 데이터를 읽고 쓸 때마다 필요한 시간이 있다. HDD 의 경우 더욱 느린데, 물리적으로 디스크 회전 시간이 필요하기 때문
  * 이러한 무거운 작업을 무려 1000만번 반복한다.&#x20;

## 5. 파일 입출력과 성능 최적화2 - 버퍼 활용

### 예제2 - 쓰기&#x20;

* 데이터를 먼저 buffer 라는 byte\[] 에 담아둔다.&#x20;
  * 이렇게 데이터를 모아서 전달하거나 모아서 전달받는 용도로 사용하는 것을 버퍼라 한다.&#x20;
* 여기서는  BUFFER\_SIZE 만큼 데이터를 모아서 write() 를 호출한다.&#x20;
  * 예를 들어 BUFFER\_SIZE 가 10이라면 10만큼 모이면 write() 를 호출해서 10byte 를 한번에 스트림에 전달한다.&#x20;

```java
package io.buffered;

import java.io.FileOutputStream;
import java.io.IOException;
import static io.buffered.BufferedConst.*;

public class CreateFileV2 {
    public static void main(String[] args) throws IOException {
        FileOutputStream fos = new FileOutputStream(FILE_NAME);
        long startTime = System.currentTimeMillis();
        
        byte[] buffer = new byte[BUFFER_SIZE];
        int bufferIndex = 0;
        
        for (int i = 0; i < FILE_SIZE; i++) {
            buffer[bufferIndex++] = 1;
        
            // 버퍼가 가득 차면 쓰고, 버퍼를 비운다.
            if (bufferIndex == BUFFER_SIZE) {
                fos.write(buffer);
                bufferIndex = 0;
            }
        }
        
        // 끝 부분에 오면 버퍼가 가득차지 않고 남아있을 수 있다. 버퍼에 남은 부분 쓰기
        if (bufferIndex > 0) {
             fos.write(buffer, 0, bufferIndex);
        }
        
        fos.close();
        
        long endTime = System.currentTimeMillis();
        System.out.println("File created: " + FILE_NAME);
        System.out.println("File size: " + FILE_SIZE / 1024 / 1024 + "MB");
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
    }
}
```

* 실행 결과를 보면 이전 예제의 쓰기 결과인 14초 보다 약 1000배 정도 빠른 것을 확인할 수 있다.

<figure><img src="../../../../.gitbook/assets/image (135).png" alt=""><figcaption></figcaption></figure>

### 버퍼 크기에 따른 쓰기 성능&#x20;

#### BUFFER\_SIZE 에 따른 쓰기 성능&#x20;

* 1  : 14368ms&#x20;
* &#x20;2 : 7474ms&#x20;
* &#x20;3 : 4829ms&#x20;
* &#x20;10 : 1692ms&#x20;
* &#x20;100 : 180ms&#x20;
* &#x20;1000 : 28ms&#x20;
* &#x20;2000 : 23ms&#x20;
* &#x20;4000 : 16ms&#x20;
* &#x20;8000 : 13ms&#x20;
* &#x20;80000 : 12ms

#### BUFFER 와 성능에 대한 이야기&#x20;

* 많은 데이터를 한 번에 전달하면 성능을 최적화 할 수 있다. 이렇게 되면 시스템 콜도 줄어들고, HDD, SSD 같은 장치 들의 작동 횟수도 줄어든다. 예를 들어 버퍼의 크기를 1 -> 2 로 변경하면 시스템 콜 횟수는 절반으로 줄어든다.&#x20;
* 그런데 버퍼의 크기가 커진다고 해서 속도가 계속 줄어들지는 않는다.&#x20;
  * 왜냐하면, **디스크나 파일 시스템에서 데이터를 읽고쓰는 기본 단위가 보통 4KB 또는 8KB 이기 때문이다.**&#x20;
* 결국 버퍼의 많은 데이터를 담아서 보내도 디스크나 파일 시스템에서 해당 단위로 나누어 저장하기 때문에, 효율에는 한계가 있다.&#x20;
* **따라서 버퍼의 크기는 보통 4KB, 8KB 정도로  잡는 것이 효율적이다.**\


### **예제2 - 읽기**&#x20;

<pre class="language-java"><code class="lang-java">package io.buffered;

import java.io.FileInputStream;
import java.io.IOException;

import static io.buffered.BufferedConst.BUFFER_SIZE;
import static io.buffered.BufferedConst.FILE_NAME;
    
public class ReadFileV2 {
<strong>    public static void main(String[] args) throws IOException {
</strong>        FileInputStream fis = new FileInputStream(FILE_NAME);
        long startTime = System.currentTimeMillis();
        
        byte[] buffer = new byte[BUFFER_SIZE];
        int fileSize = 0;
        int size;
        while ((size = fis.read(buffer)) != -1) {
            fileSize += size;
        }
        fis.close();
        
        long endTime = System.currentTimeMillis();
        System.out.println("File name: " + FILE_NAME);
        System.out.println("File size: " + (fileSize / 1024 / 1024) + "MB");
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
   }
}
</code></pre>

* 실행 결과를 보면 읽기 예제 또한 약 1000배 정도 빠른 것을 확인할 수 있다.

<figure><img src="../../../../.gitbook/assets/image (136).png" alt=""><figcaption></figcaption></figure>

## 6. 파일 입출력과 성능 최적화3 - Buffered 스트림 쓰기&#x20;

* `BufferedOutputStream` 은 버퍼 기능을 내부에서 대신 처리해준다. 따라서 단순한 코드를 유지하면서 버퍼를 사용하는 이점을 함께 누릴 수 있다.&#x20;

### 예제3 - 쓰기

* `BufferedOutputStream` 내부에서 단순히 버퍼 기능만 제공한다. 따라서 반드시 대상 `OutputStream` 이 있어야 한다.&#x20;
* 여기서는 `FileOutputStream` 객체를 생성자에 전달한다.&#x20;
* 추가로 사용할 버퍼의 크기도 함께 전달할 수 있다.&#x20;
* 코드를 보면 버퍼를 위한 `byte[]` 을 직접 다루지 않고, 마치 예제1 과 같이 단순하게 코드를 작성할 수 있다.&#x20;
* 만약 버퍼에 데이터가 남아있는 상태로 `close()` 가 호출되면 어떻게 될까?&#x20;
  * `BufferOutputStream` 을 `close()` 로 닫으면 먼저 내부에서 `flush()` 를 호출한다. 따라서 버퍼에 남아 있는 데이터를 모두 전달하고 비운다.&#x20;
  * 따라서 `close()` 를 호출해도 남은 데이터를 안전하게 저장할 수 있다.&#x20;
  * 버퍼가 비워지고 나면 `close()` 로 `BufferOutputStream` 의 자원을 정리한다.&#x20;
  * 그리고 나서 다음 연결된 스트림의 `close()` 를 호출한다.&#x20;
    * 여기서는 `FileOutputStream` 의 자원이 정리된다.&#x20;
  * **여기서 핵심은 `close()` 를 호출하면 `close()` 가 연쇄적으로 호출된다는 점이다. 따라서 마지막에 연결한 `BufferedOutputStream` 만 닫아주면 된다.**&#x20;

```java
package io.buffered;

import java.io.BufferedOutputStream;
import java.io.FileOutputStream;
import java.io.IOException;

import static io.buffered.BufferedConst.*;

public class CreateFileV3 {
    public static void main(String[] args) throws IOException {
        FileOutputStream fos = new FileOutputStream(FILE_NAME);
        BufferedOutputStream bos = new BufferedOutputStream(fos, BUFFER_SIZE);
        long startTime = System.currentTimeMillis();
        
        for (int i = 0; i < FILE_SIZE; i++) {
            bos.write(1);
        }
        bos.close();
        
        long endTime = System.currentTimeMillis();
        System.out.println("File created: " + FILE_NAME);
        System.out.println("File size: " + FILE_SIZE / 1024 / 1024 + "MB");
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
    }
}
```

<figure><img src="../../../../.gitbook/assets/image (137).png" alt=""><figcaption></figcaption></figure>

### 기본 스트림, 보조 스트림&#x20;

* **`FileOutputStream` 과 같이 단독으로 사용할 수 있는 스트림을 기본 스트림이라 한다.**&#x20;
* **`BufferedOutputStream` 과 같이 단독으로 사용할 수 없고, 보조 기능을 제공하는 스트림을 보조 스트림이라고 한다.**&#x20;
* `BufferedOutputStream` 은 `FileOutputStream` 에 버퍼라는 보조 기능을 제공한다.&#x20;
* `BufferedOutputStream` 의 생성자를 보면 알겠지만, 반드시 `FileOutputStream` 같은 대상 `OutputStream` 이 있어야 한다.&#x20;

### 예제3 - 읽기

```java
package io.buffered;

import java.io.BufferedInputStream;
import java.io.FileInputStream;
import java.io.IOException;

import static io.buffered.BufferedConst.BUFFER_SIZE;
import static io.buffered.BufferedConst.FILE_NAME;

public class ReadFileV3 {
    public static void main(String[] args) throws IOException {
        FileInputStream fis = new FileInputStream(FILE_NAME);
        BufferedInputStream bis = new BufferedInputStream(fis, BUFFER_SIZE);
        long startTime = System.currentTimeMillis();
        
        int fileSize = 0;
        int data;
        while ((data = bis.read()) != -1) {
            fileSize++;
        }
        bis.close();
        
        long endTime = System.currentTimeMillis();
        System.out.println("File name: " + FILE_NAME);
        System.out.println("File size: " + (fileSize / 1024 / 1024) + "MB");
        System.out.println("Time taken: " + (endTime - startTime) + "ms");
   }
}
```

<figure><img src="../../../../.gitbook/assets/image (138).png" alt=""><figcaption></figcaption></figure>

### 버퍼를 직접 다루는 것 보다 BufferedXxx 의 성능이 떨어지는 이유&#x20;

* 방법에 따른 쓰기 시간&#x20;
  * 예제1 쓰기: 14000ms (14초)&#x20;
  * 예제2 쓰기: 14ms (버퍼 직접 다룸)&#x20;
  * 예제3 쓰기: 102ms (BufferedXxx)
* 예제2는 버퍼를 직접 다루는 것이고, 예제3은 `BufferedXxx` 라는 클래스가 대신 버퍼를 처리해준다. 버퍼를 사용하는 것은 같기 때문에, 결과적으로 예제2와 예제3은 비슷한 성능이 나와야 한다. 그런데 왜 예제2가 더 빠른 것일까?&#x20;
* **이 이유는 동기화 때문이다.**&#x20;
  * `BufferedOutputStream` 을 포함한 `BufferedXxx` 클래스는 모두 동기화 처리가 되어 있다.&#x20;
  * 이번 예제의 문제는 1byte 씩 저장해서 총 10MB 를 처리해야 하는데, 이렇게 하려면 `write()` 를 약 1000만 번 호출해야 한다. (10  \* 1024 \* 1024)&#x20;
  * 결과적으로 락을 걸고 푸는 코드도 1000만 번 호출된다는 뜻이다.&#x20;

```java
@Override
public void write(int b) throws IOException {
    if (lock != null) {
        lock.lock();
        try {
            implWrite(b);
        } finally {
            lock.unlock();
        }
    } else {
        synchronized (this) {
            implWrite(b);
        }
    } 
}
```

#### `BufferdXxx` 클래스의 특징&#x20;

* `BufferdXxx` 클래스는 자바 초창기에 만들어진 클래스인데, 처음부터 멀티 스레드를 고려해서 만든 클래스이다.&#x20;
* 따라서 멀티 스레드에 안전하지만 락을 걸고 푸는 동기화 코드로 인해 성능이 약간 저하될 수 있다.&#x20;
* 하지만 싱글 스레드 상황에서는 동기화 락이 필요하지 않기 때문에, 직접 버퍼를 다룰 때와 비교해서 성능이 떨어진다.&#x20;
* 일반적인 상황이라면 이 정도 성능은 크게 문제가 되지 않기 때문에, 싱글 스레드여도 `BufferedXxx` 를 사용하면 충분하다.&#x20;
* 물론 매우 큰 데이터를 다루어야 하고, 성능 최적화가 중요하다면 예제 2와 같이 직접 버퍼를 다루는 방법을 고려하자.&#x20;
* 아쉽게도 동기화 락이 없는 `BufferedXxx` 클래스는 없다, 꼭 필요한 상황이라면, `BufferedXxx` 를 참고해서 동기화 코드를 제거한 클래스를 직접 만들어서 사용하면 된다.&#x20;
