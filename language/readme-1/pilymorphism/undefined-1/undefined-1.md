# 다형성의 활용 - 추상클래스

## 추상클래스

### 추상클래스란?

* 객체지향 프로그래밍에서 클래스가 설계도라면, 추상클래스는 미완성 설계도로 비유할 수 있다.
* 추상클래스는 미완성 클래스로 인스턴스를 생성할 수 없고, 상속을 통해서 자손 클래스에서 완성된다.
* 클래스 선언부에 abstract 키워드가 있다면 "추상메서드가 포함되어 있는 추상클래스다." 라고 할 수 있다.
* **이를 통해 클래스간 공통점을 추상클래스에서 구현하고, 차이점은 이를 상속 받은 클래스에서 추상메서드를 구현함으로 확장성 있는 코드를 작성할 수 있다.**\
  **-> 이를 통해서 자손 클래스에서 추상 메서드의 구현을 강요할 수 있다.**\
  **-> default 키워드를 통해서 강요하지 않을 수도 있다.**

#### **추상클래스를 사용한 동물 울음소리 코드 개선**

* 추상클래스를 통해서 sound() 메서드의 구현을 강제하자

```java
public abstract class AbstracAnimal {
    public abstract void sound();
    
    public void move(){
        System.out.println("동물이 움직입니다.");
    }
}

public class AnimalSoundMain {
    public static void main(String[] args) {

        AbstracAnimal[] animalArr = {new Dog(), new Cat(), new Cow(),};

        for(AbstracAnimal animal : animalArr){
            animalSound(animal);
        }
    }

    // 변하지 않는 부분은 메서드로..
    private static void animalSound(AbstracAnimal animal) {
        System.out.println("동물 소리 시작");
        animal.sound();
        System.out.println("동물 소리 종료");
    }
}
```

### 순수 추상클래스

* 모든 메서드가 추상메서드로 이루어진 클래스를 순수 추상클래스라고 한다.&#x20;
* **순수 추상클래스의 느낌을 생각해보면, 부모의 기능을 상속 받는 것이 아니라, 마치 어떤 규격을 맞추어야 하는 **_**인터페이스**_** 처럼 보여진다.**\
  **-> 이런 순수 추상클래스를 더욱 편리하게 사용할 수 있도록 하는 기능을 인터페이스라고 한다.**&#x20;

```java
public abstract class AbstracAnimal {
    public abstract void sound();
    public abstract void move();
}

public class AnimalSoundMain {
    public static void main(String[] args) {

        AbstracAnimal[] animalArr = {new Dog(), new Cat(), new Cow(),};

        for(AbstracAnimal animal : animalArr){
            animalSound(animal);
            animalMove(animal);
        }
    }

    // 변하지 않는 부분은 메서드로..
    private static void animalSound(AbstracAnimal animal) {
        System.out.println("동물 소리 시작");
        animal.sound();
        System.out.println("동물 소리 종료");
    }

    private static void animalMove(AbstracAnimal animal) {
        System.out.println("동물 움직이기 시작");
        animal.move();
        System.out.println("동물 움직이기 종료");
    }
}

```
