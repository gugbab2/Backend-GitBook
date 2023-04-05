# Object.toString()

### Object.toString() 이 자동적으로 호출되는 경우

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

### Object.toString() 는 대부분 Overriding 해서 사용해야 한다.

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
