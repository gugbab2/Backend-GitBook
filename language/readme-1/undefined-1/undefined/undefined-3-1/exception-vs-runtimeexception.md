# Exception vs RuntimeException

<figure><img src="../../../../../.gitbook/assets/스크린샷 2023-06-25 13.46.38.png" alt="" width="563"><figcaption></figcaption></figure>

## 1. Error

* 수습할 수 없는 심각한 오류

## 2. Exception(Checked Exception)

* 예외처리를 통해 수습할 수 있다.\
  \-> 메모리 부족, StackOverflow 가 발생하는 것은 개발자가 제어할 수 없다.
* Exception 클래스는 모든 예외 클래스의 최상위 클래스이며, 예외가 발생할 수 있는 모든 상황을 다룬다.
* Exception 클래스는 다양한 하위 클래스를 가지며, 각 하위 클래스는 특정한 유형의 예외를 나타낸다.
* Exception 의 예
  * IOException, ClassNotFoundException, SQLException ...
* **외부 리소스나, 네트워크를 사용하는 상황에서 컴파일러의 도움을 받아 적절한 예외를 처리할 수 있도록 도와준다.**

## 3. RuntimeException(Unchecked Exception)

* RuntimeException 은 Exception 클래스의 하위 클래스 중 하나이다.\
  \-> RumtimeException 과 그 하위 클래스들은 'unchecked' 예외로 분류된다.
* 이러한 예외들은 개발자가 예외 처리를 강제 받지 않으며, 명시적인 예외 처리 코드를 작성하지 않아도 된다.\
  \-> 하지만, 개발자의 주의와 적절한 프로그래밍 관행을 통해서 이러한 예외를 방지하는 것이 권장된다.
* RuntimeException 의 예
  * NullPointerException, ArrayIndexOutboundsException, ...\
    \-> 해당 RuntimeException 들은 개발자의 실수를 통해서 발생되며, 개발자는 이러한 예외를 방지하기 위한 예외처리를 해 두어야 한다.
* **기본적으로 개발자의 실수로 인해 발생하는 오류로 코드 내에서 적절한 오류코드를 처리할 수 있어야 한다.**
