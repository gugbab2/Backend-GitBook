# 클래스 - 두번째

## 생성자 다중 정의&#x20;

* 일반 메서드처럼 생성자도 매개변수 구성이 다른 생성자를 여럿 정의 할 수 있음&#x20;
  * 여러 개가 다중 정의되어 있다하더라도 `new` 연산 시 호출되는 생성자는 1개
* **생성자에서 다른 생성자 호출 가능**
  * 일반적으로 기본 생성자에 초기화와 관련된 코드가 들어가고 그 외 생성자에서 케이스마다 필요한 작업을 하게 된다.
  * **때문에, 기본 생성자 외 생성자에서 기본생성자를 호출하는 구조로 만드는 것이 코드 중복을 줄일 수 있을 것이다.**&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-03-03 19.04.20.png" alt=""><figcaption></figcaption></figure>

## 깊은 복사와 얕은 복사 (중요!)&#x20;

### 얕은 복사&#x20;

* **참조의 대상은 복사해서 늘리지 않고 참조자만 늘리는 형태 (사이드 이펙트 발생의 원인)**&#x20;
* 대상 인스턴스는 그대로 두고 참조만 늘어나는 경우

```java
class Address {
    Address(String address, String phone) {
        this.address = address;
        this.phone = phone;
    }
    public String address;
    public String phon
}

class User {
    private String name;
    private Address address;
    
    User(String name, String address, String phone) {
        this.name = name;
        this.address = new Address(address, phone);
    }
    public Address getAddress() {
        return address;
    }
    public String getName() {
        return name;
    }
    
    // 얕은 복사로 인해 동일한 참조자를 사용하게 된다.(사이드 이펙트)
    public void copy(User rhs) {
        this.name = rhs.name;
        this.address = rhs.address;
    }
}

public class Main {
    public static void main(String[] args) {
        User user1 = new User("Hosung","Hanam", "010-1111-1111");
        User user2 = new User("Hoon","Seoul", "010-2222-2222");
        
        System.out.println(user1.getName());
        System.out.println(user1.getAddress().address);
        System.out.println(user1.getAddress().phone);
        
        user1.copy(user2);
        user2.getAddress().phone = "010-3333-3333";
        user2.getAddress().address = "Busan";
        
        System.out.println(user1.getName());
        System.out.println(user1.getAddress().address);
        System.out.println(user1.getAddress().phone);
    }
}
```

### 깊은 복사

* 원본 자체도 새로 할당하고 복사하는 방식&#x20;
* 얕은 복사와 달리 두 개의 원본 두 개의 참조가 각각 별도로 존재&#x20;
* 사이드 이펙트 오류 가능성 없음&#x20;
* 참조를 멤버로 가지며 인스턴스를 동적으로 할당하는 경우 복사 생성자 구현

```java
...

// String 클래스는 레퍼런스 타입이지만 불변객체로 설계된 특별한 클래스이기 때문에, 문제가 안난다. 
public void copy(User rhs) {
    this.name = rhs.name;
    //this.address = rhs.address;
    this.address.address = rhs.address.address;
    this.address.phone = rhs.address.phone;
}

...
```

## 복사 생성자&#x20;

```java
class-name(class-name rhs)
```

* 객체의 사본은 생성할 때 사용하기 적절한 생성자 함수&#x20;
  * C++ 스타일 문법&#x20;
* `rhs` 는 Right Hand Side 의 약어이며 복사의 원본(인스턴스에 대한 참조)&#x20;
* `clone()` 메서드를 만드는 방법이 있으나 규약이 모호한 부분이 있고 예외처리가 복잡한 단점이 있다.&#x20;

<pre class="language-java"><code class="lang-java">class MyTest {
    private int[] array = null;
    public MyTest(int size) {
        array = new int[size];
    }
    // 복사 생성자 
    public MyTest(MyTest rhs) {
        this(rhs.array.length);
        this.deepCopy(rhs);
    }
    public int getAt(int index) {
        return array[index];
    }
    public void setAt(int index, int value) {
        array[index] = value;
    }
    public void shallowCopy(MyTest rhs) {
        array = rhs.array;
    }
    public void deepCopy(MyTest rhs) {
        for(int i = 0; i &#x3C; rhs.array.length; ++i)
            array[i] = rhs.array[i];
    }
}

public class Main {
<strong>    public static void main(String[] args) {
</strong>        MyTest t1 = new MyTest(3);
        t1.setAt(0, 10);
        t1.setAt(1, 20);
        t1.setAt(2, 30);
        
        MyTest t2 = new MyTest(3);
        t2.shallowCopy(t1);
        MyTest t3 = new MyTest(3);
        t3.deepCopy(t1);
        MyTest t4 = new MyTest(t1);
        
        t1.setAt(0, -1);
        System.out.println("t1[0]: " + t1.getAt(0));    // -1
        System.out.println("t2[0]: " + t2.getAt(0));    // -1
        System.out.println("t3[0]: " + t3.getAt(0));    // 10
        System.out.println("t4[0]: " + t4.getAt(0));    // 10
    }
}
</code></pre>

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-03-03 19.54.19.png" alt=""><figcaption></figcaption></figure>

## 보이지 않는 임시 객체&#x20;

* 클래스가 함수의 반환 자료형이 될 경우(이름이 없는), 임시 객체를 생성&#x20;
* **`String` 클래스는 덧셈 연산 시 임시 객체 생성**&#x20;
  * 비효율의 직접적 원인이 될 수 있음&#x20;
  * `String` 은 명백한 클래스인데, 불변객체이기 때문에 덧셈 연산 시 새로운 `String` 객체를 생성해 반환한다.
  * `String` 을 리터럴로 사용하는 경우 힙 영역을 사용하는 것이 아닌, Runtime Constant Pool 을 사용한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-03-03 20.24.06.png" alt=""><figcaption></figcaption></figure>

## 정적 멤버&#x20;

* **클래스 인스턴스 없어도 독립적으로 존재 가능**&#x20;
  * 필드(모든 인스턴스에서 공유), 메서드&#x20;
* 일반 메서드와 달리 인스턴스 선언 없이 호출 가능&#x20;
* 메서드에서 `this` 를 사용할 수 없음&#x20;
* **정적 필드는 `final` 선언 함으로써 심볼릭 상수로 활용하는 경우가 많음**

### 정적 멤버와 인스턴스 메모리 차이&#x20;

* 정적 필드는 인스턴스 메모리와 독립적이며 메서드 코드는 인스턴스마다 다르지 않고 하나만 존재
* 클래스 메서드는 Method area 라는 영역에서 따로 존재한다.&#x20;
  * 왜냐면 인스턴스마다 필드는 달라질 수 있지만, 메서드는 동일하기 때문이다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-03-03 20.47.35.png" alt=""><figcaption></figcaption></figure>





