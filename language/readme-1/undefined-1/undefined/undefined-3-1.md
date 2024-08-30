# 예외

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-08-29 12.02.01.png" alt="" width="563"><figcaption></figcaption></figure>



## 예외의 종류는 세 가지다.&#x20;

* 예외의 종류는 3가지가 존재하는데, 그 종류는 다음과 같다. \
  \-> error, runtime exception 을 제외한 모두는 checked exception 에 해당한다.&#x20;
  * **checked exception**&#x20;
  * **error**
  * **runtime exception or unchecked exception**&#x20;

### error&#x20;

* **에러는 자바 프로그램 밖에서 발생한 예외를 말한다.** \
  \-> 가장 흔한 예가, 서버의 디스크가 고장 났다든지, 메인 보드가 고장나서 프로그램이 잘 동작하지 못하는 경우가 해당된다.&#x20;
* 뭔자 자바 프로그램에 오류가 발생했을 때, 오류의 이름이 Error 로 끝나면 에러이고, Exception 으로 끝나면 예외이다.&#x20;
* **Error, Exception 으로 끝나는 오류의 가장 큰 차이는 프로그램 안에서 발생했는지, 밖에서 발생했는지 여부이다.**&#x20;
  * 하지만 더 큰 차이는 프로그램이 멈추어 버리냐, 계속 실행할 수 있느냐의 차이다.&#x20;
  * 더 정확하게 말하면 **Error 는 프로세스에 영향을 주고, Exception 은 쓰레드에 영향을 준다.**&#x20;

### runtime exception (unchecked exception)&#x20;

* 런타임 예외는 예외가 발생할 것을 미리 감지하지 못했을 때 발생한다.&#x20;
* 이러한 예외들은 컴파일 시 체크를 하지 않기 때문에 unchecked exception 이라고 부르는 것이다.&#x20;
  * **`Exception` 을 바로 확장한 클래스들이 checked exception 이라고 부르고**
  * **`RuntimeException` 을 확장한 클래스들이 unchecked exception 이다.**&#x20;

### checked exception&#x20;

* error, runtime exception 을 제외한 모든 예외는 checked exception 이다.&#x20;
* 컴파일 타임에 체크해주는 예외이다.&#x20;

## 자바는 왜? Checked Exception 과 Runtime Exception 으로 나뉘게 되었을까?

* 자바는 안전성과 명확성을 중요시하는 언어이다.&#x20;
* **프로그램 실행 중 발생할 수 있는 오류 상황을 개발자가 명확히 인지하고 처리하도록 하며, 런타임에 발생할 수 있는 예기치 않은 오류를 최소화하는 것이 중요한 목표였다.**&#x20;
* Checked Exception : **주로 외부 리소스와 상호작용에서 발생할 가능성이 높은 예외들로, 개발자가 미리 예측하고 처리해야 하는 상황들이다.** 때문에 해당 예외에 대해서는 예외 처리를 강제함으로써 프로그램이 더 안정적으로 돌아가게 하는 것이 목표이다. **(핸들러 처리를 강제)**
* UnChecked Exception : **주로 프로그래머의 실수나 논리적 오류에서 발생하는 예외들로, 이러한 예외들은 범용적인 성격이 있는 예외들이다.** 만약 이 예외들을 모두 예외 처리 한다면 코드가 지나치게 복잡해지고 비효율적이다. **(런타임에 에러 확인)**&#x20;

## 모든 예외의 할아버지! java.lang.Throwable

* `Error`, `Exception` 클래스 모두 `Throwable` 클래스를 상속받아 처리하도록 되어 있다.&#x20;
* 상속 관계가 이렇게 되어있는 이유는 `Error`, `Exception` 의 성격은 다르지만 모두 동일한 이름의 메소드를 사용하여 처리할 수 있도록 하기 위함이다.&#x20;
* 가장 유용하세 사용할 수 있는 메서드 3가지&#x20;
  * `getMessage()` : 가장 간단한 메시지&#x20;
  * `toString()` : 약간 더 자세한 메시지&#x20;
  * `printStackTrace()` : 메시지와 함께 스택 트레이스 출력\
    \-> 로그의 양이 많아질 수 있기 때문에, 개발시에만 사용하자.
* 더 자세한 내용은 링크\
  \-> [https://docs.oracle.com/javame/8.0/api/cldc/api/index.html?java/lang/Throwable.html](https://docs.oracle.com/javame/8.0/api/cldc/api/index.html?java/lang/Throwable.html)

## 직접 만드는 예외&#x20;

```java
public class MyException extends Exception {
    public MyException () {
        super(); 
    }
    public MyException(String message) {
        super(message); 
    }
}
```

* `Throwable` 을 직접 상속 받는 클래스는 `Exception`, `Error` 가 있다.&#x20;
  * `Error` 와 관련된 클래스는 개발자가 손댈 필요도 없고 손대어서도 안된다.&#x20;
  * **하지만, `Exception` 을 처리하는 예외 클래스는 개발자가 임의로 추가해서 만들 수 있다.** \
    **-> 단 한가지 조건이 있는데, `Throwable` 이나 그 자식 클래스를 상속받아야 한다는 것이다.**&#x20;
* 예외 클래스가 되기 위한 조건은 위 코드와 같이 상당히 간단하다. 예외 관련 클래스를 확장하면 된다.&#x20;

## 자바 예외 처리의 진화&#x20;

* 초기 자바에서 Checked Exception 은 프로그램의 안정성을 높이기 위해 도입되었다. 이 예외 처리 방식의 주요 목적은 개발자가 예외 상황을 명시적으로 처리하도록 강제하는 것이었다. (Exception Handler 유무 체크)&#x20;
* 하지만 Checked Exception 의 예외 처리가 강제되면서 코드의 복잡성이 증가하고 예외 처리 로직이 과도하게 추가되는 문제가 발생했다. \
  \-> 실제로, 모든 예외 상황을 예측하고 처리하는 것은 실제로 매우 어렵다..
* 이러한 단점을 해결하기 위해서 Unchecked Exception 이 사용되기 시작하였고, 이는 예외 처리 코드를 최소화하면서 꼭 필요한 경우에만 예외 처리를 수행할 수 있게 되었다. 이로 인해 예외 처리 로직을 간결하게 유지해 줄 수 있게 되었다.&#x20;
* 이러한 장점들로 인해 현재 자바 커뮤니티에서는 Unchecked Exception 을 선호하는 경향이 강해지고 있다. \
  \-> 현대 소프트웨어 개발에서는 유연성과 유지보수성이 매우 중요한 요소이다.&#x20;

### 예외 처리 전략의 권장 사항&#x20;

* 먼저 Checked Exception 과 UnChecked Exception 각각의 장단점을 이해하고 상황에 맞게 적절한 예외 처리 방식을 선택하는 것이 중요하다.&#x20;
* 일반적으로 복구 가능한 예외 상황에서는 Checked Exception 을 사용하고, 프로그램의 실행 계속할 수 없는 치명적인 예외 상황에서는 Unchecked Exception 을 사용하는 것이 권장된다.&#x20;
* 또한 예외 처리 로직을 작성할 때는 예외의 원인을 명확히 하고, 가능한 한 구체적인 예외 타입을 사용 하는 것이 좋다. \
  \-> 이를 통해 예외 상황을 더 정확하게 파악하고, 적절한 처리를 수행할 수 있다.&#x20;
