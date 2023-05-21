# 스프링이 사랑한 디자인 패턴

## 1. 어댑터 패턴(Adapter Pattern)

* 어댑터는 변환기로 서로 다른 두 인터페이스 사이에 통신이 가능하도록 하는 것이다. \
  \-> 다른 클래스에서 비슷한 기능을 하는 메서드를 어댑터를 통해서 동일한 메시지를 통해 호출 하는 방법\
  \-> DB 의 JDBC / 플랫폼 별 JRE
* 어댑터 패턴은 합성(객체를 속성으로 만들어서 참조하는)을 사용하는 디자인 패턴으로 한문장으로 정리하면 다음과 같다. \
  \-> 호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기(변환 클래스) 를 통해 호출하는 패턴
* 각각의 어탭터 클래스를 인터페이스를 통해서 더욱 더 개선할 수 있다.&#x20;

## 2. 프록시 패턴(Proxy Pattern)

```java
public interface IService{
    public abstract String runSomething();
}

public class Service implement IService{
    public String runSomething(){
        return "서비스 긋.";
    }
}

public class Proxy implement IService{
    IService service;
    
    public String runSomething(){
        System.out.println("호출에 대한 흐름 제어가 주목적, 반환 결과를 그대로 전달.");
        
        Service1 = new Service();
        return service1.runSimething();
    }
}

public class ClientProxy{
    public static void main(String[] args){
        IService proxy = new Proxy();
        System.out.println(proxy.runSomething());
    }
}
```

* 대리자(인터페이스)를 통해서 메서드를 호출도록 하는 패턴
* 프록시 패턴 중요 포인트
  * **대리자는 실제 서비스와 같은 이름으로 메서드를 구현한다. 이 때 인터페이스를 사용한다.**
  * **대리자는 실제 서비스에 대한 참조 변수를 갖는다(합성).**
  * **대리자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 돌려준다.**&#x20;
  * **대리자는 실제 서비스의 메서드 호출 전후의 별도의 로직을 수행할 수 있다.**&#x20;
* 프록시 패턴은 실제 서비스 메서드의 반환 값에 가감하는 것을 목적으로 하지 않고 제어의 흐름을 변경하거나 다른 로직을 수행하기 위해 사용한다.&#x20;

## 3. 데코레이터 패턴(Decorator Pattern)

```java
public interface IService{
    public abstract String runSomething();
}

public class Service implement IService{
    public String runSomething(){
        return "서비스 긋.";
    }
}

public class Decoreator implement IService{
    IService service;
    
    public String runSomething(){
        System.out.println("호출에 대한 흐름 제어가 주목적, 반환 결과에 장식을 더하여 전달.");
        
        Service1 = new Service();
        return "장식" + service1.runSimething();
    }
}

public class ClientProxy{
    public static void main(String[] args){
        IService decor = new Decoreator();
        System.out.println(decor.runSomething());
    }
}
```

* 기본적으로 데코레이터 패턴은 프록시 패턴과 구현 방법이 같다. \
  \-> 하지만! 프록시 패턴은 클라이언트가 최종적으로 돌려 받는 반환값을 조작하지 않고 그대로 전달하는 반면,\
  **데코레이터 패턴은 클라이언트가 받는 반환값에 장식을 덧입힌다!**
* 데코레이션 패턴의 중요 포인트
  * 장식자는 실제 서비스와 같은 이름의 메서드를 구현한다.&#x20;
  * 장식자는 실제 서비스에 대한 참조 변수를 갖는다. (합성)
  * 장식자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고, \
    \====================================
  * 그 값에 장식을 더해서 클라이언트에게 돌려준다.
  * 장식자는 실제 서비스의 메서드 호출 전후에 별도의 로직을 수행할 수 있다.(비지니스 로직 등..)

## 4. 싱글턴 패턴(Singleton Pattern)

<pre class="language-java"><code class="lang-java"><strong>public class Singleton{
</strong><strong>    static Singleton singletonObject; //정적 참조 변수
</strong>    
    private Singleton(){};            //private 생성자
    
    //객체 반환 정적 메서드
    public static Singleton getInstance(){
        if(simpletonObject == null){
            simpletonObject = new Singleton();
        }
        
        return simpletonObject;
    }
}
</code></pre>

* 싱글턴 패턴은 인스턴스 하나만 만들어서 사용하기 위한 패턴이다. \
  \-> 커넥션 풀, 스레드 풀, 디바이스 설정 등과 같은 경우 인스턴스를 여러개 만들면 불필요한 자원을 사용하게 되고, 프로그램이 예상하지 못한 결과를 만들어낼 수 있다.&#x20;
* 싱글턴 패턴의 요소
  * new 를 실행할 수 없도록 생성자에 private 접근 제어자를 설정
  * 유일한 단일 객체에 반환할 수 있는 정적 메서드가 필요하다.
  * 유일한 단일 객체를 참조할 정적 변수가 필요하다.&#x20;
* 주의점!
  * **싱글턴 패턴은 하나의 객체를 공유하게 되는데, 해당 객체가 속성을 가지게 되면 사이드 이펙트가 생겨날 수 있기 때문에, 속성은 반드시 제거해야 한다.** \
    **-> 다만 읽기 전용 속성을 가지는 것은 문제가 되지 않는다.**&#x20;
* 싱글턴 패턴의 주요 포인트
  * **private 생성자를 갖는다.**
  * **단일 객체 참조 변수를 정적 속성으로 갖는다.**
  * **단일 객체 참조 변수가 참조하는 단일 객체를 반환하는 getInstance() 정적 메서드를 갖는다.**
  * **단일 객체는 쓰기 가능한 속성을 갖지 않는것이 정석이다.**

