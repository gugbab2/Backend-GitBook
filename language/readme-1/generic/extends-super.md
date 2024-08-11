# 와일드카드 extends / super 사용시기

## PECS 공식&#x20;

* PESC 란, Producer-Extends / Consumer-Super 라는 단어의 약자인데 다음을 의미한다.&#x20;
  * **외부에서 온 데이터를 생산(Producer) 한다면 \<? extends T> 를 사용 (하위타입으로 제한)**&#x20;
  * **외부에서 온 데이터를 소비(Consumer) 한다면 \<? super T> 를 사용(상위타입으로 제한)**&#x20;

```java
class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;

    // 외부로부터 리스트를 받아와 매개변수의 모든 요소를 내부 배열에 추가하여 인스턴스화 하는 생성자
    public MyArrayList(Collection<? extends T> in) {
        for (T elem : in) {
            element[index++] = elem;
        }
    }

    // 외부로부터 리스트를 받아와 내부 배열의 요소를 모두 매개변수에 추가해주는 메서드
    public void clone(Collection<? super T> out) {
        for (Object elem : element) {
            out.add((T) elem);
        }
    }
}
```

### Producers Extend

* 위 예제에서 extends 가 쓰이는 곳은 생성자 메서드의 매개변수 부분이다.&#x20;
* 즉, 외부에서 온 데이터를 매개변수에 담아 for 문으로 순회하여 MyArrayList 를 인스턴스화(생성) 하는 생산자(Producer) 역할을 하고 있다고 말할 수 있다.&#x20;

```java
class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;

    // 외부로부터 리스트를 받아와 매개변수의 모든 요소를 내부 배열에 추가하여 인스턴스화 하는 생성자
    public MyArrayList(Collection<? extends T> in) {
        for (T elem : in) { 
            element[index++] = elem;
        }
    }

    ...
}
```

### Consumer Super&#x20;

* 외부에서 리스트를 받아 요소를 복사하여 적재하는 clone 메서드의 매개변수에는 super 와일드카드 키워드가 쓰였다.&#x20;
* 즉, MyArrayList 의 내부 배열을 소비하여 매개변수 리스트에 적재하는 행위를 하고 있다고 볼 수 있는 것이다.&#x20;

```java
class MyArrayList<T> {
    Object[] element = new Object[5];
    int index = 0;
    
    ...
    
    // 외부로부터 리스트를 받아와 내부 배열의 요소를 모두 매개변수에 추가해주는 메서드
    public void clone(Collection<? super T> out) {
        for (Object elem : element) {
            out.add((T) elem);
        }
    }
}
```

