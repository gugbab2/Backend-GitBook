# 다형성의 활용 - 인터페이스

## 인터페이스&#x20;

* **순수 추상클래스를 더욱 편리하게 사용할 수 있도록 하는 기능을 인터페이스라고 한다.**&#x20;
* 인터페이스는 다음과 같은 특징을 가지고 있다.&#x20;
  * 순수 추상클래스의 특징
    * 인스턴스를 생성할 수 없다.(메서드의 구현체가 없기 때문에..)
    * 상속 시 모든 메서드를 오버라이딩 해야 한다.&#x20;
  * **추가적인 특징**&#x20;
    * **메서드에 `public abstract` 를 생략할 수 있다.&#x20;**_**참고로 생략이 권장된다.**_&#x20;
    * **모든 인스턴스변수는 `public static final` 이어야 하며, 이를 생략할 수도 있다.**
    * **인터페이스 다중 상속을 지원한다.**

#### 인터페이스를 사용한 동물 울음소리 코드 개선

```java
public interface InterfaceAnimal {
    void sound();
    void move();
}

public class InterfaceMain {
    public static void main(String[] args) {
        Cat cat = new Cat();
        Dog dog = new Dog();
        Cow cow = new Cow();

        animalSound(cat);
        animalSound(dog);
        animalSound(cow);

    }
    private static void animalSound(InterfaceAnimal animal) {
        System.out.println("동물 소리 시작");
        animal.sound();
        System.out.println("동물 소리 종료");
    }
}
```

### 상속 vs 구현

* 부모 클래스의 기능을 자식클래스가 상속 받을 때, 클래스는 상속 받는다고 표현하지만,\
  -> **상속은 이름 그대로 부모의 기능을 물려 받는 것이 목적이다.**&#x20;
* 부모 인터페이스의 기능을 자식이 상속 받을 때는, 인터페이스를 구현한다고 표현한다. \
  &#xNAN;**-> 인터페이스는, 모든 메서드가 추상메서드이다. 따라서 물려받을 기능이 없고, 오히려 인터페이스에서 정의한 모든 메서드를 자식이 오버라이딩 해서 기능을 구현해야 한다.** \
  **-> 따라서, 구현하다고 표현한다.**&#x20;
* **인터페이스는 메서드 이름만 있는 설계도이고, 실제 어떻게 작동하는지는 하위 클래스에서 모두 구현해야 한다.**&#x20;

### **인터페이스를 사용해야 하는 이유**

#### 제약

* 인터페이스를 만드는 이유는, 인터페이스를 구현하는 곳에서 인터페이스 메서드를 반드시 구현하라는 제약을 주는 것이다.&#x20;
* 하지만, 순수 추상 클래스의 경우 미래의 누군가가 실행 가능한 메서드를 끼워넣을 수 있다.\
  -> 이런 경우 제약이라는 장점이 사라지게 된다.

#### 다중 구현&#x20;

* 자바가 클래스 다중 상속이 문제가 되는 경우
  * 만약 두 조상으로부터 상속 받는 멤버 중 멤버변수의 이름이 같거나, 메서드의 선언부가 일치하고 구현 내용이 다르다면 이 두 조상으로부터 상속받는 자손 클래스는 어느 조상의 것을 상속받게 되는지 알 수 없다.\
    (다이아몬드 상속)
* 인터페이스가 다중 상속을 지원하는 이유
  * **인터페이스에는 구현체가 없고, 인터페이스를 상속받는 클래스에 구현체가 있기 때문에 다이아몬드 상속 문제가 없다.** \
    **-> 여러개의 인터페이스를 상속 받더라도 구현체는 하나일 뿐이다.**

### **클래스와 인터페이스를 활용하는 방법**

* 다음과 같이 추상클래스와 인터페이스를 함께 구현할 수 있다.

```java
public abstract class AbstractAnimal {
    public abstract void sound();
    public void move() {
        System.out.println("동물이 이동합니다.");
    }
}

public interface Fly {
    void fly();
}

public class SoundFlyMain {
    public static void main(String[] args) {
        Dog dog = new Dog();
        Bird bird = new Bird();
        Chicken chicken = new Chicken();

        animalSound(dog);
        animalSound(bird);
        animalSound(chicken);

        animalFly(bird);
        animalFly(chicken);
    }

    private static void animalSound(AbstractAnimal animal) {
        System.out.println("동물 소리 시작");
        animal.sound();
        System.out.println("동물 소리 종료");
    }

    private static void animalFly(Fly fly) {
        System.out.println("동물 날기 시작");
        fly.fly();
        System.out.println("동물 날기 종료");
    }
}


```
