# 스프링이 사랑한 디자인 패턴

#### 디자인 패턴은 객체 지향의 특성 중 상속(extends), 인터페이스(implements), 합성(객체를 속성으로 사용) 을 이용한다. 다른 방식은 없다.&#x20;

## 1. 어댑터 패턴(Adapter Pattern)

* **어댑터는 변환기(convert) 로 서로 다른 두 인터페이스 사이에 통신이 가능하도록 하는 것이다.**
  * 다양한 데이터베이스 시스템을 공통의 인터페이스인 JDBC 를 통해 조작한다.&#x20;
  * 다양한 운영체제의 기계어를 JVM 을 통해서 만들어낸다.&#x20;
* 한문장으로 정리하면 다음과 같다. \
  "호출당하는 쪽의 메서드를 호출하는 쪽의 코드에 대응하도록 중간에 변환기를 통해 호출하는 패턴"

### 어댑터 패턴이 적용되지 않은 코드&#x20;

* `main()` 메서드를 살펴보면 sa, sb 를 통해 호출되는 메서드가 비슷한 일을 하지만, 다른 메서드 명을 사용하고 있는 것을 볼 수 있다.

```java
public class ServiceA {
    void runServiceA() {
        System.out.println("ServiceA");
    }
}

public class ServiceB {
    void runServiceB() {
        System.out.println("ServiceB");
    }
}

public class NoAdapter {
    public static void main(String[] args) {
        ServiceA sa = new ServiceA();
        ServiceB sb = new ServiceB(); 
        
        sa.runServiceA();
        sb.runServiceB(); 
    }
}
```

### 어댑터 패턴이 적용된 코드&#x20;

#### 어댑터 패턴을 적용해 메서드 명을 통일해보자.&#x20;

<pre class="language-java"><code class="lang-java"><strong>public class AdapterA {
</strong><strong>    ServiceA sa = new ServiceA();
</strong><strong>    
</strong><strong>    void runService() {
</strong><strong>        sa.runServiceA(); 
</strong><strong>    }
</strong><strong>}
</strong><strong>
</strong>public class AdapterB {
    ServiceB sa = new ServiceB();
    
    void runService() {
        sa.runServiceB(); 
    }
}

public class NoAdapter {
    public static void main(String[] args) {
        AdapterA sa = new AdapterA();
        AdapterB sb = new AdapterB(); 
        
        sa.runService();
        sb.runService(); 
    }
}
</code></pre>

#### 인터페이스를 토입해 조금 더 개선해보자.

<pre class="language-java"><code class="lang-java"><strong>public interface Adpater {
</strong><strong>    void runService(); 
</strong><strong>}
</strong><strong>
</strong><strong>public class AdapterA implements Adpater{
</strong>    ServiceA sa = new ServiceA();
    
    @Override
    void runService() {
        sa.runServiceA(); 
    }
}

public class AdapterB implements Adpater {
    ServiceB sa = new ServiceB();
    
    @Override
    void runService() {
        sa.runServiceB(); 
    }
}

public class NoAdapter {
    public static void main(String[] args) {
        Adpater sa = new AdapterA();
        Adpater sb = new AdapterB(); 
        
        sa.runService();
        sb.runService(); 
    }
}
</code></pre>

## 2. 프록시 패턴(Proxy Pattern)

* **대리자(인터페이스)를 통해서 메서드를 호출도록 하는 패턴**
* 프록시 패턴 구현의 중요 포인트
  * 대리자는 실제 서비스와 같은 이름으로 메서드를 구현한다. 이 때 인터페이스를 사용한다.
  * 대리자는 실제 서비스에 대한 참조 변수를 갖는다(합성).
  * 대리자는 실제 서비스의 같은 이름을 가진 메서드를 호출하고 그 값을 클라이언트에게 돌려준다.
  * 대리자는 실제 서비스의 메서드 호출 전후의 별도의 로직을 수행할 수 있다.
* **프록시 패턴은 실제 서비스 메서드의 반환 값에 가감하는 것을 목적으로 하지 않고 제어의 흐름을 변경하거나 다른 로직을 수행하기 위해 사용한다.**
  * **실제 서비스 메서드의 반환 값에 가감하는 것을 목적으로 하는 것은 데코레이터 패턴이다.**&#x20;
* OCP(Open Closed Principle) DIP(Dependency Inversion Principle) 원칙을 살펴볼 수 있다.
  * OCP : 배우가 바뀌어도 배역의 역할은 변경되지 않는다.&#x20;
  * DIP : 자동차가 타이어에 의존해야지 스노우 타이어에 의존해서는 안된다.&#x20;

### 프록시 패턴이 적용되지 않은 코드&#x20;

```java
public class Service {
    public String runSomething() {
        return "서비스 짱!";
    }
}

public class ClientWithNoProxy {
    public static void main(String[] args) {
        Service service = new Service();
        System.out.println(service.runSomething()); 
    }
}
```

### 프록시 패턴이 적용된 코드&#x20;

