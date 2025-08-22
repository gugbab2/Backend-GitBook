# 7. 컬렉션 프레임워크 - HashSet

## 직접 구현하는 Set1 - MyHashSetV1

#### `Set` 은 중복을 허용하지 않는 자료구조이다.

`Set` 을 구현하는 방법은 단순하다. 인덱스가 없기 때문에 단순히 데이터를 저장하고, 데이터가 있는지 확인하고, 데이터를 삭제하는 정도면 충분하다. 그리고 `Set` 은 중복을 허용하지 않기 때문에, 데이터를 추가할 때 중복 여부만 체크하면 된다.&#x20;

* `add(value)` : 셋에 값을 추가한다. 중복 데이터는 저장하지 않는다.&#x20;
* `contains(value)` : 셋에 값이 있는지 확인한다.
* `remove(value)` : 셋에 있는 값을 제거한다.

### 코드 구현 (해시 알고리즘 사용)&#x20;

```java
package collection.set;

import java.util.Arrays;
import java.util.LinkedList;

public class MyHashSetV1 {

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    LinkedList<Integer>[] buckets;

    private int size = 0;
    private int capacity = DEFAULT_INITIAL_CAPACITY;

    public MyHashSetV1() {
        initBuckets();
    }

    public MyHashSetV1(int capacity) {
        this.capacity = capacity;
        initBuckets();
    }

    private void initBuckets() {
        buckets = new LinkedList[capacity];
        for (int i = 0; i < capacity; i++) {
            buckets[i] = new LinkedList<>();
        }
    }

    public boolean add(int value) {
        int hashIndex = hashIndex(value);
        LinkedList<Integer> bucket = buckets[hashIndex];
        if (bucket.contains(value)) {
            return false;
        }
        bucket.add(value);
        size++;
        return true;
    }

    public boolean contains(int searchValue) {
        int hashIndex = hashIndex(searchValue);
        LinkedList<Integer> bucket = buckets[hashIndex];
        return bucket.contains(searchValue);
    }

    public boolean remove(int value) {
        int hashIndex = hashIndex(value);
        LinkedList<Integer> bucket = buckets[hashIndex];
        boolean result = bucket.remove(Integer.valueOf(value));
        if (result) {
            size--;
            return true;
        } else {
            return false;
        }
    }

    private int hashIndex(int value) {
        return value % capacity;
    }

    public int getSize() {
        return size;
    }

    @Override
    public String toString() {
        return "MyHashSetV1{" +
                "buckets=" + Arrays.toString(buckets) +
                ", size=" + size +
                ", capacity=" + capacity +
                '}';
    }
}
```

```java
package collection.set;

public class MyHashSetV1Main {

    public static void main(String[] args) {
        MyHashSetV1 set = new MyHashSetV1(10);
        set.add(1);
        set.add(2);
        set.add(5);
        set.add(8);
        set.add(14);
        set.add(99);
        set.add(9);
        System.out.println("set = " + set);

        // 검색
        int searchValue = 9;
        boolean result = set.contains(searchValue);
        System.out.println("set.contains(" + searchValue + ") : " + result);

        // 삭제
        boolean removeValue = set.remove(searchValue);
        System.out.println("set.remove(" + searchValue + ") : " + result);
        System.out.println("set = " + set);
    }
}
```

### 정리&#x20;

* **생성** : `new MyHashSetV1(10)` 을 사용해서 배열의 크기를 10으로 지정했다. \
  (여기서는 기본 생성자를 사용하 지 않았다.)
* **저장** : 실행결과를보면 `99`, `9` 의 경우 해시 인덱스가 9로 충돌하게 된다. 따라서 배열의 같은 9번 인덱스 위치에  저장된 것을 확인할 수 있다. 그리고 그 안에 있는 연결리스트에 `99`, `9` 가 함께 저장된다.
* **검색** : `9` 를 검색하는 경우 해시 인덱스가 `9` 이다. 따라서 배열의 `9`번 인덱스에 있는 연결 리스트를 먼저 찾는다.\
  해당 연결 리스트에 있는 모든 데이터를 순서대로 비교하면서 `9` 를 찾는다.
  * 먼저 `99` 와 `9` 를 비교한다. -> 실패&#x20;
  * 다음으로 `9` 와 `9` 를 비교한다. -> 성공