## 5. 템플릿 메서드 패턴(Template Method Pattern)

```java
public abstract class Animal{
    //템플릿 메서드
    public void playWithOwner(){
        System.out.println("귀염둥이 이리온");
        play();
        runSomething();
        System.out.println("잘했어");
    }
    
    abstract void play();
    
    void runSomething(){
        System.out.println("꼬리 살랑살랑");
    }
}

public class Dog extends Animal{
    @Override
    void play(){
        System.out.println("멍멍");
    }
    
    @Override
    void runSomething(){
        System.out.println("멍멍 꼬리 살랑살랑");
    }
}    

public class Cat extends Animal{
    @Override
    void play(){
        System.out.println("야옹");
    }
    
    @Override
    void runSomething(){
        System.out.println("야옹 꼬리 살랑살랑");
    }
}    

public class Driver{
    public static void main(String[] args){
        Animal bolt = new Dog();
        Animal kitty = new Cat();
        
        bolt.playWithOwner();
        
        System.out.println();
        System.out.println();
        
        kitty.playWithOwner();
    }
}    
```

* **상위 클래스에 공통 로직을 수행하는 템플릿 메서드와 하위 클래스에 오버라이딩을 강제하는 추상 메서드 또는 선택적으로 오버라이딩 할 수 있는, 훅 메서드를 두는 패턴을 템플릿 메서드 패턴이라고 한다.** \
  **-> 추상클래스, 인터페이스를 통해서 구현**
* **"상위 클래스의 템플릿 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴"**

## 6. 팩터리 메서드 패턴(Factory Method Pattern)

```java
public abstract class Animal{
    abstract AnimalToy getToy();
}

public abstract class AnimalToy{
    abstract void identify();
}

public class Dog extends Animal(){
    @Override
    AnimalToy getToy(){
        return new DogToy();
    }
}

public class DogToy extends AnimalToy{
    public void identify(){
        System.out.println("나는 테니스공, 강아지의 친구");
    }
}

public class Cat extends Animal(){
    @Override
    AnimalToy getCat(){
        return new CatToy();
    }
}

public class CatToy extends AnimalToy{
    public void identify(){
        System.out.println("나는 캣타워, 고양이의 친구");
    }
}

public class Driver{
    public static void main(String[] args){
        Animal bolt = new Dog();
        Animal kitty = new Cat();
        
        AnimalToy boltBall = bolt.getToy();
        AnimalToy kittyTour = kitty.getToy();
        
        boltBall.identify();
        kittyTour.identify();
    }
}
```

* 객체 지향에서 팩터리는 객체를 생성한다. \
  \-> 결국 팩터리 메서드는 객체를 생성 반환하는 메서드를 말한다.&#x20;
* **"오버라이드 된 메서드가 객체를 반환하는 패턴이다.** "

## 7. 전략 패턴(Strategy Pattern)

```java
public interface Strategy{
    public abstract void runStrategy();
}

public class StrategyGun implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("탕탕탕");
    }
}

public class StrategySword implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("챙챙");
    }
}

public class StrategBow implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("슉슉");
    }
}

public class Soldier{
    void runContext(Strategy strategy){
        System.out.println("전투 시작");
        strategy.runStrategy();
        System.out.println("전투 종료");
    }
}

public class Client{
    public static void main(String[] args){
        Strategy strategy = null;
        Soldier rambo = new Soldier();
        
        strategy = new StrategyGun();
        rambo.runContext(strategy);
        
        System.out.println();
        
        strategy = new StrategySword();
        rambo.runContext(strategy);
        
        System.out.println();
        
        strategy = new StrategBow();
        rambo.runContext(strategy);
        
        System.out.println();
    }
}
```

* 전략 패턴을 구성하는 요소 세가지
  * 전략 메서드를 가진 전략 객체
  * 전략 객체를 사용하는 컨텍스트(전략 객체의 사용자/소비자)
  * 전략 객체를 생성해 컨텍스트에 주입하는 클라이언트(제3자, 전략 객체의 공급자)
* 같은 문제를 템플릿 메서드 방법을 통해서도 해결할 수 있다.
  * **템플릿 메서드 : 상속을 이용**
  * **전략 패턴 : 객체 주입을 이용**
* **"클라이언트가 전략을 생성해서 전략을 실행한 컨텍스트에 주입하는 패턴"**

## 8. 템플릿 콜백 패턴

```java
public interface Strategy{
    public abstract void runStrategy();
}

public class Soldier{
    void runContext(String weaponSound){
        System.out.println("전투 시작");
        strategy.executeWeapon(weaponSound);
        System.out.println("전투 종료");
    }
    
    private Strategy executeWeapon(final String weaponSound){
        return new Strategy(){
            @Override
            public void runStrategy(){
                System.out.println(weaponSound);
            }
        }
    }
}

public class Client{
    public static void main(String[] args){
        Soldier rambo = new Soldier();
        
        rambo.runContext("탕탕");
        
        System.out.println();
        
        rambo.runContext("챙챙");
        
        System.out.println();
        
        rambo.runContext("슉슉");
        
        System.out.println();
    }
}
```

* 전략 패턴과 거의 모든 것이 동일하고, 전략을 익명 클래스로 정의해서 사용한다.
* **"전략을 익명 내부 클래스로 구현한 전략 패턴"**
