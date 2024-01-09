# Object.toString()

### 1. 오버라이딩 하지 않은, toString() 은 주소값일까?

* Java 는 주소라는 개념을 가지고 있지 않다!
* **참조라는 개념만을 가지고 있다!**\
  **-> 오버라이딩 하지 않은 toString() 에서 나오는 값은 참조값이라고 보는 시점이 더 맞다**

## 2. Object.toString() 이 자동적으로 호출되는 경우

* CASE
  * _**System.out.println() 메소드에 매개 변수로 들어가는 경우**_
  * _**객체에 대하여 더하기 연산을 하는 경우**_\
    **-> 더하기 연산은, 참조자료형 중 String 만 할 수 있기 때문에, String 이 아닌 참조변수끼리는 서로 더할 수 없다.**\
    **-> String + String 이 아닌 참조변수의 형식으로 더하기 연산이 가능하다.**&#x20;
* default toString 형식
  * getClass().getName() + '@' + Integer.toHexString(hashCode())\
    \-> Full Package + '@' + 객체의 해시코드 값

```java
public class ToString {
    public void toStringMethod(Object obj){
        System.out.println(obj);             //org.example.chapter12.ToString@ea30797
        System.out.println(obj.toString());  //org.example.chapter12.ToString@ea30797
        System.out.println("plus + " + obj); //plus + org.example.chapter12.ToString@ea30797
    }
}
```

## 3. Object.toString() 는 대부분 Overriding 해서 사용해야 한다.

* 사용할 때?
  * DTO(Data Transfore Object) 를 사용할 때는 기본적으로 toString() 메서드를 오버라이딩 해야 내용 확인이 쉽다

```java
public void toStringMethod(Object obj){
    System.out.println(obj);                //ToString Class
    System.out.println(obj.toString());     //ToString Class
    System.out.println("plus + " + obj);    //plus + ToString Class
}

public String toString(){
    return "ToString Class";
}
```
