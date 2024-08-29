# interface, abstract class, enum

#### interface, abstract class 를 사용하는 이유&#x20;

* 설계시 선언해 두면 개발할 때 기능을 구현하는 데에만 집중할 수 있다.&#x20;
* 개발자의 역량에 따른 메서드의 이름과 매개 변수 선언의 격차를 줄일 수 있다.&#x20;
* 공통적인 인터페이스와 추상클래스를 선언해 놓으면, 선언과 구현을 구분할 수 있다.&#x20;

## interface

```java
public interface MemberManager {
    public boolean addMember(MemberDTO member); 
    public boolean removeMember(String name, String phone);
    public boolean updateMember(MemberDTO member);
}

public class MemberManagerImpl implements MemberManager {
    @Override 
    public boolean addMember(MemberDTO member) {
        return false; 
    }
    
    @Override 
    public boolean removeMember(String name, String phone) {
        return false; 
    }
    
    @Override 
    public boolean updateMember(MemberDTO member) {
        return false; 
    }
}

public class interfaceExample {
    public static void main(String args[]){
        // MemberManager member = new MemberManager();    // compile error
        MemberManager member = new MemberManagerImpl();    // success
    }
}
```

* 위 코드는 인터페이스를 선언하고, 클래스가 인터페이스를 상속받은 코드를 볼 수 있다.&#x20;
  * **인터페이스의 메서드는 선언부만 있고 구현부가 없다.**&#x20;
  * **상속받은 클래스에서 메서드의 구현부를 작성해주어야 한다.** \
    **-> 구현부를 작성하지 않으면 컴파일 에러가 발생한다.**&#x20;
  * **인터페이스는 메서드의 구현부가 없기 때문에(다른말로 하면 실체가 없다), 객체를 생성할 수 없다.** \
    **-> 타입은 인터페이스 타입으로 하더라도, 생성하는 타입은 인터페이스를 상속받은 타입이어야만 한다.**&#x20;
  * 인터페이스는 `static`, `final` 메서드가 선언되어 있으면 안된다.&#x20;
  * 다중 상속이 가능하다.&#x20;

## abstract class

```java
public abstract class MemberManagerAbsract {
    public abstract boolean addMember(MemberDTO member); 
    public abstract boolean removeMember(String name, String phone);
    public abstract boolean updateMember(MemberDTO member);
    public void printLog(String data){
        System.out.println("Data="+data);
    }
}

public class MemberManagerImpl extends MemberManagerAbsract {
    @Override 
    public boolean addMember(MemberDTO member) {
        return false; 
    }
    
    @Override 
    public boolean removeMember(String name, String phone) {
        return false; 
    }
    
    @Override 
    public boolean updateMember(MemberDTO member) {
        return false; 
    }
}

public class interfaceExample {
    public static void main(String args[]){
        // MemberManager member = new MemberManagerAbsract();    // compile error
        MemberManager member = new MemberManagerImpl();    // success
    }
}
```

* 위 코드는 추상클래스를 선언하고, 클래스가 추상클래스를 상속받은 코드를 볼 수 있다.&#x20;
  * 추상클래스는 클래스 선언시 `absract` 예약어가 앞에 추가되면 된다.&#x20;
  * 추상클래에서는 추상메서드가 0개 이상 있으면 된다.&#x20;
  * 추상메서드가 하나라도 있으면 그 클래스는 반드시 추상클래스로 선언되어야 한다.&#x20;
  * 추상클래스는 구현이 있는 메서드가 0개 이상 있을 수 있으며,&#x20;
  * `static`, `final` 메서드가 선언되어 있어도 된다.&#x20;
  * 다중 상속은 불가하다.&#x20;

## final&#x20;

### 클래스에 final

```java
public final class FinalClass {

}
```

* 위 코드와 같이 클래스 선언시 `final` 키워드를 사용할 수 있다.&#x20;
* **클래스에 `final` 키워드를 사용하게 되면 상속을 받을 수 없게 된다.**&#x20;
* 왜 사용할까?
  * `String` 과 같이 매우 중요도가 높은 클래스를 상속 받아 메서드를 의도와 다르게 오버라이딩 하게 된다면, 기본 기능을 변경하는 것이다.&#x20;
  * 더 이상 확장해서는 안되는 클래스를 선언할 때 `final` 키워드를 사용하자.

### 메서드에 final

```java
public absract class FinalMethodClass {
    public final void printLog(String data) {
        System.out.println("Data="+data);
    }
}
```

* 메서드의 경우에도 클래스와 비슷한 이유로 `final` 을 사용한다.&#x20;
* **메서드에 `final` 키워드를 사용하게 되면 오버라이딩 할 수 없게 된다.**&#x20;

### 변수에 final&#x20;

```java
public class FinalVariable {
    // final int instanceVariable;    // compile error
    final int instanceVariable = 0;    // success
}
```

* 클래스, 메서드에 `final` 을 붙이게 되면 각각 상속, 오버라이딩을 할 수 없게 된다.&#x20;
* **변수에 `final` 키워드는 성격이 조금 다른데, 변수에 `final` 키워드를 사용하게 되면 해당 변수를 변경할 수 없다는 의미가 된다. (기본 자료형, 참조 자료형 모두 동일하게 해당된다)**&#x20;
* **`final` 변수에 대해서는 선언과 동시에 초기화를 해주어야 한다. 안 그러면 컴파일 에러가 발생한다.**&#x20;
  * **매개변수는 사용 시 초기화가 되기 때문에, 매서드 선언시 초기화 하지 않아도 된다.**&#x20;
  * **지역변수의 초기화는 메서드를 사용하는 시점에 이루어진다. 때문에, 선언과 초기화가 동시에 이루어지지 않아도 된다.** \
    **-> 하지만, 한번의 초기화는 반드시 이루어져야 한다.**&#x20;

## enum

```java
public enum OverTimeValues{
    THREE_HOUR, 
    FIVE_HOUR, 
    WEEKEND_FOUR_HOUR,
    WEEKEND_EIGHT_HOUR;
}
```

* 만약 어떤 클래스가 상수(`final String`)로만 이루어져 있다면 굳이, 클래스로 만들지 않아도 된다.&#x20;
* 이 때 enum 클래스를 만들 수 있는데, 이 객체는 상수 집합이라는 뜻이다.&#x20;
* 만약, enum 의 값을 지정하지 않는다면, 값을 가지지 않는다. \
  \-> C, C++ 은 정수값을 가지는 것과 대조된다.&#x20;

### enum 을 class 처럼 사용하기&#x20;

```java
public enum Day {
    MONDAY("Monday"),
    TUESDAY("Tuesday"),
    WEDNESDAY("Wednesday");

    private String dayName;

    // enum 생성자
    Day(String dayName) {
        this.dayName = dayName;
    }

    public String getDayName() {
        return this.dayName;
    }
}

public class Main {
    public static void main(String[] args) {
        Day day = Day.MONDAY;
        System.out.println(day); // MONDAY 출력 (문자열이 아니다) 
        System.out.println(day.getDayName()); // "Monday" 출력
    }
}

```

* 만약 enum 속성에 특정한 값으로 초기화하고 싶다면 변수, 생성자, get메서드를 정의해서 사용할 수 있다.&#x20;
* enum 속성을 출력하면 문자열처럼 나오지만, 문자열이 아니다..&#x20;
