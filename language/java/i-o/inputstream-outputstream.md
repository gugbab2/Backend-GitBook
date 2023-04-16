# InputStream, OutputStream

## InputStream, OutputStream 은 자바 스트림의 부모들이다.

* 자바의 I/O 는 기본적으로 InputStream 과 OutputStream 이라는 추상클래스를 통해서 제공된다. \
  **-> 어떤 대상의 데이터를 읽을 때, InputStream / 쓸 때, OutputStream**\
  **-> 해당 클래스는 Closeable 이라는 인터페이스를 확장하는데, close() 메서드만 가지고 있다.** \
  **-> 이 뜻은, 해당 객체를 사용하면 반드시 자원을 반납하라는 의미랑 같다.**&#x20;
* **InputStream** **메서드**&#x20;
  * int available() : 스트림에서 중단없이 읽을 수 있는 바이트의 개수를 리턴한다.&#x20;
  * void mark(int readlimit) : 스트림의 현재 위치를 표시해둔다. 매개변수는 표시해둔 자리의 최대 유효길이 이다.&#x20;
  * void reset() : 현재 위치를 mark() 가 호출되었던 위치로 되돌린다.&#x20;
  * boolean markSupported() : mark() reset() 메서드가 호출 가능한지 확인
  * **abstract int read() : 스트림에서 다음 바이트를 읽는다.**&#x20;
  * **int read(byte\[] b) : 매개변수로 넘어온 배열에 데이터를 담는다.**&#x20;
  * **int read(byte\[] b, int off, int len) : 데이터를 담는 위치, 길이를 설정하고 매개변수로 넘어온 배열에 데이터를 담는다.**&#x20;
  * long skip(long n) : 매개변수 만금 데이터를 스킵한다.&#x20;
  * **void close() : 자원을 반납한다.** \
    **-> 반드시 호출해야 한다!**
* **OutputStream 메서드**\
  **-> OutputStream 은 Flushable 인터페이스를 추가적으로 확장하는데, flush() 메서드만 가지고 있다.** \
  **-> 일반적으로 데이터를 쓸때 매번 쓰기 작업을 요청할 때마다 저장하면 효율이 않좋다 ..**\
  **-> 그래서 대부분 Buffer 를 가지고 데이터를 쌓아두었다가 어느 정도 차게 되면 한번에 저장한다.** \
  **-> flush() 메서드는 "현재 버퍼에 있는 내용을 기다리지 말고 무조건 저장해!" 라는 의미로 생각하면 된다.**&#x20;
  * **void write(byte\[] b) : 매개변수로 받은 바이트 배열을 저장한다.**&#x20;
  * **void write(byte\[] b, int off, int len) : 매개변수로 받은 바이트 배열의 특정 구간만을 저장한다.**&#x20;
  * **abstract void write(int b) : 매개변수로 받은 바이트를 저장한다. 타입은 바이트이지만 실제는 바이트로 저장된다.**&#x20;
  * **void flush() : 버퍼에 저장하겨고 대기하고 있는 데이터를 강제로 저장한다.**&#x20;
  * **void close() : 자원을 반납한다.** \
    **-> 반드시 호출해야 한다!**
