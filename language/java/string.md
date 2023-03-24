# String

## String 클래스 구조

<pre class="language-java"><code class="lang-java"><strong>public final class String extends Object 
</strong> implements Serializable, Comparable&#x3C;String>, CharSequence 
</code></pre>

* public final : 외부로 공개되어 있지만, 더이상 확장할 수 없는 클래스(상속x)
* Serializable : 구현해야 할 메서드가 없는 특이한 인터페이스이다. \
  \-> 해당 인터페이스를 선언만 해놓으면, 해당 객체를 파일로 저장하거나, 다른 서버에 전송 가능한 상태가 된다.
* Comparable\<String> :&#x20;
