# 혼동할 수 있는 와일드카드 표현

## 와일드카드는 설계가 아닌 사용을 위한 것

* 와일드카드에 대한 흔한 착각은 와일드카드를 어디서나 사용할 수 있다고 생각하는 것이다.&#x20;
* 하지만 와일드카드는 아래와 같이 클래스나 인터페이스 제네릭을 설계할 때는 아예 사용할 수 없다.&#x20;

```java
class Sample<? extends T> { // ! Error
    
}
```

* 와일드카드는 이미 만들어진 제네릭 클래스나 메스드를 사용할 때 이용하는 것으로 보면 된다. \
  **(Like! Collection Framework)**
* 예를들어, 다음과 같이 변수나 매개변수에 어떠한 객체의 타입 파라미터를 받을 때 그에 대한 범위를 한정을 정할 때 사용된다고 보면 된다.&#x20;

```java
class Sample<T> {
    public static <E> void run(List<? super E> l) {}
}

public class Main {
    public static void main(String[] args) {
        Sample<?> s2= new Sample<String>();
        
        Sample<? extends Number> s1 = new Sample<Integer>();
        
        Sample.run(new ArrayList<>());    // 타입을 지정하지 않는 경우 Object 로 추론된다. 
    }
}
```

## \<T extends 타입> 과 \<? extends U> 차이점&#x20;

* 바로 위에서 언급했듯이, 와일드카드는 제네릭 클래스를 만들 때 사용하는 것이 아니라, 이미 만들어진 제네릭 클래스를 사용할 때 타입을 지정할 때 이용되는 것이다.
* **즉, \<T extends 타입> 은 제네릭 클래스를 설계할 때 적어주는 것이고, \<? extends 타입> 은 이미 만들어진 제네릭 클래스를 인스턴스화 하여 사용할 때 타입 파라미터로 넘겨줄 때 이용되는 것이다.**&#x20;
