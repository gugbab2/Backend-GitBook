# Object Class

* 자바 개발은 한다면 Object 클래스에서 사용하는 메서드들을 잘 알고 있어야 한다.&#x20;
  * 객체 처리를 위한 메서드&#x20;
    * toString()
    * equals()
    * hashCode()
    * getClass()
    * clone()
    * finalize()
  * 쓰레드 처리를 위한 메서드
    * notify()
    * notifyAll()
    * wait()

## toString()&#x20;

* toString() 메서드는 해당 클래스가 어떤 객체인지 나타내는 가장 중요한 메서드이다.&#x20;
* **이 메서드가 자동으로 호출되는 경우는 다음과 같다.**&#x20;
  * **System.out.println() 메서드에 매개 변수로 들어가 있는 경우**&#x20;
  * **객체에 대해 더하기 연산을 하고 있는 경우**&#x20;
* 오버로딩 하지 않은 toString() 의 경우 다음과 같은 형태를 띄게 된다.&#x20;
  * **ClassName**: 객체의 클래스 이름입니다. 패키지 이름을 포함한 클래스의 전체 이름이 반환됩니다.
  * **@**: "at" 기호입니다.
  * **HashCodeInHexadecimal**: 객체의 해시 코드(hash code)를 16진수(헥사)로 변환한 값입니다. 이 해시 코드는 객체의 메모리 주소를 기반으로 생성된 숫자로, 고유하게 객체를 식별하는 데 사용됩니다.\
    \-> **자바는 주소라는 개념을 가지고 있지 않기 때문에, 실제 주소값은 아니고 JVM 내에서 주소와 매핑되는 해시값이다.**&#x20;

```java
ClassName@HashCodeInHexadecimal
```

* 일반적으로 toString() 은 DTO 를 정의할 때 오버라이딩 해 해당 객체를 쉽게 확인할 수 있도록 만들어두는 것이 유용하다.&#x20;

```java
public void toStringMethod(Object obj){
    System.out.println(obj);                //ToString Class
    System.out.println(obj.toString());     //ToString Class
    System.out.println("plus + " + obj);    //plus + ToString Class
}

public String toString(){
    return "ToString Class";
}
```

## equals()&#x20;

* 비교연산자 중 `==`, `!=` 는 값의 같은지 다른지를 비교할 수 있다. **하지만 기본 자료형에 한정해서 사용할 수 있다는 제약사항을 가지고 있다.**\
  **-> 참조 자료형도 사용할 수 있지만, "값" 을 비교하는 것이 아닌, "레퍼런스" 값을 비교한다.**&#x20;
* 이를 위해서 `Object` 클래스 에서는 `equals()` 메서드가 제공되는데 **사용하고자 하는 참조자료형에서 요구사항에 맞게 overriding 해서 사용하면 된다.**
* **`equals()` 메서드에서는 다음과 같은 5가지의 조건을 만족시켜야 한다. (자바 API 문서 명시)**
  * **재귀** : null 이 아닌 x 라는 객체의 `x.equals(x)` 는 항상 true 이어야 한다.&#x20;
  * **대칭** : null 이 아닌 x 와 y 객체가 있을 때 `y.equals(x)` 가 true 을 리턴했다면, `x.equals(y)` 또한 반드시 true 를 리턴해야 한다.&#x20;
  * **타동적** : null 이 아닌 x,y,z 가 있을 때 `x.equals(y)` 가 true 를 리턴하고, `y.equals(z)` 가 true 를 리턴했다면 `x.equals(z)` 는 반드시 true 를 리턴해야 한다.&#x20;
  * **일관** : null 이 아닌 x,y 가 있을 때 객체가 변경되지 않은 상황에서는 몇번을 호출하더라도 `x.equals(y)` 가 항상 true 를 리턴해야 한다.&#x20;
  * **null 과의 비교** : null 이 아닌 객체 x 라는 객체의 `x.equals(null)` 결과는 항상 false 를 리턴해야 한다.&#x20;
* **`equals()` 메서드 Overriding 시 `hashCode()` 메서드도 함께 Overriding 해야한다.** \
  **-> `equals()` 메서드를 통해서 값이 같을 수는 있지만, 그 객체의 주소 값이 같지는 않기 때문이다.**&#x20;

## hashCode()&#x20;

* `hashCode()` 메서드는 기본적으로 객체의 메모리 주소를 16 진수로 리턴한다. **만약 어떤 두 개의 객체가 서로 동일하다면 `hashCode()` 값은 무조건 동일해야만 한다.**&#x20;
* **`hashCode()` 메서드에서는 다음과 같은 3가지의 조건을 만족시켜야 한다. (자바 API 문서 명시)**
  * 자바 애플리케이션이 수행되는 동안에 어떤 객체에 대해서 이 메서드가 호출될 때에는 항상 동일한 int 값을 리턴해주어야 한다. 하지만 자바를 실행할 때마다 같은 값이어야 할 필요는 없다.&#x20;
  * 어떤 두개의 객체에 대하여 equals() 메서도를 사용하여 비교한 결과가 true 일 경우, 두 객체의 hashCode() 값은 항상 동일한 int 값을 리턴해야만 한다.&#x20;
  * 두 객체를 equals() 메소드를 사용하여비교한 결과 false 를 리턴했다고 해서, hashCode() 메서드를 호출한 int 값이 무조건 달라야 할 필요는 없다. 하지만 이 경우에 서로 다른 int 값을 제공하면 hashtable 의 성능을 향상시키는데 도움이 된다. \
    \-> hash 값을 사용하는 Collection 라이브러리에서 사용된다.&#x20;
* 위와 같은 까다로운 제약조건 때문에, 직접 해당 메서드를 오버라이딩 하는 것은 권장되지 않는다. \
  \-> IDE 에서 제공하는 기능을 사용하자.&#x20;
