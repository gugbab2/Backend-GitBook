# 다형성(Pilymorphism)

### 다형성

* 다형성 : 형태가 다양하다.(사전적 의미)
* 자바에서 다형성 : 형 변환을 하더라도 실제 호출되는 것은 원래 객체에 있는 메소드가 호출되는 것을 의미.\
  \-> 아래 코드에서 예제를 확인할 수 있다.&#x20;

```java
public class Main {
    public static void main(String[] args) {

        ParentCasting parentCasting = new ParentCasting();
        ParentCasting parentCasting1 = new ChildCasting();

        parentCasting.printName();    //ParentCasting.printName()
        parentCasting1.printName();   //ChildCasting.printName()
        
    }
}
```
