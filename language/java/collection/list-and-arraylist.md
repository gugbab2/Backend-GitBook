# List & ArrayList

## List 기본

* Collection 인터페이스를 상속한 List 인터페이스는 Collection 인터페이스의 메서드와 별 차이는 없지만, **배열처럼 순서가 있다는 점이 매우 중요한 특징이다.**
* **List 인터페이스를 상속한 Vector, ArrayList 클래스**의 기능과 사용법은 거의 비슷하지만, Vector 는 Thread Safe 하고 ArrayList 는 Thread Safe 하지 못하다...
* **Stack 클래스는 Vector 클래스를 확장해 만들었다.**  Stack 클래스의 가장 큰 특징은, LIFO(Last In First Out) 를 지원하기 위함이다. \
  \-> 스택 메모리에 스택프레임이 들어왔다 나가는 과정을 생각해보자!\
  \-> 이와 같은 데이터 구조를 다뤄야 할 때 Stack 클래스를 사용하자!!
* LinkedList 클래스는 'List' 에도 속하지만, 'Queue' 에도 속한다.