`MyHashSetV1` 은 해시 알고리즘을 사용한 덕분에 등록, 검색, 삭제 모두 평균 O(1)로 연산 속도를 크게 개선했다.

#### 남은 문제&#x20;

해시 인덱스를 사용하려면 데이터의 값을 배열의 인덱스로 사용해야 한다.&#x20;

그런데 배열의 인덱스는 `0`, `1`, `2` 같은 숫자만 사용할 수 있다. \
`"A"`, `"B"`와 같은 문자열은 배열의 인덱스로 사용할 수 없다.

다음 예와 같이 숫자가 아닌 문자열 데이터를 저장할 때, 해시 인덱스를 사용하려면 어떻게 해야할까?

```java
MyHashSetV1 set = new MyHashSetV1(10);
set.add("A");
set.add("B");
set.add("HELLO");
```

## 문자열 해시 코드&#x20;

### 구현 코드 (문자열 숫자 변경)

모든 문자는 본인만의 고유한 숫자로 표현할 수 있다. 예를 들어서 `'A'` 는 `65` , `'B'` 는 `66` 으로 표현된다.\
가장 단순하게 `char` 형을 `int` 형으로 캐스팅하면 문자의 고유한 숫자를 확인할 수 있다.\
그리고 `AB`와 같은 연속된 문자는 각각의 문자를 더하는 방식으로 숫자로 표현하면 된다. 65 + 66 = 131이다.

```java
package collection.set;

public class StringHashMain {

    static final int CAPACITY = 10;

    public static void main(String[] args) {

        // char
        char charA = 'A';
        char charB = 'B';
        System.out.println("charA = " + (int)charA);
        System.out.println("charB = " + (int)charB);

        // hashCode
        System.out.println("hashCode('A') = " + hashCode("A"));
        System.out.println("hashCode('B') = " + hashCode("B"));
        System.out.println("hashCode('AB') = " + hashCode("AB"));

        // hashIndex
        System.out.println("hashIndex('A') = " + hashIndex(hashCode("A")));
        System.out.println("hashIndex('B') = " + hashIndex(hashCode("B")));
        System.out.println("hashIndex('AB') = " + hashIndex(hashCode("AB")));

    }

    static int hashCode(String str) {
        char[] charArray = str.toCharArray();
        int sum = 0 ;
        for (char c : charArray) {
            sum += c;
        }
        return sum;
    }

    static int hashIndex(int value) {
        return value % CAPACITY;
    }
}
```

### 해시 코드와 인덱스&#x20;

여기서는 `hashCode()` 라는 메서드를 통해서 문자를 기반으로 고유한 숫자를 만들었다. 이렇게 만들어진 숫자를 해시코드라 한다.&#x20;

여기서 만든 해시 코드는 숫자이기 때문에, 배열의 인덱스로 사용할 수 있다. 전체 과정을 그림으로 살펴보자.&#x20;

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-18 16.44.39.png" alt=""><figcaption></figcaption></figure>

* `hashCode()` 메서드를 사용해서 문자열을 해시 코드로 변경한다. 그러면 고유한 정수 숫자 값이 나오는데, \
  이것을 해시 코드라 한다.
* 숫자 값인 해시 코드를 사용해서 해시 인덱스를 생성한다.
* 이렇게 생성된 해시 인덱스를 배열의 인덱스로 사용하면 된다.

### 정리&#x20;

문자 데이터를 사용하 때도, 해시 함수를 이용해서 정수 기반의 해시 코드로 변환한 덕분에, 해시 인덱스를 사용할 수 있게\
되었다. 따라서 문자의 경우에도 해시 인덱스를 통해 빠르게 저장하고 조회할 수 있다.&#x20;

여기서 핵심은 해시코드이다!&#x20;

세상에 어떤 객체든지 정수로 만든 해시 코드로 정의할 수 있다면 해시 인덱스를 사용할 수 있다.&#x20;

## 자바의 hashCode()

해시 인덱스를 사용하는 해시 자료 구조는 데이터 추가, 검색, 삭제의 성능이 O(1)로 매우 빠르다. 따라서 많은 곳에서 자주 사용된다. 그런데 앞서 학습한 것처럼 해시 자료 구조를 사용하려면 정수로 된 숫자 값인 해시 코드가 필요하다.

