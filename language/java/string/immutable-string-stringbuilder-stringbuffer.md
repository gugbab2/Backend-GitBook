# immutable 한 String 의 단점을 보완한 클래스 StringBuilder, StringBuffer

* immutable : '불변' 이라는 의미로 String 은 immutable 한 객체이다. \
  \-> **String 객체를 더한다고 해서 객체가 변하는 것이 아닌, 새로운 객체가 생겨나고 기존의 객체는 버려지는 것이다.** \
  \-> **만약, String 객체를 계속적으로 더하면 새로운 String 객체가 생겨나고 버려지는 프로세스의 반복이다..**\
  **(계속적인 Garbage를 만들어 GC 가 일하게 되므로 메모리가 낭비된다..)**
* 위의 문제점을 해결하기 위해서 나온 객체가 StringBuilder, StringBuffer 이다. \
  \-> 두 클래스에서 제공하는 메서드는 동일하다. \
  \-> **하지만,  StringBuffer 는 Thread Safe 하고 StringBuilder 는 Thread Safe 하지 않다...**\
  \-> **속도는 StringBuilder 가 빠르다!**
* StringBuilder, StringBuffer 두 객체는 **문자열을 더할 때, '+' 가 아닌 .append() 메서드를 사용**해야 한다.&#x20;
