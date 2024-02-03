# 다형성 기본

## 다형성&#x20;

* 다형성은 이름 그대로 "다양한 형태", "여러 형태" 를 뜻한다.&#x20;
* 프로그래밍에서 다형성은 한 객체가 여러 타입의 객체로 취급될 수 있는 능력을 뜻한다. \
  \-> 다형성을 사용하면 하나의 객체가 다른 타입으로 사용될 수 있다는 뜻이다.

### 다형적 참조

<pre class="language-java"><code class="lang-java"><strong>public class PolyMain {
</strong><strong>
</strong>    public static void main(String[] args) {
        // 부모 변수가 부모 인스턴스 참조
        System.out.println("Parent -> Parent");
        Parent parent = new Parent();
        parent.parentMethod();

        // 자식 변수가 부모 인스턴스 참조
        System.out.println("Child -> Child");
        Child child = new Child();
        child.parentMethod();
        child.childMethod();

        // 부모 변수가 자식 인스턴스 참조(다형적 참조)
        System.out.println("Parent -> Child");
        Parent poly = new Child();
        poly.parentMethod();
        poly.childMethod(); // error : 부모 타입은 자식 객체의 정보를 알지 못한다.
    }
}

</code></pre>

* 위 코드에서 보듯이 부모 타입은 자식 객체를 담을 수 있다. \
  \-> 부모 타입이 자식 객체를 담을 수 있는 것을 **다형적 참조**라 부른다.&#x20;
* 하지만, 반대로 자식 타입은 부모 객체를 담을 수 없다.\
  \-> ex, `Child poly = new Parent();   // error`

### 다형적 참조와 인스턴스 실행&#x20;

* `poly.parentMehod()` 를 호출하면 먼저 참조값을 사용해서 인스턴스를 찾는다. 그리고 다음으로 인스턴스 안에서 실행할 타입도 찾아야 한다. \
  \-> **poly 의 타입은 Parent**
* 때문에, `poly.childMehod()` 를 작성하면 컴파일 오류가 발생하게 되는데, 이때 해당 메서드를 실행하고 싶다면 어떻게 해야할까?&#x20;
* 바로 자식 타입으로 캐스팅이 필요하다. \
  \-> **실제 인스턴스가 자식 인스턴스이기 때문에, 자식 타입으로 캐스팅 하는 것은 문제가 되지 않는다.**&#x20;

### 다형적 참조의 문제를 해결하기 위한 다운캐스팅&#x20;

```java
public class CastingMain1 {
 
    public static void main(String[] args) {
        // 부모 변수가 자식 인스턴스 참조(다형적 참조)
        Parent poly = new Child();
        // 단 자식의 기능은 호출할 수 없다. (컴파일 오류 발생)
        // poly.childMethod(); => error

        // 다운 캐스팅(부모 타입 -> 자식 타입)
        Child child = (Child)poly;
        child.childMethod();
    }
}

```

## 다운캐스팅의 문제점

### 참조자료형의 형변환

* 참조 자료형은 자식 클래스의 타입을 부모 클래스의 타입으로 형 변환하면, 부모클래스에서 호출할 수 있는 메소드들은 자식 클래스에서도 호출할 수 있으므로 전혀 문제가 되지 않는다. (상속관계이기 때문에!)\
  \-> 따라서 형 변환을 명시적으로 해줄 필요가 없다.
* 아래코드와 같이 부모객체를 자식객체에서 사용하려는 경우 런타임 에러가 생긴다.\
  \-> 자식객체에서 사용하려는 객체가 실제로 부모객체인 경우 사용할 수 가 없다.(ClassCastException)\
  \-> 보통의 자식객체를 부모 객체보다 **확장된 개념이다.**

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

### instanceof : 다운캐스팅의 불안함을 해소하는 방법 &#x20;

```java
public class InheritanceCasting {

    public void objectCast(){
        ParentCasting parentCasting = new ParentCasting();
        ChildCasting childCasting = new ChildCasting();

        ParentCasting parentCasting1 = childCasting;
        if(parentCasting istanceof ChildCasting)
            ChildCasting childCasting1 = (ChildCasting)parentCasting;
    }
}
```

## 메서드 오버라이딩&#x20;

#### 가장 중요한 점은! 오버라이딩 된 메서드가 항상 우선권을 가진다.&#x20;

* 아래 코드와 같이, 다형적 참조가 이루어질 때, 변수는 오버라이딩이 안되지만(**인스턴스가 아닌 타입을 따라간다**) ,
* 메서드는 오버라이딩이 된다(**타입이 아닌 인스턴스를 따라간다**).&#x20;
* 메서드 오버라이딩은 항상 우선권을 가진다.

```java
public class OverridingMain {

    public static void main(String[] args) {
    
        Child child = new Child();
        System.out.println("Child -> Child");
        System.out.println("value = " + child.value);
        child.method();

        Parent parent = new Parent();
        System.out.println("Parent -> Parent");
        System.out.println("value = " + parent.value);
        parent.method();

        // 다형적 참조
        Parent poly = new Child();
        System.out.println("Parent -> Child");
        System.out.println("value = " + poly.value);    // 변수는 오버라이딩 X
        poly.method();                                  // 메서드는 오버라이딩 O
    }
}

```