자바에는 정수 `int` , `Integer` 뿐만 아니라 `char` , `String` , `Double` , `Boolean` 등 수 많은 타입이 있다. \
뿐만 아니라 개발자가 직접 정의한 `Member`, `User` 와 같은 사용자 정의 타입도 있다.

이 모든 타입을 해시 자료 구조에 저장하려면 모든 객체가 숫자 해시 코드를 제공할 수 있어야 한다.

### Object.hashCode()&#x20;

자바는 모든 객체가 자신만의 해시 코드를 표현할 수 있는 기능을 제공한다. 바로 `Object` 에 있는 `hashCode()` 메서드이다.&#x20;

* 이 메서드를 그대로 사용하기 보다는 보통 재정의(오버라이딩)해서 사용한다.&#x20;
* **이 메서드의 기본 구현은 객체의 참조(레퍼런스)값을 기반으로 해시 코드를 생성한다.**
  * **하지만 대부분의 경우 동일성(`==`) 보다는 동등성(`equals`)을 기반으로 생각하기 때문에,**\
    **객체의 참조값이 아닌, 객체의 멤버변수에 따라 해시 코드를 생성하도록 오버라이딩 해야 한다.**
* 쉽게 이야기해서 객체의 인스턴스가 다르면 해시 코드도 다르다.

### Object 의 해시 코드 비교&#x20;

**`Object`가 기본으로 제공하는 `hashCode()` 는 객체의 참조(레퍼런스)값을 해시 코드로 사용한다.** \
따라서 멤버 변수가 같더라도 각각의 인스턴스마다 서로 다른 값을 반환한다.

### 자바의 기본 클래스의 해시 코드&#x20;

* `Integer`, `String` 같은 자바의 기본 클래스들은 대부분 내부 값을 기반으로 해시 코드를 구할 수 있도록`hashCode()` 메서드를 재정의해 두었다.
* 따라서 데이터의 값이 같으면 같은 해시 코드를 반환한다.
* 해시 코드의 경우 정수를 반환하기 때문에 마이너스 값이 나올 수 있다.

## 직접 구현하는 Set2 - MyHashSetV2&#x20;

`MyHashSetV1` 은 `Integer` 숫자만 저장할 수 있었다. 여기서는 모든 타입을 저장할 수 있는 `Set` 을 만들어보자.

자바의 `hashCode()` 를 사용하면 타입과 관계없이 해시 코드를 편리하게 구할 수 있다.

### 구현 코드&#x20;

모든 객체를 받을 수 있도록 `buckets` 의 제네릭 타입을 `Object` 로 변경했다.&#x20;

```java
package collection.set;

import java.util.Arrays;
import java.util.LinkedList;

public class MyHashSetV2 {

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    LinkedList<Object>[] buckets;

    private int size = 0;
    private int capacity = DEFAULT_INITIAL_CAPACITY;

    public MyHashSetV2() {
        initBuckets();
    }

    public MyHashSetV2(int capacity) {
        this.capacity = capacity;
        initBuckets();
    }

    private void initBuckets() {
        buckets = new LinkedList[capacity];
        for (int i = 0; i < capacity; i++) {
            buckets[i] = new LinkedList<>();
        }
    }

    public boolean add(Object value) {
        int hashIndex = hashIndex(value);
        LinkedList<Object> bucket = buckets[hashIndex];
        if (bucket.contains(value)) {
            return false;
        }
        bucket.add(value);
        size++;
        return true;
    }

    public boolean contains(Object searchValue) {
        int hashIndex = hashIndex(searchValue);
        LinkedList<Object> bucket = buckets[hashIndex];
        return bucket.contains(searchValue);
    }

    public boolean remove(Object value) {
        int hashIndex = hashIndex(value);
        LinkedList<Object> bucket = buckets[hashIndex];
        boolean result = bucket.remove(value);
        if (result) {
            size--;
            return true;
        } else {
            return false;
        }
    }

    private int hashIndex(Object value) {
        return Math.abs(value.hashCode()) % capacity;   // Math.abs : 절대값
    }

    public int getSize() {
        return size;
    }

    @Override
    public String toString() {
        return "MyHashSetV2{" +
                "buckets=" + Arrays.toString(buckets) +
                ", size=" + size +
                ", capacity=" + capacity +
                '}';
    }
}
```

