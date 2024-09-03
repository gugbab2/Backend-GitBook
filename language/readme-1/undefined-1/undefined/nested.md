# Nested 클래스

## 내부 클래스 구분&#x20;

* Nested class&#x20;
  * Static nested class
  * inner class
    * Local inner class
    * Anonymous inner class&#x20;

## Static nested 클래스 특징&#x20;

* 내부 클래스는 감싸고 있는 외부 클래스의 어떤 변수에도 접근할 수 있다.&#x20;
  * 심지어 private 로 선언 된 변수까지도 접근이 가능하다.&#x20;
* 하지만 Staic nested 클래스를 그렇게 사용하는 것은 불가능하다.&#x20;
  * **static 변수, 메소드만 접근이 가능하다.** \
    **-> 만약 static 이 붙어 있지 않은 변수, 메소드에 접근하면 컴파일 에러가 발생한다.**&#x20;

```java
public class OutherOfStatic {
    static class StaticNested {
        private int value = 0;
        public int getValue() {
            return value; 
        }
        
        public void setValue(int value){
            this.value=value; 
        }
    }
}
```

* 위 코드에서 내부에 있는 Nested 클래스는 별도로 컴파일 할 필요가 없다.&#x20;
  * 왜냐면, 여기서 OuterOfStatic 이라는 감싸고 있는 클래스를 컴파일하면 자동으로 컴파일되기 때문이다.&#x20;
* 컴파일하면 다음과 같이 두 개의 클래스가 만들어진다.&#x20;
  * `OuterOfStatic.class`&#x20;
  * `OuterOfStatic$StaticNested.class`

### Static nested 클래스 객체를 만드는 방법&#x20;

```java
public class NestedSample{
    public static void main(String[] args){
        NestedSample sample = new NestedSample(); 
        sample.makedStaticNestedObject(); 
    }
    public void makedStaticNestedObject() {
        OuterOfStatic.StaticNested staticNested = new OuterOfStatic.StaticNested();
        staticNested.setValue(3);
        System.out.println(staticNested.getValue());
    }
    
}
```

#### 왜 이렇게 귀찮게 Static Nested 클래스를 만들까?

* 일반적으로 Static Nested 클래스를 만드는 이유는 클래스를 묶기 위해서이다.&#x20;
* 고등학교, 대학교 별 학생을 관리하고 싶다면 어떻게 만들 수 있을까?&#x20;

```java
public class University{
    static class Student{
    
    }
}
```

```java
public class HighSchool {
    static class Student{
    
    }
}
```

* **위 코드와 같이 겉으로 보기에는 유사하지만, 내부적으로 구현이 달라야 할 때 Static Nested 클래스를 사용한다.**&#x20;

## 내부 클래스와 익명 클래스&#x20;

### 내부 클래스&#x20;

* 내부 클래스는 위에서 살펴 본 Static Nested 클래스와 겉으로 차이는 Static 을 쓰느냐 쓰지 않느냐의 차이만 있을 뿐이다.&#x20;

```java
public class OutofInner{
    class Inner {
        private int value = 0;
        public int getValue() {
            return value; 
        }
        public void setValue(int value) {
            this.value = value; 
        }
    }
}
```

```java
public class InnerSample {
    public static void main(String[] args){
        InnerSample sample = new InnserSample();
        sample.makeInnerObject();
    }
    
    public void makeInnerObject() {
        OuterOfInner outer = new OuterOfInner();
        OuterOfInner.Inner inner = outer.new Inner();
        inner.setValue(3);
        System.out.println(inner.getValue());
    }
}
```

* 위 코드에서 객체를 생성한 후 사용하는 방법은 차이가 없지만, 객체를 생성하는 방법에는 차이가 있다.&#x20;
  * 내부 클래스는 바깥을 감싸고 있는 `OuterOfInner` 클래스를 먼저 생성해야 한다.&#x20;
  * 그 후 `Inner` 클래스를 생성할 수 있다.&#x20;

#### 왜 이렇게 귀찮게 내부 클래스를 만들까?

* 내부 클래스를 만드는 이유는 캡슐화 때문이다.&#x20;
* 하나의 클래스에서 어떤 공통적인 작업을 수행하는 클래스가 필요한데, 다른 클래스가 전혀 필요가 없을 때 이러한 내부 클래스를 만들어 사용한다.&#x20;
  * 내부 클래스는 GUI 관련 프로그램을 개발할 때 가장 많이 사용한다.&#x20;
  * 하나의 애플리케이션에서 어떤 버튼이 눌렸을 때 수행해야 하는 작업은 대부분 상이하다. 그러니 하나의 별도 클래스를 만들어 사용하는 것보다 내부 클래스를 만드는 것이 훨씬 편하다.&#x20;

### 익명 클래스&#x20;

* 내부 클래스를 만드는 것보다도 더 간단한 방법은 "익명 클래스" 를 만드는 것이다.&#x20;

```java
public class MagicButton {
    public MagicButton() {
    
    }
    
    private EventListener listener;
    public void setListener(EventListener listener){
        this.listener = listener;
    }
    public void onClickProcess() {
        if(listener!=null){
            listener.onClick();
        }
    }
}
```

```java
public interface EventListener{
    public void onClick();
}
```

```java
public class AnonymousSample {
    public static void main(String[] args){
        AnonymousSample sample = new AnonymousSample();
        sample.setButtonListener();
    }
    
    public void setButtonListener() {
        MagicButton button = new MagicButton();
        button.setListener(new EventListener() {
            public void onClick(){
                System.out.println("Magic Button Clicked!!!);
            }
        });
        button.onClickProcess();
    }
}
```

* 위 코드와 같이 클래스를 생성하는 것이 아닌, 실행 시점에 클래스에 필요한 구현들을 초기화해주고 사용하면 된다.&#x20;

#### 왜 이렇게 귀찮게 익명 클래스를 만들까?

* 만약 재사용하지 않는 클래스를 정의해야 한다고 했을 때 새로운 클래스를 정의하는 것은 유지보수 측면에서 불필요한 자원 낭비일 수 있다.&#x20;
* 이 때, 익명 클래스를 사용해 인스턴스를 바로 만들어 사용하는 것이 유지보수 측면에서 이점이 클 것이다.&#x20;



