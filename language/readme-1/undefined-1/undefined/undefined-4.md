# 상속

## 상속이란?

* 부모 클래스에서는 기본 생성자를 만들어 놓는 것이 이외에는 상속을 위해서 아무런 작업을 할 필요가 없다.&#x20;
* 자식 클래스는 클래스 선언시 `extends` 다음에 부모 클래스 이름을 적어준다.&#x20;
* 자식 클래스의 생성자가 호출되면, 자동으로 부모 클래스의 매개변수 없는 생성자가 실행된다.&#x20;
  * 만약 부모 클래스가 기본 생성자가 없다면, `super` 키워드를 통해서 호출하고자 하는 부모 생성자를 지칭해주어야 한다. \
    \-> `null` 을 매개변수로 넘겨주게 되면 "클래스 참조가 모호하다" 라는 오류 메시지를 확인할 수 있기에, 정확한 타입을 넘겨주도록 하자.
* 자식 클래스에서는 부모 클래스에 있는 `public`, `protected` 로 선언된 모든 인스턴스 및 클래스 변수와 메소드를 사용할 수 있다.&#x20;
* 다중 상속은 불가하다 ..&#x20;
* **상속의 가장 큰 목적은 확장이 가장 크다.** \
  **-> 상속을 지칭하는 키워드 또한 `extends`(확장) 이다.**

## 메서드 overriding

* 상속의 관계에서 자식이 부모의 메서드를 재정의 하고 싶을 때 사용할 수 있는 방법으로 부모 메서드 시그니처와 동일한 형태로 자식 클래스에서 재정의 하여 사용할 수 있다.&#x20;
* 리턴 타입은 달라져도 되지 않을까? 라고 생각할 수 있지만, 리턴타입까지 동일해야 한다.&#x20;
* 접근제어자의 경우, 확대되는 경우는 가능하지만, 축소되는 경우는 컴파일 오류가 발생한다. &#x20;
  * ex) private -> public (O)
  * ex) public -> private (X)&#x20;

## 참조자료형의 형변환&#x20;

```java
public class ParentCasting {
    public ParentCasting(){
    
    }
    public ParentCasting(String name){
    
    }
    public void printName() {
        System.out.println("printName() - Parent");
    }
}

public class ChildCasting extends ParentCasting {
    public ChildCasting(){
    
    }
    public ChildCasting(String name){
    
    }
    public void printName() {
        System.out.println("printName() - Child");
    }
    public void printAge() {
        System.out.println("printAge() - 18 month");
    }
} 
```

* 위와 같은 상속 관계가 존재할 때, 부모 타입에 자식 인스턴스를 할당할 수 있지만, 자식 타입에 부모 인스턴스를 할당할 수 없다 ..&#x20;

```java
ParentCasting obj = new ChildCasting();     // O
ChildCasting obj2 = new Parentcasting();    // X
```

* 왜그럴까? `ParentCasting` 클래스에서는 `ChildCasting` 클래스에 있는 모든 메서드와 변수를 사용할 수 있고, 그렇지 않을수도 있다. 만약 `ChildCasting` 클래스에 추가된 메서드나 변수가 없으면 가능할 수 있다.&#x20;
* **하지만, 자바 컴파일러에서는 자식 객체를 생성할 때 부모 생성자를 사용하면 안 된다고 못 박아 버린다.** \
  **-> 명시적으로 형 변환을 한다고 알려주어야 한다.** \
  **-> 만약 명시적으로 캐스팅하지 않으면 컴파일 에러가 발생한다.**&#x20;

```java
public class InheritanceCasting{
    public static void main(String args[]){
        InheritanceCasting inheritance = new InheritanceCasting();
        inheritance.objectCast();
    }
    
    public void objectCast(){
        ParentCasting parent = new ParentCasting();  
        ChildCasting child = new ChildCasting();    
        
        ParentCasting parent2 = child;
        ChildCasting child2 = (ChildCasting)parent;    // Runtime Error
    }
}
```

* **위 코드에서 명시적으로 캐스팅해 컴파일 에러는 넘어가더라도 런타임 에러가 발생한다.**\
  **-> `parent` 는 본래 `ParentCasting` 클래스의 인스턴스이기 때문에 `ChildCasting` 타입에 변수에서는 사용하지는 못하는 것이다.**&#x20;

```java
public class InheritanceCasting{
    public static void main(String args[]){
        InheritanceCasting inheritance = new InheritanceCasting();
        inheritance.objectCast2();
    }
    
    public void objectCast2(){
        ChildCasting child = new ChildCasting();    
        ParentCasting parent2 = child;
        ChildCasting child2 = (ChildCasting)parent2;    // Success
    }
}
```

* **하지만, 위 코드에서는 문제가 발생하지 않는다.** \
  **-> `parent2` 는 본래 `ChildCasting` 클래스의 인스턴스이기 때문에 `ChildCasting` 타입에 변수에서는 사용하지는 못하는 것이다.**&#x20;

## 다형성(Polymorphism)

```java
public class InheritancePoly{
    public static void main(String args[]){
        InheritancePoly inheritance = new InheritancePoly();
        inheritance.callPrintNames();
    }
    
    public void callPrintNames(){
        Parent parent1 = new Parent();
        Parent parent2 = new Child1();
        Parent parent3 = new Child2();
        
        parent1.printName();
        parent2.printName();
        parent3.printName();
    }
}
```

* 위 코드에서 `Parent` 타입으로 선언되어 있지만, 실제 인스턴스는 제 각각 이기 때문에, `printName` 메서드의 실행 결과는 다를 수 있다.&#x20;
* **다형성은 말 그대로 형태가 다양하다는 말로, 위와 같이 하나의 타입으로 실제 인스턴스에 따라 다양한 결과를 볼 수 있는 것을 의미한다.**&#x20;