```java
package collection.set;

public class MyHashSetV2Main1 {

    public static void main(String[] args) {
        MyHashSetV2 set = new MyHashSetV2(10);
        set.add("A");
        set.add("B");
        set.add("AB");
        set.add("SET");
        System.out.println("set = " + set);

        // hashCode
        System.out.println("\"A\".hashCode() = " + "A".hashCode());
        System.out.println("\"B\".hashCode() = " + "B".hashCode());
        System.out.println("\"AB\".hashCode() = " + "AB".hashCode());
        System.out.println("\"SET\".hashCode() = " + "SET".hashCode());

        // 검색
        String searchValue = "SET";
        boolean result = set.contains(searchValue);
        System.out.println("set.contains(" + searchValue + ") : " + result);
    }
}
```

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-18 16.59.45.png" alt=""><figcaption></figcaption></figure>

## 직접 구현하는 Set3 - 직접 만든 객체 보관

### 구현 코드 (직접 만든 객체를 Set에 보관)

`MyHashSetV2` 는 `Object` 를 받을 수 있다. 따라서 직접 만든 `Member` 와 같은 객체도 보관할 수 있다.

여기서 주의할 점은 직접 만든 객체가 `hashCode()` , `equals()` 두 메서드를 반드시 구현해야 한다는 점이다.

```java
package collection.set.member;

import java.util.Objects;

public class Member {

    private String id;

    public Member(String id) {
        this.id = id;
    }

    public String getId() {
        return id;
    }

    @Override
    public boolean equals(Object o) {
        if (o == null || getClass() != o.getClass()) return false;
        Member member = (Member) o;
        return Objects.equals(id, member.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }

    @Override
    public String toString() {
        return "Member{" +
                "id='" + id + '\'' +
                '}';
    }
}
```

```java
package collection.set;

import collection.set.member.Member;

public class MyHashSetV2Main2 {

    public static void main(String[] args) {
        MyHashSetV2 set = new MyHashSetV2(10);
        Member hi = new Member("hi");
        Member java = new Member("Java");
        Member spring = new Member("Spring");
        Member jpa = new Member("JPA");

        System.out.println("hi.hashCode() = " + hi.hashCode());
        System.out.println("java.hashCode() = " + java.hashCode());
        System.out.println("spring.hashCode() = " + spring.hashCode());
        System.out.println("jpa.hashCode() = " + jpa.hashCode());

        set.add(hi);
        set.add(java);
        set.add(spring);
        set.add(jpa);
        System.out.println("set = " + set);

        // 검색
        Member searchValue = new Member("JPA");
        boolean result = set.contains(searchValue);
        System.out.println("set.contains(" + searchValue + ") : " + result);
    }
}
```

<figure><img src="../../../../../.gitbook/assets/스크린샷 2025-04-18 17.03.20.png" alt=""><figcaption></figcaption></figure>

### equals() 의 사용처&#x20;

`JPA`를 조회할 때 해시 인덱스는 0이다. 따라서 배열의 `0` 번 인덱스를 조회한다.\
여기에는 `[hi, JPA]` 라는 회원 두 명이 있다. 이것을 하나하나 비교해야 한다. 이때 `equals()` 를 사용해서 비교한다.

따라서 해시 자료 구조를 사용할 때는 `hashCode()` 는 물론이고, `equals()` 도 반드시 재정의해야 한다. 참고로 자바 가 제공하는 기본 클래스들은 대부분 `hashCode()` , `equals()` 를 함께 재정의해 두었다.

### 참고 - 해시 함수는 해시 코드가 최대한 충돌하지 않도록 설계&#x20;

다른 데이터를 입력해도 같은 해시 코드가 출력될 수 있다. 이것을 해시 충돌이라 한다.

해시 함수로 해시 코드를 만들 때 단순히 문자의 숫자를 더하기만 해서는 해시가 충돌할 가능성이 높다. \
해시가 충돌하면 결과적으로 같은 해시 인덱스에 보관된다. 따라서 성능이 나빠진다.

**자바의 해시 함수는 이런 문제를 해결하기 위해 문자의 숫자를 단순히 더하기만 하는 것이 아니라 내부에서 복잡한 추가 연산을 수행한다.**&#x20;

* 따라서 자바 해시 코드를 사용하면 `"AB"` 의 결과가 `2081` , `"SET"` 의 결과가 `81986` 으로 복잡한 계산 결과가 나온다.

