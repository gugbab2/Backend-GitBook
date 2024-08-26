# String 클래스로 살펴보는 불변객체

### Immutable(불변)

* String 은 immutable 한 객체이다.&#x20;
* String 객체를 더한다고 해서 객체가 변하는 것이 아닌, 새로운 객체가 생겨나고 기존의 객체는 버려지는 것이다.&#x20;
* 만약, String 객체를 계속적으로 더하면 새로운 String 객체가 생겨나고 버려지는 프로세스의 반복이다. \
  (계속적인 Garbage를 만들어 GC 가 일하게 되므로 메모리가 낭비된다..)

> #### new String("") 형식이 메모리 낭비가 되는 이유
>
> * Java 는 문자열 리터럴을 효율적으로 관리하기 위해 **String 상수 풀(String constant pool)**이라는 매커니즘을 사용한다.&#x20;
> * 이 풀은 동일한 문자열 리터럴을 하나만 저장하고, 해당 리터럴을 참조하는 모든 변수가 동일한 객체를 공유하도록 한다.&#x20;
>
> <pre class="language-java"><code class="lang-java"><strong>// 아래 변수는 같은 객체를 참조하게 된다. 
> </strong><strong>// 문자열 상수풀은 하나의 문자열 리터럴만 저장하게 된다. 
> </strong><strong>String str1 = "Hello";
> </strong>String str2 = "Hello";
> </code></pre>
>
> * 그러나 new String("") 형식을 사용하면 동일한 문자열 리터럴 객체가 상수풀에 존재하더라도, 새로운 객체를 생성한다. (상수풀을 사용하지 않는 것이다)
> * **때문에, new String("") 의 형식을 사용하는 것은 메모리를 낭비하는 것이다.**
