# 제네릭에 ?(와일드 카드)

## 제네릭에 ?가 있는 것은 무엇인가?

* wildcardStringMethod(); 의 매개변수는  WildcardGeneric\<String> 밖에 오지 못한다..

```java
public class WildcardGeneric<W> {
    W wildcard;

    public void setWildcard(W wildcard) {
        this.wildcard = wildcard;
    }

    public W getWildcard(){
        return this.wildcard;
    }
}

public class WildcardSample {

    public void callWildcardMethod(){
        WildcardGeneric<String> wildcard = new WildcardGeneric<>();
        wildcard.setWildcard("A");
        wildcardStringMethod(wildcard);
    }

    public void wildcardStringMethod(WildcardGeneric<String> c){
        String value = c.getWildcard();
        System.out.println("value = " + value);
    }
}
```

* 이런경우 메서드의 매개변수의 타입을 WildcardGeneric\<?> 와 같이 변경해 줄 수 있다.
  * 하지만 이런식으로 타입을 명확하게 하지 않는 경우, 메서드의 입장에서 어떤 객체가 들어올지 모르기 때문에, 객체의 최상위 부모인 Object 로 처리해야만 한다.&#x20;

```java
public void wildcardStringMethod(WildcardGeneric<?> c){
    Object value = c.getWildcard();
    System.out.println("value = " + value);
}
```

## 제네릭 선언에 사용하는 타입의 범위를 지정

* 본래 제네릭은 <>안에 어떤 타입이라도 상관 없다고 했지만, ? 사용을 통해서 사용하는 타입을 제한할 수 있다.
* ? 대신 ? extends "타입" 의 형태로 사용하면 된다.&#x20;
* boundWildcardMethod(WildcardGeneric\<? extends Car> c) 를 통해서 Object 객체를 포함한 타입을 제한할 수 있다.&#x20;

```java
public class CarWildcardSample {
    public void callBoundedWildcardMethod(){
        WildcardGeneric<Car> wildcard = new WildcardGeneric<>();
        WildcardGeneric<Car> wildcard2 = new WildcardGeneric<>();

        wildcard2.setWildcard(new Car("car"));
        wildcard.setWildcard(new Bus("bus"));

        boundWildcardMethod(wildcard2);
        boundWildcardMethod(wildcard);
    }

    public void boundWildcardMethod(WildcardGeneric<? extends Car> c){
        Car value = c.getWildcard();
        System.out.println("name = " + value.name);
    }
}
```