**복잡한 추가 연산으로 다양한 범위의 해시 코드가 만들어지므로 해시가 충돌할 가능성이 낮아지고,** \
**결과적으로 해시 자료구조를 사용할 때 성능이 개선된다.**

해시 함수는 같은 입력에 대해서 항상 동일한 해시 코드를 반환해야 한다.

좋은 해시 함수는 해시 코드가 한 곳에 뭉치지 않고 균일하게 분포하는 것이 좋다.&#x20;

* 그래야 해시 인덱스도 골고루 분포되어서 해시 자료 구조의 성능을 최적화할 수 있다.&#x20;

이런 해시 함수를 직접 구현하는 것은 쉽지 않다. 자바가 제공하는 해시 함수들을 사용하면 이런 부분을 걱정하지 않고 최적화 된 해시 코드를 구할 수 있다.

하지만 자바가 제공하는 해시 함수를 사용해도 같은 해시 코드가 생성되어서 해시 코드가 충돌하는 경우도 간혹 존재한다.예)

* `"Aa".hashCode() = 2112`
* `"BB".hashCode() = 2112`&#x20;

이 경우 같은 해시 코드를 가지기 때문에 해시 인덱스도 같게 된다.

**하지만, `equals()` 를 사용해서 다시 비교하기 때문에 해시 코드가 충돌하더라도 문제가 되지는 않는다.** \
**그리고 매우 낮은 확률로 충돌하기 때문에 성능에 대한 부분도 크게 걱정하지 않아도 된다.**

## 직접 구현하는 Set4 - 제네릭과 인터페이스 도입&#x20;

```java
package collection.set;

public interface MySet<E> {
    boolean add(E element);

    boolean remove(E value);

    boolean contains(E value);
}
```

```java
package collection.set;

import java.util.Arrays;
import java.util.LinkedList;

public class MyHashSetV3<E> implements MySet<E>{

    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    LinkedList<E>[] buckets;

    private int size = 0;
    private int capacity = DEFAULT_INITIAL_CAPACITY;

    public MyHashSetV3() {
        initBuckets();
    }

    public MyHashSetV3(int capacity) {
        this.capacity = capacity;
        initBuckets();
    }

    private void initBuckets() {
        buckets = new LinkedList[capacity];
        for (int i = 0; i < capacity; i++) {
            buckets[i] = new LinkedList<>();
        }
    }

    public boolean add(E value) {
        int hashIndex = hashIndex(value);
        LinkedList<E> bucket = buckets[hashIndex];
        if (bucket.contains(value)) {
            return false;
        }
        bucket.add(value);
        size++;
        return true;
    }

    public boolean contains(E searchValue) {
        int hashIndex = hashIndex(searchValue);
        LinkedList<E> bucket = buckets[hashIndex];
        return bucket.contains(searchValue);
    }

    public boolean remove(E value) {
        int hashIndex = hashIndex(value);
        LinkedList<E> bucket = buckets[hashIndex];
        boolean result = bucket.remove(value);
        if (result) {
            size--;
            return true;
        } else {
            return false;
        }
    }

    private int hashIndex(E value) {
        return Math.abs(value.hashCode()) % capacity;   // Math.abs : 절대값
    }

    public int getSize() {
        return size;
    }

    @Override
    public String toString() {
        return "MyHashSetV3{" +
                "buckets=" + Arrays.toString(buckets) +
                ", size=" + size +
                ", capacity=" + capacity +
                '}';
    }
}
```

```java
package collection.set;

import collection.set.member.Member;

public class MyHashSetV3Main {

    public static void main(String[] args) {
        MySet<String> set = new MyHashSetV3<>(10);
        String hello = "Hello";
        String java = "Java";
        String spring = "Spring";
        String jpa = "JPA";

        System.out.println("hello.hashCode() = " + hello.hashCode());
        System.out.println("java.hashCode() = " + java.hashCode());
        System.out.println("spring.hashCode() = " + spring.hashCode());
        System.out.println("jpa.hashCode() = " + jpa.hashCode());

        set.add(hello);
        set.add(java);
        set.add(spring);
        set.add(jpa);
        System.out.println("set = " + set);

        // 검색
        String searchValue = "JPA";
        boolean result = set.contains(searchValue);
        System.out.println("set.contains(" + searchValue + ") : " + result);
    }
}
```