* 프록시 패턴의 경우 실제 서비스가 가진 메서드와 같은 이름의 메서드를 사용하는데, 이를 위해서 인터페이스를 사요한다.&#x20;
* 인터페이스를 사용하면 서비스 객체가 들어갈 자리에 프록시 객체를 대신 투입해 클라이언트 쪽에서는&#x20;
  * **실제 서비스 객체를 통해서 메서드를 호출하고 반환값을 받는지,**&#x20;
  * **아니면 대리자 객체를 통해서 메서드를 호출하고 반환값을 받는지 전혀 모르게 처리할 수 있다.**&#x20;

```java
public interface IService{
    String runSomething();
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
        
        service = new Service();
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

## 3. 데코레이터 패턴(Decorator Pattern)

* 프록시 패턴과 구현 방법은 동일하지만, 목적이 다르다.&#x20;
  * **프록시 패턴 : 제어의 흐름을 변경하거나 별도의 로직 처리를 목적으로 한다. 클라이언트가 받는 반환값을 특별한경우가 아니면 변경하지 않는다.** &#x20;
  * **데코레이터 패턴 : 클라이언트가 받는 반환값에 장식을 더한다.**&#x20;

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
        
        service = new Service();
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

## 4. 싱글턴 패턴(Singleton Pattern)

* **싱글턴 패턴은 인스턴스를 하나만 만들어서 사용하기 위한 패턴이다.**&#x20;
  * 커넥션 풀, 스레드 풀, 디바이스 설정 객체 등과 같은 경우 여러 인스턴스를 만들게 되면 불필요한 자원을 사용하게 되고, 또 예상치 못한 결과를 맞이할 수 있다.&#x20;
  * 싱글턴 패턴은 오직 인스턴스를 하나만 만들고 그것을 계속해서 사용하는 것이다.&#x20;
* 싱글턴 패턴을 적용할 경우 의미상 두 개의 객체가 존재할 수 없다.&#x20;
* 이를 위한 요소를 생각하면 다음과 같다.&#x20;
  * **`new` 를 실행할 수 없도록 생성자에 `private` 접근제어자 사용**&#x20;
  * **유일한 단일 객체를 반환할 수 있는 `static` 메서드 필요** &#x20;
  * **유일한 단일 객체를 참조할 `static` 참조 변수 필요**
* **`static` 을 사용하므로 프로세스 내에서 공유가 가능한 참조 변수를 사용할 수 있게 된다.**&#x20;

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

#### 주의점!&#x20;

* **싱글톤 패턴은 전역 변수와 비슷한 특성을 갖게 되어서 공유 자원을 갖게 된다면 코드의 복잡성을 높일 수 있다.**&#x20;
* **때문에, 싱글톤 객체 내에는 속성을 갖지 않도록! 하는 것이 정석이다.**&#x20;
* **다만 읽기 전용 속성을 갖는 것은 문제가 되지 않는다.**&#x20;

> 싱글톤 패턴이 안티 패턴으로 불리는 이유&#x20;
>
> 1. 싱글톤 패턴은 간단하고 명료한 해결책을 제시하지만, **장기적인 유지보수성과 테스트 가능성에 악영향을 미친다.**&#x20;
> 2. 작은 프로젝트에서는 유용할 수 있으나, **대규모 시스템에서는 전역 상태관리와 결합도 증가 문제로 인해 권장되지 않는다.**&#x20;
> 3. 싱글톤 대신 DI 컨테이너나 팩토리 패턴 등 더 나은 설계 방식을 고려하는 것이 좋다.&#x20;

## 5. 템플릿 메서드 패턴(Template Method Pattern)

* **템플릿 메서드 패턴은 상위 클래스에 골격(틀) 을 정의하고, 구체적인 구현은 하위 클래스에서 제공하는 디자인 패턴.**
  * **추상클래스, 인터페이스를 통해서 구현**
  * "상위 클래스의 템플릿 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴"
* **템플릿 메서드 패턴은 의존 역전 원칙(DIP) 를 활용하고 있음을 알 수 있다.**

### 템플릿 메서드 패턴이 적용되지 않은 코드&#x20;

* 강아지, 고양이 클래스 모두 `playWithOwner()` 메서드 내에서 2번째 문자열 출력을 제외하고는 다른 내용이 없다. \
  (반복된 코드..)&#x20;

```java
public class Dog {

    public void playWithOwner(){
        System.out.println("귀염둥이 이리온");
        System.out.println("멍멍");
        System.out.println("잘했어");
    }
}    

public class Cat {

