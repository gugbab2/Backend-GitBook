# 와일드카드

## 와일드카드 등장

* 제네릭이 등장했지만 오히려 실용성이 떨어지는 상황들이 발생하면서, 모든 타입을 대신할 수 있는 와일드카드 타입을 추가하였다.&#x20;
* 와일드카드는 컴퓨터프로그래밍에서 `?` 와 같이 하나 이상의 문자들을 상징하는 특수 문자를 뜻한다. 이는 여러 타입이 들어올 수 있음을 의미하는데, 이미 만들어진 제네릭 타입을 기반으로 활용할 때 사용된다.&#x20;

```java
// 제네릭 메서드
public <T> void generitMethod(Box<T> box) {
     System.out.println("T = " + box.get());
}
 
// 와일드카드를 활용한 일반적인 메서드
public void wildcardMethod(Box<?> box) {
     System.out.println("? = " + box.get());
}
```

* **와일드카드인 ? 는 정해지지 않은 임의의 타입을 나타내며**, 타입 제한이 없어 이를 **비제한 와일드카드 타입**이라고 한다.&#x20;
* 비제한 와일드카드 타입을 통해서 아래와 같이 제네릭의 활용성을 높일 수 있게 되었다. 자바에서 모든 부모의 최상위 객체는 Object 이므로 다음과 같은 코드를 작성할 수 있게 되었다.&#x20;

```java
void printCollection(Collection<?> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

* 하지만 요소를 추가하는 경우는 다르다. **비제한 와일드카드인 `?` 타입은 Unknown 타입으로 컴파일러가 구체적으로 어떤 타입인지 알 수 없다. 이 컬렉션에 요소를 추가하는 것을 허용하면 타입 안정성을 보장할 수 없기 때문에, 요소를 추가하는 연산은 허용하지 않는다.**&#x20;

```java
@Test
void genericTest() {
    Collection<?> c = new ArrayList<String>();
    c.add(new Object()); // 컴파일 에러
}
```

* 즉, `add()` 메서드로 컬렉션의 데이터를 추가하기 위해서는 제네릭 타입, 또는 제네릭 타입의 자식 타입을 넣어주어야 하는데, **비제한 와일드 카드는 Unknown 타입으로 범위가 무제한이다. 이는 타입이 정해지지 않았으므로, 타입체크가 불가하다.**&#x20;
* 반면, `get()` 메서드로 컬렉션에서 데이터를 꺼내는 것은 문제가 없다. 왜냐면, 꺼낸 결과가 Unknown 타입이어도 타입체크가 필요하지 않으며, 심지어 적어도 Object 타입인 것은 확실할 수 있기 때문이다.&#x20;
* 결국, **이는 와일드카드 타입이 Unknown 타입이기 때문이다.**&#x20;
* 이처럼 자바 제네릭은 기본적으로 공변, 반공변을 지원하지 않지만 `<? extends T>` , `<? super T>` 와일드카드를 사용하면 컴파일러 트릭을 통해 공변, 반공변이 적용될 수 있도록 했다.&#x20;
  * 상한 경계 와일드카드 `<? extends T>` : 공변성 적용&#x20;
  * 하한 경계 와일드카드 `<? super T>` : 반공변성 적용

## 한정적 와일드카드&#x20;

### 상한 경계 와일드카드 (공변)&#x20;

* 아래 코드에서 `MyArrayList` 를 설계한 개발자의 의도는 `Collection<Integer>`, `Collection<Double>` 객체를 생성자의 인수로 모두 받아서 배열에 넣고 싶은 것이다.&#x20;
* 이를 위해서 제네릭에 상한 경계 와일드카드를 적용시킨다.&#x20;
* **상한 경계로 지정한 타입이 저장할 수 있는 최대 한도라는 의미이다.**&#x20;
* 그러면, 공변 성질이 적용되어서 컴파일 에러 없이, 정상적으로 `MyArrayList` 에 요소가 들어가 생산되었음을 확인할 수 있다.&#x20;

```java
class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;

    // 외부로부터 리스트를 받아와 매개변수의 모든 요소를 내부 배열에 추가하여 인스턴스화 하는 생성자
    public MyArrayList(Collection<? extends T> in) {
        for(T elem : in) {
            element[index++] = elem;
        }
    }
    
    // ...
}

