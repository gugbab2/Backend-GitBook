# Reader, Writer

* 앞선 Stream 은 바이트를 처리하기 위함이고 Reader, Writer 는 문자열을 처리하기 위한 클래스이다.&#x20;

## Reader

```java
public abstract class Reader 
extends Object
implements Readble, Closeable 
```

* &#x20;메서드는 다음과 같다
  * boolean ready() : 읽을 준비가 되어 있는지 확인
  * void mark(int readAheadLimit) : Reader 의 현재 위치를 표시해둔다. 매개변수로 넘긴 Int 값은 표시해 둔 자리의 최대 유효길이이다.&#x20;
  * void reset() : 현재 위치를 mark() 메서드가 호출된 위치로 옮긴다.&#x20;
  * boolean markSupported() : mark() reset() 메서드가 호출 가능한 상태인지 확인
  * int read() : 하나의 char 을 읽는다.&#x20;
  * int read(char\[] cbuf) : 매개 변수로 넘어온 char 배열에 데이터를 담는다.&#x20;
  * abstract int read(char\[] cbuf, int off, int len) : 매개 변수로 넘어온 char 배열에 특정 위치 부터 일정 길이 만큼만 Char 배열에 담는다.&#x20;
  * int read(CharBuffer target) :  매개 변수로 넘어온 CharBuffer에 데이터를 담는다.&#x20;
  * long skip(long n ) : 매개 변수로 넘어온 개수 만큼의 char 을 건너뛴다.
  * abstract void close() : 자원 반납&#x20;

## Writer

```java
public abstract class Writer
extends Object
implements Appendable, Closeable, Flushable
```

* Appendable 클래스는 각종 문자열을 추가하기 위해서 Jav5 부터 추가되었다.&#x20;
* 메서드는 다음과 같다
  * Writer append(char c) : 매개변수로 넘어온 데이터를 추가한다.&#x20;
  * Writer append(CharSequence csq) : 매개변수로 넘어온 CharSequence 를 추가한다. \
    \-> **CharSequence 를 확장한 클래스는 String, StringBilder, StringBuffer**
  * Writer append(CharSequence csq, int start, int end) : 매개변수로 넘어온 CharSequence 를 특정 위치에 추가한다. &#x20;
  * void write(char\[] cbuf) : 매개변수로 넘어온 char\[] 을 추가한다. &#x20;
  * abstract void write(char\[] cbuf, int off, int len) : 매개변수로 넘어온 char\[] 을 특정위치에 추가한다. &#x20;
  * void write(int c) : 매개변수로 넘어온 Int 값에 매핑되는 문자를 추가한다.
  * void write(String str) : 매개변수로 넘어온 문자열을 추가한다.&#x20;
  * void write(String str, int off, int len) : 매개변수로 넘어온 문자열을 특정위치에 추가한다.
  * abstract void flush() : 버퍼에 있는 데이터를 강제로 대상 리소스에 쓰도록 한다.&#x20;
  * abstract void close() : 자원 반납&#x20;
