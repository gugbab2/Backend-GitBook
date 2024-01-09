# 팩토리 메서드 패턴과 Java Reflection

## 1. 팩토리 메서드 패턴?

* 공장에서 만들어지는 제품을 생각하자\
  \-> 복잡한 생산 과정을 숨기고, 완성된 인스턴스만 반환한다. \
  \-> 객체가 생성되는 과정을 숨기고 객체를 리턴하는 것을 팩터리 메서드 패턴이라 부른다.&#x20;
* 다음과 같이 new 를 통해서 객체를 만드는 것이 아닌, 메서드를 통해서 객체를 만들 때 우리는 펙토리 메서트 패턴이라 부른다.&#x20;

```java
public class BusFactory {

    public Bus getBus(){
        return new Bus();
    }
}
```

```java
public class BusFactoryMain {
    public static void main(String[] args) {
        BusFactory bf1 = BusFactory.getInstance();
        BusFactory bf2 = BusFactory.getInstance();

        Bus bus1 = bf1.getBus();
        Bus bus2 = bf2.getBus();
    }
}
```

