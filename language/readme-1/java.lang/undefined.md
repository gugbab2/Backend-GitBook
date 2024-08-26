# 숫자를 처리하는 클래스

## 숫자를 처리하는 클래스 기본

* 기본적으로 숫자의 자료형은 힙메모리가 아닌, 스택메모리에 저장이 된다.\
  \-> 계산할 때 보다 빠른 처리가 가능하다.
* 하지만, 이러한 기본 자료형 또한 참조 자료형으로 처리해야 할 필요가 있을 수 있다.\
  때문에, 기본자료형을 참조자료형으로 선언한 클래스가 있다.
* Character 를 제외하고서 나머지 7개의 기본자료형의 앞글자만 대문자로 바꾸었다고 생각하면 된다.
* Character, Boolean 을 제외한 숫자를 처리하는 클래스들은 모두 Number 라는 추상클래스를 확장한다.
* **겉보기에는 모두 참조자료형이지만 기본자료형처럼 사용이 가능하다.**\
  **-> 자바 컴파일러에서 자동으로 형변환을 해주기 때문이다!**

## 숫자를 처리하는 클래스 사용

* Charactor 클래스를 제외하고서는 .parse타입이름() / .valueOf() 두가지의 특별한 메서드를 제공한다.
* .parse타입이름() : 기본자료형을 반환
* .valueOf() : 참조자료형을 반환
* **참조자료형 중 더하기 연산이 가능한 것은 String 뿐이다.**\
  \-> 숫자를 처리하는 클래스들은 기본자료형처럼 사용이 가능하기 때문에, new 를 사용하지 않아도 값을 할당할 수 있다.

```java
public void numberTypeCheck(){
    String value1 = "3";
    String value2 = "5";
    byte byte1 = Byte.parseByte(value1);
    byte byte2 = Byte.parseByte(value2);
    System.out.println(byte1 + byte2);

    Integer refInt1 = Integer.valueOf(value1);
    Integer refInt2 = Integer.valueOf(value2);
    System.out.println(refInt1 + refInt2 + "7");    //87
}
```

## 숫자를 처리하는 클래스를 만든 이유

* 매개 변수를 참조 자료형으로 받는 메소드를 처리하기 위해서
* 제네릭과 같이 기본자료형을 사용하지 않는 기능을 사용하기 위해서
* MIN\_VALUE, MAX\_VALUE 와 같이 클래스에 선언된 상수 값을 사용하기 위해서
* 문자열을 숫자, 숫자를 문자열로 쉽게 변환하고, 진수 변환을 쉽게 하기 위해서
