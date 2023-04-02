# 메소드를 제네릭하기 선언하기

* 이전의 제네릭 사용 방법에 큰 단점 중 하나는 매개변수로 사용된 객체에 값을 추가할 수 없다는 것이다.&#x20;
* 이 문제를 해결하기 위해서, 리턴타입 앞에, <>로 제네릭 타입을 선언할 수 있다 다음 코드를 보자.
  * 메서드 리턴타입 앞에 <> 로 제네릭 타입을 선언함으로 매개변수로 사용된 객체에 제네릭 타입을 지정해 줄 수 있다.&#x20;

```java
public class WildcardSample {

    public void callWildcardMethod(){
        WildcardGeneric<String> wildcard = new WildcardGeneric<>();
        wildcard.setWildcard("A");
        wildcardStringMethod(wildcard);
    }

    public <T> void wildcardStringMethod(WildcardGeneric<T> c){
        String value = c.getWildcard();
        System.out.println("value = " + value);
    }
}
```
