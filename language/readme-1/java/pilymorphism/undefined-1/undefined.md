# 다형성의 활용 - 기본

## 다형성의 활용

#### 다형성을 활용하지 않고, 다음과 같이 동물 울음소리를 내는 코드를 만들어보자 -> 반복이 너무나 많다..

```java
package poly.ex1;

public class AnimalSoundMain {
    public static void main(String[] args) {
        Dog dog = new Dog();
        Cat cat = new Cat();
        Cow cow = new Cow();

        System.out.println("동물 소리 테스트 시작");
        dog.sound();
        System.out.println("동물 소리 테스트 종료");

        System.out.println("동물 소리 테스트 시작");
        cat.sound();
        System.out.println("동물 소리 테스트 종료");

        System.out.println("동물 소리 테스트 시작");
        cow.sound();
        System.out.println("동물 소리 테스트 종료");
    }
}

```

#### 다형성을 활용해 중복되는 코드를 메서드로 분리해보자

```java
public class AnimalSoundMain {
    public static void main(String[] args) {
        Animal dog = new Dog();
        Animal cat = new Cat();
        Animal cow = new Cow();
        Animal duck = new Duck();

        soundAnimal(dog);
        soundAnimal(cat);
        soundAnimal(cow);
        soundAnimal(duck);
    }

    public static void soundAnimal(Animal animal){
        System.out.println("동물 소리 테스트 시작");
        animal.sound();
        System.out.println("동물 소리 테스트 종료");
    }
}
```

#### 배열을 통해 반복되는 코드와 고정된 코드를 분리해보자

\-> 하지만 Animal 클래스를 상속받는 자식 클래스에서, Sound() 메서드를 오버라이딩 하지 않을 수도 있다는 문제가 있다. \
\-> 이 문제를 추상클래스를 통해서 개선할 수 있다.

```java
public class AnimalSoundMain2 {
    public static void main(String[] args) {

        Animal[] animalArr = {new Dog(), new Cat(), new Cow(), new Duck()};

        for(Animal animal : animalArr){
            animalSound(animal);
        }
    }

    // 변하지 않는 부분은 메서드로..
    private static void animalSound(Animal animal) {
        System.out.println("동물 소리 시작");
        animal.sound();
        System.out.println("동물 소리 종료");
    }
}
```
