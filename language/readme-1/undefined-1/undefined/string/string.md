# String 구조

## String 클래스 구조

<pre class="language-java"><code class="lang-java"><strong>public final class String extends Object 
</strong> implements Serializable, Comparable&#x3C;String>, CharSequence 
</code></pre>

* public final : 외부로 공개되어 있지만, 더 이상 확장할 수 없는 클래스(상속x / 불변객체)
* Serializable : 구현해야 할 메서드가 없는 특이한 인터페이스이다.\
  \-> 해당 인터페이스를 선언만 해놓으면, **해당 객체를 파일로 저장하거나, 다른 서버에 전송 가능한 상태가 된다.**
* Comparable : 해당 인터페이스는 compareTo() 라는 메서드 하나만 선언되어 있다.\
  해당 메서드의 리턴타입은 int 로 같으면 0, 순서상으로 앞에 있으면 -1, 뒤에 있으면 1을 리턴한다.\
  \-> 순서를 처리할 때 유용한 메서드이다.
* CharSequence : 해당 클래스가 문자열을 다루기 위한 클래스라는 것을 명시적으로 나타내는데 사용된다.\
  \-> **StringBuilder, StringBuffer 클래스도 CharSequence 인터페이스를 구현해 놓았다.**
