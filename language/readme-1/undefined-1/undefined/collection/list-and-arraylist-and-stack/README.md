# List

> 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/8/docs/api/java/util/List.html](https://docs.oracle.com/javase/8/docs/api/java/util/List.html)

## List&#x20;

* `Collection` 인터페이스를 상속한 `List` 인터페이스는 `Collection` 인터페이스의 메서드와 별 차이는 없지만, **배열처럼 순서가 있다는 점이 매우 중요한 특징이다.**
  * **여기서 순서란 정렬을 의미하는 것이 아닌, 사용자가 저장하는 순서대로 저장된다 생각하면 된다.**
* **`Set` 과 달리 중복을 허용한다.**&#x20;
* `List` 인터페이스를 상속한 `Vector`, `ArrayList` 클래스의 기능과 사용법은 거의 비슷하지만, **`Vector` 는 Thread Safe 하고 `ArrayList` 는 Thread Safe 하지 못하다...**
* `Stack` 클래스는 `Vector` 클래스를 확장해 만들었다. `Stack` 클래스의 가장 큰 특징은, LIFO(Last In First Out) 를 지원하기 위함이다.
  * 스택 메모리에 스택프레임이 들어왔다 나가는 과정을 생각해보자!
  * 이와 같은 데이터 구조를 다뤄야 할 때 Stack 클래스를 사용 할 수 있지만, `Stack` 보다는 `ArrayDeque` 를 사용한자. \
    **-> `Stack` 에서 상속 받는 `Vector` 클래스는 deprecated 되었다.**
