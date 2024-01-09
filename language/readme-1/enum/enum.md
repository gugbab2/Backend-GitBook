# Enum

## 0. Enum 이전 상수 사용

* Enum 이 나오기 전인 JDK 5 이전에는 다음과 같은 방식을 사용했다. (final static)\
  **->** **하지만 다음과 같은 경우 해당 변수만 사용하라는 강제를 할 수 없기 때문에, 안전하지 않다는 단점이 있다.**\
  **-> 특정 상수 타입이 아닌, 대부분 기본자료형을 사용한다..**

```java
public class DayType{
    public final static int MONDAY = 0;
    public final static int THUSDAY = 1;
    public final static int WENHSDAY = 2;
    public final static int THURSDAY = 3;
    public final static int FRIDAY = 4;
    public final static int SATURDAY = 5;
    public final static int SUMDAY = 6;
}
```

## 1. Enum

* 어떤 클래스가 상수만으로 이루어져 있을 때 반드시 class 를 사용할 필요가 없다.&#x20;
* enum 을 사용하게 되면, "이 객체는 상수의 집합이다." 라는 것을 명시적으로 나타낸다.

### 1-1. Enum 작성

* Enum에 있는 상수들은 별도의 타입을 지정할 필요도, 값을 지정할 필요도 없다.

<pre class="language-java"><code class="lang-java"><strong>public enum OverTimeValues{
</strong>    THREE_HOUR,
    FIVE_HOUR,
    WEEKEND_FOUR_HOUR,
    WEEKEND_EIGHT_HOUR;
}
</code></pre>

### 1-2. Enum 을 제대로 사용하기

* Enum 은 new 키워드 없이 객체가 생성된다고 생각하면 된다. 때문에 다음 코드와 같이 바로 인스턴스 변수에 접근해 사용할 수 있다.
* Enum 은 class 와 같이 생성사를 만들어주지 않아도 기본 생성자를 만들어주기 때문에, 생성자를 만들어주지 않아도 코드가 정상적으로 작동한다.&#x20;

```java
public enum OverTimeValues{
    THREE_HOUR(18000),
    FIVE_HOUR(30000),
    WEEKEND_FOUR_HOUR(40000),
    WEEKEND_EIGHT_HOUR(60000);
    
    private final int amount;
    OverTimeValues(int amount){
        this.amount = amount;
    }
    
    public int getAmount(){
        return amount;
    }
}

public class OverTimeManager{
    public static void main(String[] args){
        OverTimeValues value = OverTimeValues.FIVE_HOUR;
        System.out.println(value);
        System.out.println(value.getAmount());
    }
}
```
