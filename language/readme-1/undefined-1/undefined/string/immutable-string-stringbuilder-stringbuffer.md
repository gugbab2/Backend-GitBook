# StringBuilder, StringBuffer

### Immutable 한 String 의 단점을 해결한 문자열 클래스&#x20;

* 위의 문제점을 해결하기 위해서 나온 객체가 StringBuilder, StringBuffer 이다.\
  \-> 두 클래스에서 제공하는 메서드는 동일하다.
* **하지만, StringBuffer 는 Thread Safe 하고 StringBuilder 는 Thread Safe 하지 않다...**\
  **-> StringBuffer 는 synchronized 블럭으로 주요 데이터 처리 부분을 감싸 주었고, StringBuilder 는 synchronized 를 사용하지 않았다.. 그 차이이다! (속도는 StringBuilder 가 빠르다)**
* StringBuilder, StringBuffer 두 객체는 **문자열을 더할 때, '+' 가 아닌 .append() 메서드를 사용**해야 한다.
