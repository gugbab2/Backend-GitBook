# 참조자료형의 형변환

### 참조자료형의 형변환

* 참조 자료형은 자식 클래스의 타입을 부모 클래스의 타입으로 형 변환하면, 부모클래스에서 호출할 수 있는 메소드들은 자식 클래스에서도 호출할 수 있으므로 전혀 문제가 되지 않는다. (상속관계이기 때문에!)\
  \-> 따라서 형 변환을 명시적으로 해줄 필요가 없다.
* 아래코드와 같이 부모객체를 자식객체에서 사용하려는 경우 런타임 에러가 생긴다.\
  \-> 자식객체에서 사용하려는 객체가 실제로 부모객체인 경우 사용할 수 가 없다.(ClassCastException)\
  \-> 보통의 자식객체를 부모 객체보다 **확장된 개념이다.**&#x20;

```java
public class InheritanceCasting {
    public void objectCast(){
        ParentCasting parentCasting = new ParentCasting();
        ChildCasting childCasting = new ChildCasting();

        ParentCasting parentCasting1 = childCasting;
        ChildCasting childCasting1 = (ChildCasting)parentCasting;
    }
}
```

* 하지만 아래 코드와 같이 **자식객체에서 사용하려는 객체의 실제 모습이 자식객체라면 문제없이 사용할 수 있다.**\
  **-> 실제 인스턴스가 어떠한 값을 가지고 있는지가 중요하다.**

```java
public class InheritanceCasting2 {
    public void objectCast(){
        ChildCasting childCasting = new ChildCasting();
        ParentCasting parentCasting = childCasting;
        ChildCasting childCasting1 = (ChildCasting) parentCasting;
    }
}
```

### 참조자료형의 형변환 정리

* 참조자료형도 형 변환이 가능하다.
* 자식 타입의 객체를 부모 타입으로 형 변환 하는 것은 자동으로 된다.
* 부모 타입의 객체를 자식 타입으로 형 변환을 할 때는 명시적으로 타입을 지정해주어야 한다.\
  \-> **이때 부모 타입의 실제 객체는 자식 타입이어야만 한다.**
* instanceof 예약어를 사용하면 객체의 타입을 확인할 수 있다.
* instanceof 로 타입 확인할 때 부모 타입도 true 라는 결과를 제공한다.\
  \-> **때문에 타입을 확인할 때는 가장 하위에 있는 자식 타입부터 확인을 해야 제대로 타입 점검이 가능하다.**

