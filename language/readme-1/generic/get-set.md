# 와일드카드 GET / SET 경계

## List\<? extends U>&#x20;

#### 결론

* **GET : 안전하게 꺼내려면 U 타입으로 받아야 한다.**&#x20;
* **SET : 어떠한 타입의 자료도 넣을 수 없음(null 만 삽입 가능)**
* **꺼낸 타입은 U / 저장은 NO**

#### 이유&#x20;

* 메서드의 매개변수 제네릭 타입이 `<? extends Fruit>` 라는 것은 `Apple, Banana, Fruit` 타입을 전달 받아 내부에서 다룰 수 있다는 점이다.&#x20;
* 그런데 왜 GET 을 왜 Fruit 로 받아야 하냐면,&#x20;
  * **만약, 매개변수에 `List<Banana>` 타입으로 들어올 경우, `Apple f2` 에 형제 캐스팅이 불가하기 때문이다.**&#x20;
  * **때문에, 이러한 논리 오류로 와일드카드 최상위 범위인 `Fruit` 타입으로만 안전하게 꺼낼 수 있는 것이다.**&#x20;

```java
class FruitBox {
    public static void method(List<? extends Fruit> item) {
        // 안전하게 꺼내려면 Fruit 타입으로만 받아야한다
        Fruit f1 = item.get(0);

        Apple f2 = (Apple) item.get(0); // ! 잠재적 ERROR
        Banana f3 = (Banana) item.get(0); // ! 잠재적 ERROR
    }
}

public class Main {
    public static void main(String[] args) {
        List<Banana> bananas = new ArrayList<>(
                Arrays.asList(new Banana(), new Banana(), new Banana())
        );
        FruitBox.method(bananas);
    }
}
```

* 또한, SET 을 어떠한 타입도 불가능하냐면
  * **만일 매개변수에 `List<Banana>` 타입으로 들어올 경우 형제 객체인 `new Apple()` 저장이 불가능하기 때문이다.**&#x20;
  * **만일 매개변수에 `List<Fruit>` 타입으로 들어올 경우 문제는 없겠지만, 위의 논리 오류로 그냥 컴파일 에러로 처리된다.**&#x20;
  * **따라서 만일 매개변수에 값을 넣고 싶다면 무조건 `super` 와일드카드를 사용해야 한다.**&#x20;

```java
class FruitBox {
    public static void method(List<? extends Fruit> item) {
        // 저장은 NO
        item.add(new Fruit()); // ! ERROR
        item.add(new Apple()); // ! ERROR
        item.add(new Banana()); // ! ERROR
        
        item.add(null); // null 만 삽입 가능
    }
}
```

## List\<? super U>

#### 결론

* **GET : 안전하게 꺼내려면 Object 타입으로만 받아야 한다.**&#x20;
* **SET : U와 U의 자손 타입만 넣을 수 있다.(U의 상위 타입 불가능)**
* **꺼낸 타입은 Object / 저장은 U 와 그의 자손만**

#### 이유&#x20;

* 메서드의 매개변수의 제네릭 타입이 `<? super Fruit>` 라는 것은 `Fruit, Food, Object` 타입을 전달 받아 내부에서 다룰 수 있다는 것이다.
* 그런데 왜 GET 을 `Object` 타입으로 받아야 되냐면,&#x20;
  * 만일 매개변수에 `List<Food>` 타입이 들어올 경우 `Fruit f3` 에 캐스팅이 불가하기 때문이다.&#x20;
  * 이러한 논리 오류로 와일드카드 최상위 범위인 `Object` 타입으로만 안전하게 꺼낼 수 있는 것이다.&#x20;

```java
class FruitBox {
    public static void method(List<? super Fruit> item) {
        // 안전하게 꺼내려면 Object 타입으로만 받아야한다
        Object f1 = item.get(0);

        Food f2 = (Food) item.get(0); // ! 잠재적 ERROR
        Fruit f3 = (Fruit) item.get(0); // ! 잠재적 ERROR
        Apple f4 = (Apple) item.get(0); // ! ERROR
        Banana f5 = (Banana) item.get(0); // ! ERROR
    }
}

public class Main {
    public static void main(String[] args) {
        List<Food> foods = new ArrayList<>(
                Arrays.asList(new Food(), new Food(), new Food())
        );
        FruitBox.method(foods);
    }
}
```

* 또한, SET 을 거꾸로 Fruit 와 그의 자손 타입만 올 수 있냐면,&#x20;
  * 만일 매개변수에 `List<Fruit>` 타입으로 들어올 경우 `new Food()` 저장이 불가능하기 때문이다.&#x20;
  * 따라서 어떠한 타입이 와도 업캐스팅이 가능 상한인 `Fruit` 타입으로만 제한된다.

```java
class FruitBox {
    static void method(List<? super Fruit> item) {
        // 저장은 무조건 Fruit와 그의 자손 타입만 넣을수 있다
    	item.add(new Fruit());
        item.add(new Apple());
        item.add(new Banana());
        
        item.add(new Object()); // ! ERROR
        item.add(new Food()); // ! ERROR
    }
}
```

## List\<?>&#x20;

#### 결론

* **GET : 안전하게 꺼내려면 Object 타입으로만 받아야 한다. -> super 의 특징**
  * 비제한 와일드카드는 어떤 타입이 들어올지 모르기 때문에, 최상위 타입인 Object 로 받아야 한다.&#x20;
* **SET : 어떠한 타입의 자료도 넣을 수 없음(null 만 삽입 가능) -> extends 의 특징**&#x20;
  * 넣어야 하는 타입이 정해지지 않았는데? 값을 넣을수가 있나?
* **꺼낸 타입은 Object / 저장은 NO**

#### 이유

```java
class FruitBox {
    static void method(List<?> item) { 
        // 꺼내는건 Object 타입만 가능
        Object f1 = item.get(0);

        Food f2 = (Food) item.get(0); // ! 잠재적 ERROR
        Fruit f3 = (Fruit) item.get(0); // ! 잠재적 ERROR
        Apple f4 = (Apple) item.get(0); // ! 잠재적 ERROR
        Banana f5 = (Banana) item.get(0); // ! 잠재적 ERROR
    }
}
```

```java
class FruitBox {
    static void method(List<?> item) {
        // 저장은 NO (null만 저장)
        item.add(new Object()); // ! ERROR
        item.add(new Food()); // ! ERROR
        item.add(new Fruit()); // ! ERROR
        item.add(new Apple()); // ! ERROR
        item.add(new Banana()); // ! ERROR

        item.add(null);
    }
}
```
