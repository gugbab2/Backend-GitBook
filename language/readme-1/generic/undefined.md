# 제네릭 기본

## 자바의 공변성 / 반공변성&#x20;

* 자바의 제네릭과 와일드카드에 대해서 이해하기 전에, 공변과 불공변에 대해서 알아야 한다.&#x20;
  * 공변 : A 가 B 의 하위 타입일 때, T\<A> 가 T\<B> 의 하위 타입이면 공변 (업캐스팅!)&#x20;
  * 불공변 : A 가 B 의 하위 타입일 때, T\<A> 가 T\<B> 의 하위 타입이 아니면 불공변 (다운캐스팅!)&#x20;
* 대표적으로 배열은 공변, 제네릭은 불공변인데 이를 코드로 살펴보자.
* 아래 코드에서 배열은 공변이기 때문에, Integer 은 Object 의 하위 타입이므로, Integer 배열은 Object 배열의 하위 타입이다. &#x20;

```java
@Test
void genericTest() {
    Integer[] integers = new Integer[]{1, 2, 3};
    printArray(integers);
}

void printArray(Object[] arr) {
    for (Object e : arr) {
        System.out.println(e);
    }
}
```

* 하지만 제네릭은 불공변이어서 제네릭을 사용하는 컬렉션 코드는 컴파일 에러가 발생한다. \
  \-> **제네릭은 불공변이기 때문에, List\<Integer>, List\<Objcet> 두 타입의 아무런 관계가 없다.**&#x20;

```java
@Test
void genericTest() {
    List<Integer> list = Arrays.asList(1, 2, 3);
    printCollection(list);   // 컴파일 에러 발생
}

void printCollection(Collection<Object> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```

### 제네릭과 와일드카드를 쉽고 완벽하게 이해하기&#x20;

#### 제네릭 등장 이전&#x20;

* 제네릭은 JDK 1.5 부터 등장하였는데, 제네릭이 존재하기 전 컬렉션 요소를 출력하는 메서드는 다음과 같이 만들 수 있었다.&#x20;
* 하지만 아래 코드와 같이, 컬렉션 요소들을 다루는 메서드들은 타입이 보장되지 않아서, 문제가 발생하고는 했다.&#x20;

```java
void printCollection(Collection c) {
    Iterator i = c.iterator();
    for (k = 0; k < c.size(); k++) {
        System.out.println(i.next());
    }
}
```

* 예를 들어서 컬렉션 요소들의 합을 구하는 메서드를 아래와 같이 구현한다고 해보자.&#x20;
* 문제는 아래 메서드에서 String 같이 다른 타입을 갖는 컬렉션도 호출이 가능하다는 점이다.&#x20;
  * **String 타입을 같는 컬렉션은 컴파일 타임에는 문제가 없다가, 런타임에 타입이 맞지 않아 문제가 발생했다.**&#x20;
  * **그래서 Java 개발자들은 타입을 지정하여 컴파일 타임에 타입안정성을 제공받고자 했다.**&#x20;

```java
int sum(Collection c) {
    int sum = 0;
    Iterator i = c.iterator();
    for (k = 0; k < c.size(); k++) {
        sum += Integer.parseInt(i.next());
    }
    return sum;
}
```

#### 제네릭 등장 이후&#x20;

* 제네릭이 등장 하면서, 컬렉션 타입을 지정할 수 있게 되었고, 위 메서드를 다음과 같이 수정할 수 있게 되었다.&#x20;
* 이제는 다른 타입을 가진 컬렉션이 해당 메서드를 호출하게 되면 **컴파일 에러를 통해 타입 안정성을 보장 받을 수 있게 되었다.**&#x20;

```java
void sum(Collection<Integer> c) {
    int sum = 0;
    for (Integer e : c) {
        sum += e;
    }
    return sum;
}
```

* 하지만 제네릭은 불공변이기 때문에, 또 다른 문제가 발생하는데, 아래 `printCollection` 처럼 모든 타입에 공통적으로 사용되는 메서드를 만들 수 없다는 것이다.&#x20;
* **매개변수 타입을 Integer 에서 Object 로 변경하여도, 제네릭은 불공변이기 때문에, Collection\<Integer> 는 Collection\<Object> 의 하위 타입이 아니다. 이로 인해 컴파일 에러가 발생한다.**&#x20;
* 이러한 상황은 제네릭이 추가되기 전 보다 오히려, 실용성이 떨어졌기 때문에, 와일드카드라는 타입이 추가되었다.&#x20;

```java
@Test
void genericTest() {
    List<Integer> list = Arrays.asList(1, 2, 3);
    printCollection(list);   // 컴파일 에러
}

void printCollection(Collection<Object> c) {
    for (Object e : c) {
        System.out.println(e);
    }
}
```
