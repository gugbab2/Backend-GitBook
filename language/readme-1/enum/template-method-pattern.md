# 템플릿 메서드 패턴(Template Method Pattern) 으로 배우는 추상 클래스

## 1. 템플릿 메서드 패턴이란?

* 어떤 작업을 처리하는 일부분을 서브 클래스로 캡슐화해서 전체적인 구조는 바꾸지 않으면서 특정 단계에서 수행하는 내용을 바꾸는 패턴이다.&#x20;

### 1-1. 코드로 살펴보자

* 일단 기본적으로 Controller 라는 클래스는 초기화(init), 실행(run), 마무리(close) 3단계를 진행하게 되고 초기화와 마무리는 같은 코드를 갖되 실행 코드는 다른 코드를 가지도록 하고싶다.&#x20;
* 서브 클래스에서 run 을 오버라이딩 할 수 있도록, 추상메서드로 만들어준다.\
  \-> 이로 인해 Controller 클래스는 추상 클래스가 된다.&#x20;

```java

public abstract class Contoller {

    public void init(){
        System.out.println("초기화 하는 코드");
    }

    public void close(){
        System.out.println("마무리 하는 코드");
    }

    public abstract void run();

    // 내가 가지고 있는 메서드를 호출한다.
    // 어떤 순서를 가지고 있다.
    // 이런 메서드를 템플릿 메서드라고 한다. -> 정해진 무언가(순서) 대로 실행한다.
    public void execute(){
        this.init();
        this.run();
        this.close();
    }
}
```

* Controller 클래스를 상속받은, FirstController 클래스는 run() 메서드를 오버라이딩해 구현한다.&#x20;

```java
public class FirstController extends Contoller{
    @Override
    protected void run() {
        System.out.println("별도로 동작하는 코드 111");
    }
}
```

* 다음과 같이 main() 메서드에서 실행하게 되면 오버라이딩 한 run() 메서드가 실행되는 것을 볼 수 있다.&#x20;

```java
package example_controller;
    public static void main(String[] args) {
        Contoller c1 = new FirstController();
        c1.execute();
    }
}
```

### 1-2. 코드를 개선해보자

* 우리가 실제로 사용하는 메서드는 execute() 메서드 뿐이고, 오버라이딩 해야하는 메서드는 run() 메서드 뿐이다.\
  \-> 만약 중간에 클래스에서 원치않게 init(), close() 클래스를 오버라이딩 하게 된다면 원치 않는 결과를 맞이할 수 있다.&#x20;
* 때문에, 우리는 private 키워드를 통해 오버라이딩을 금지할 수 있다.&#x20;

```java
public abstract class Contoller {

    private void init(){
        System.out.println("초기화 하는 코드");
    }

    private void close(){
        System.out.println("마무리 하는 코드");
    }

    protected abstract void run();

    public void execute(){
        this.init();
        this.run();
        this.close();
    }
}

```
