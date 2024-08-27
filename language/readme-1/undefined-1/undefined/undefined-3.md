# 참조 자료형

## 생성자&#x20;

#### 자바에서 생성자는 왜 필요할까?&#x20;

* 자바 생성자는 클래스의 객체를 생성하기 위해서 존재한다.&#x20;
* 생성자가 없다면 객체를 생성할 방법이 사라진다..&#x20;

#### 생성자는 몇개까지 만들 수 있을까?&#x20;

* 생성자의 개수는 정해져 있지 않다. 1개 이상 만들 수 있다.&#x20;
* 생성자의 개수가 정해져 있지 않은 이유는 무엇일까? 자바 패턴 중에서 DTO(Data Transfer Object) 라는 것이 있는데, 어떤 속성을 갖는 클래스를 만들고, 그 속성들을 쉽게 전달하기 위해서 DTO 라는 것을 만든다.
* 클래스 객체를 생성할 때, 알고 있는 속성의 값이 상황에 따라서 다를 수 있다. 이때 생성자는 상황에 맞는 DTO 를  만들 수 있는 도구가 되곤 한다.&#x20;
* 물론 그렇다고 해서 무분별한 생성자 생성은 좋지 않다.. 꼭 필요한 경우에만 만드는 습관을 길러야 한다.&#x20;

### 기본 생성자

* 자바는 생성자를 만들지 않아도 자동으로 만들어지는 기본 생성자가 있다.&#x20;
  * `new` 키워드 옆에 있는 `ReferenceDefault()` 가 생성자이다. \
    \-> 이와 같이 매개변수 없는 기본 생성자는 다른 생성자가 없는 경우 컴파일러가 기본적으로 만들어준다. \
    \-> 만약 클래스 내부에서 다른 생성자를 추가한다면 기본 생성자는 만들어지지 않는다.&#x20;

```java
ReferenceDefault reference = new ReferenceDefault(); 
```

## this&#x20;

* `this` 예약어는 말 그대로 "이 객체" 라는 의미이다.&#x20;
* 매개변수를 하나만 받는 생성자를 살펴보자&#x20;
  * 매개변수와 속성의 이름은 같지만, 속성을 갖는 주체가 누구인지 `this` 키워드를 통해서 명시하고 있다. &#x20;

```java
public class MemberDTO{
    public String name; 
    public String phone;
    public String email;
    public MemberDTO(String name) {
        this.name = name;
    }
}
```

## overloading

* 클래스 내에 생성자, 메서드를 선언할 때 이름이 동일해도, 매개변수가 다르다면 컴파일러는 다른 생성자, 메서드로 인식하는데 이러한 동작을 오버로딩(overloading) 이라고 부른다.&#x20;

```java
public class ReferenceOverloading {
    ...
    public void print(int data) {
    
    }
    public void print(String data){
    
    }
    public void print(int intData, String stringData){
    
    }
    public void print(String stringData, int intData){
    
    }
}
```

#### 왜 오버로딩이 필요할까?&#x20;

* 같은 기능을 사용하지만, 매개변수를 다르게 사용하고 싶을 때가 존재한다. \
  \-> `System.out.println()` 같이..
* 이런 경우, 매개변수에 따라서 메서드의 이름을 변경한다는 것은 매우 소모적인 작업이 될 것이다.&#x20;
* 때문에, 오버로딩 기능을 제공함으로 위와 같은 소모적인 작업이 이루어지지 않도록 한다.&#x20;

## static 메서드&#x20;

* static 메서드는 객체를 생성하지 않아도 메서드를 호출할 수 있는 마법의 메서드이다.&#x20;
* **static(정적) 키워드가 들어간 변수, 메서드는 프로세스가 올라갈 때 메모리에 적재가 되는데, 때문에 static 메서드에는 인스턴스 변수, 지역 변수는 사용할 수 없고 클래스 변수만 사용이 가능하다.** \
  \-> 라이프사이클이 다르기 때문에, 같은 라이프사이클인 클래스 변수만 사용이 가능하다.&#x20;

```java
public class ReferenceStatic {
    public static String name = "Min";
    // ...
    public static void staticMethodCallVariable(){
        System.out.println(name);
    }
}
```

## static 블록&#x20;

* **static 블록은 객체가 생성되기 전에 한 번만 호출되고, 그 이후에는 호출하려고 해도 호출 할 수 없다.**&#x20;
* **그리고, 클래스 내에 선언되어 있어야 하며, 메소드 내에서는 선언할 수 없다.** \
  \-> 즉, 인스턴스 변수나 클래스 변수와 같이 어떤 메소드나 생성자에 속해 있으면 안된다..
* static 메서드와 마찬가지로 static 블록 안에서는 static 한 것(클래스 변수, static 메서드)들만 호출 가능하다.&#x20;

## Pass by value vs Pass by reference

### Pass by value&#x20;

* 모든 기본 자료형은 Pass by value 로 데이터를 전달한다.&#x20;

```java
public class PassByValueExample {
    public static void main(String[] args) {
        int original = 10;
        modifyPrimitive(original);
        System.out.println("After method call: " + original); // 10
    }

    public static void modifyPrimitive(int number) {
        number = 20; // 원본에는 영향이 없음
    }
}

```

### Pass by reference

* 참조 자료형은 값이 아닌 참조를 전달하는 Pass by reference 로 데이터를 전달한다.&#x20;

```java
import java.util.ArrayList;
import java.util.List;

public class PassByReferenceExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Original");
        
        modifyReference(list);
        System.out.println("After method call: " + list); // [Modified]
    }

    public static void modifyReference(List<String> list) {
        list.add("Modified"); // 원본 리스트가 변경됨
    }
}

```