    public void playWithOwner(){
        System.out.println("귀염둥이 이리온");
        System.out.println("야옹");
        System.out.println("잘했어");
    }
} 
```

### 템플릿 메서드 패턴이 적용된 코드&#x20;

* `Animal` 이라는 추상클래스에서 틀을 만들고, 이를 상속한 하위 클래스에서 구현을 제공한다.&#x20;
* `main` 메서드는 구체화된 클래스(`Dog`, `Cat`)가 아닌, 추상화된 클래스(`Animal`)를 의존하기 때문에 DIP 원칙이 지켜진 것을 확인할 수 있다.&#x20;

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

## 6. 팩터리 메서드 패턴(Factory Method Pattern)

* 팩터리는 공장을 의미한다. 공장은 물건을 생산하는데, **객체지향의 세계에서는 공장은 객체를 생산한다.**&#x20;
* **결국, 팩터리 메서드는 객체를 생성 반환하는 메서드를 의미한다.**&#x20;
* **여기에, 패턴이 붙으면 하위 클래스에서 팩터리 메서드를 오버라이딩해서 객체를 반환하게 하는 것을 의미한다.**&#x20;
* **팩토리 메서드 패턴 또한 의존 역전 원칙(DIP) 를 활용하고 있음을 알 수 있다.**

```java
public abstract class Animal{
    // 추상 팩터리 메서드 
    abstract AnimalToy getToy();
}

// 팩터리 메서드가 생성할 객체의 상위 클래스 
public abstract class AnimalToy{
    abstract void identify();
}

// 팩터리 메서드가 생성할 객체 
public class DogToy extends AnimalToy{
    public void identify(){
        System.out.println("나는 테니스공, 강아지의 친구");
    }
}

// 팩터리 메서드가 생성할 객체 
public class CatToy extends AnimalToy{
    public void identify(){
        System.out.println("나는 캣타워, 고양이의 친구");
    }
}

public class Dog extends Animal(){
    // 추상 팩터리 메서드 오버라이딩 
    @Override
    AnimalToy getToy(){
        return new DogToy();
    }
}

public class Cat extends Animal(){
    // 추상 팩터리 메서드 오버라이딩 
    @Override
    AnimalToy getToy(){
        return new CatToy();
    }
}

public class Driver{
    public static void main(String[] args){
        // 팩터리 메서드를 보유한 객체들 생성 
        Animal bolt = new Dog();
        Animal kitty = new Cat();
        
        // 팩터리 메서드가 반환하는 객체들 
        AnimalToy boltBall = bolt.getToy();
        AnimalToy kittyTour = kitty.getToy();
        
        // 팩터리 메서드가 반환한 객체들 사용
        boltBall.identify();
        kittyTour.identify();
    }
}
```

## 7. 전략 패턴(Strategy Pattern)

* 전략 패턴을 구성하는 요소 세가지
  * **전략 메서드를 가진 전략 객체**\
    \-> 각각의 무기. 총, 검, 활 등등 ..
  * **전략 객체를 사용하는 컨텍스트(전략 객체의 사용자/소비자)**\
    \-> 무기를 사용할 군인
  * **전략 객체를 생성해 컨텍스트에 주입하는 클라이언트(제3자, 전략 객체의 공급자)**\
    \-> 군인에게 무기를 전달해 줄 보급 장교
* 같은 문제를 템플릿 메서드 방법을 통해서도 해결할 수 있다.
  * **템플릿 메서드 : 상속을 이용**
  * **전략 패턴 : 객체 주입을 이용**
* **"클라이언트가 전략을 생성해서 전략을 실행한 컨텍스트에 주입하는 패턴"**
* **전략 메서드 패턴은 개방 폐쇄 원칙(OCP), 의존 역전 원칙(DIP) 를 활용하고 있음을 알 수 있다.**

<pre class="language-java"><code class="lang-java">// 전략 인터페이스 
public interface Strategy{
    public abstract void runStrategy();
<strong>}
</strong>
// 전략 - 총 
public class StrategyGun implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("탕탕탕");
    }
}

// 전략 - 칼
public class StrategySword implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("챙챙");
    }
}

// 전략 - 활
public class StrategBow implements Strategy{
    @Override
    public void runStrategy(){
        System.out.println("슉슉");
    }
}

// 전략을 사용할 객체 - 군인 
public class Soldier{
    void runContext(Strategy strategy){
        System.out.println("전투 시작");
        strategy.runStrategy();
        System.out.println("전투 종료");
    }
}

// 전략을 생성하고 주입해줄 객체 - 장교 
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
</code></pre>

## 8. 템플릿 콜백 패턴

* 템플릿 콜백 패턴은 전략 패턴의 변형으로, 스프링의 3대 프로그래밍 모델 중 하나인 DI(의존성 주입) 에서 사용되는 특별한 형태의 전략 패턴이다.&#x20;
* **템플릿 콜백 패턴은 전략 패턴과 모든 것이 동일한데, 전략을 익명 내부 클래스로 정의해서 사용한다는 특징이 있다.**&#x20;
* **템플릿 콜백 패턴 또한 개방 폐쇄 원칙(OCP), 의존 역전 원칙(DIP) 를 활용하고 있음을 알 수 있다.**

```java
// 전략 인터페이스 
public interface Strategy{
    public abstract void runStrategy();
}

// 전략이 군인 내부로 들어왔다. 
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