public static void main(String[] args) {
    // MyArrayList의 제네릭 T 타입은 Number
    MyArrayList<Number> list;

    // MyArrayList 생성하기
    Collection<Integer> col = Arrays.asList(1, 2, 3, 4, 5);
    list = new MyArrayList<>(col);
    
    // MyArrayList 출력
    System.out.println(list); // [1, 2, 3, 4, 5]
}
```

* 즉 `Integer` 가 `Object` 의 하위 타입일 경우, `C<Integer>` 는 `C<? extends Object>` 의 하위 타입이 되는 것이다.&#x20;

```java
ArrayList<? extends Object> parent = new ArrayList<>();
ArrayList<? extends Integer> child = new ArrayList<>();

parent = child;    // 공변성! (제네릭 타입 업캐스팅)
```

### 하한 경계 와일드카드(반공변)&#x20;

* &#x20;파라미터가 무엇이든 인자로 받은 컬렉션 매개변수에 요소들을 집어넣고 싶은 것이다.&#x20;
* 이를 위해 제네릭에 하한 경계 와일드카드를 적용시킨다.&#x20;
* **하한 경계로 지정한 타입이 저장할 수 있는 최저 한도라는 의미이다.**&#x20;
* 그러면 반공변 설징일 적용되어 컴파일 에러 없이 정상적으로 MyArrayList 의 요소가 들어가 소비되었음을 확인할 수 있다.&#x20;

<pre class="language-java"><code class="lang-java">class MyArrayList&#x3C;T> {
    Object[] element = new Object[5];
    int index = 0;

    // 외부로부터 리스트를 받아와 매개변수의 모든 요소를 내부 배열에 추가하여 인스턴스화 하는 생성자
<strong>    public MyArrayList(Collection&#x3C;? extends T> in) {
</strong>        for (T elem : in) {
            element[index++] = elem;
        }
    }

    // 외부로부터 리스트를 받아와 내부 배열의 요소를 모두 매개변수에 추가해주는 메서드
    public void clone(Collection&#x3C;? super T> out) {
        for (Object elem : element) {
            out.add((T) elem);
        }
    }

    @Override
    public String toString() {
        return Arrays.toString(element); // 배열 요소들 출력
    }
}

public static void main(String[] args) {
    // MyArrayList의 제네릭 T 타입은 Number
    MyArrayList&#x3C;Number> list = new MyArrayList&#x3C;>(Arrays.asList(1, 2, 3, 4, 5));

    // LinkedList 에 MyArrayList 요소들 복사하기
    List&#x3C;Object> temp = new LinkedList&#x3C;>();
    list.clone(temp);

<strong>    // LinkedList 출력
</strong>    System.out.println(temp); // [1, 2, 3, 4, 5]
}
</code></pre>

* 즉, Object 가 Integer 의 상위 타입일 경우, C\<Object> 는 C\<? super Integer> 의 하위 타입이 되는 것이다.&#x20;

```java
ArrayList<? super Object> parent = new ArrayList<>(); 
ArrayList<? super Integer> child = new ArrayList<>(); 

child = parent;    // 반공변성 (제네릭 다운캐스팅) 
```

* 어디다 써야할지도 애매한 이런 거꾸로의 상속 관계는 위 사례와 같이, 인스턴스화 된 클래스의 제네릭보다 상위 타입의 데이터를 적재해야 할 때 반공변을 사용한다고 이해하면 된다.&#x20;

> 자바 제네릭은 기본적으로 변성이 없지만, 한정적 와일드카드 타입을 통해서 타입의 공변성(extends) 또는 반공변성(super) 을 지정할 수 있다.&#x20;
>
> 이렇게 타입 매개변수 지점에 변성을 정하는 자바의 방식을 **'사용자 지점 변성'** 이라고 한다.&#x20;
>
> 자바와 같이 JVM 을 이용하는 코틀린에서도 사용자 지점 변성을 제공한다.&#x20;

