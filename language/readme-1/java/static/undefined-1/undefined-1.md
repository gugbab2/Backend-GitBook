# 메소드가 실행될 때 어떤일이 일어나는가?

* 우리가 .java 파일을 컴파일 하고 .class 파일을 생성하는 시작점은 어디일까?\
  \-> 틀린 답 : 현재폴더?\
  \-> 맞는 답 : ClassPath
* ClassPath
  * 말 그대로 클래스를 찾기위한 경로, JVM 이 클래스파일을 찾는데 기준이 되는 경로를 의미한다.
  * ClassPath 를 지정하는 방법
    * 환경변수 CLASSPATH 를 사용 (아마 InteliJ 와 같은 IDE 를 사용하게 되면 자동으로 지정)
    * Java Runtime 에 -classpath 플래그를 사용하는 방법

## 메소드가 실행될 때 일어나는 일의 순서

1. .java 파일을 컴파일 한 결과인, .class(바이트 코드) 파일이 생성된다.
2. JVM 은 ClassPath 에 저장된 .class 파일을 읽어들이고, 메서드 영역에 클래스 수준의 정보를 저장한다.\
   \-> 이 과정을 통해 JVM 은 해당 static 정보와, 클래스 정보를 파악하게 된다.
3. 프로그램의 시작점인 main 메서드를 찾고 실행하게 된다.
4. Stack 영역에 main 에서 실행된 메서드 정보를 올리게 된다.\
   \-> Stack 영역에 저장된 실행 정보 하나를 StackFrame 이라고 표현한다.\
   \-> StackFrame 안에 해당 메서드의 지역변수 정보가 저장된다.\
   \-> StackFrame 은 메서드 단위로 생성된다.
5. Heap 영역에 Stack 영역에서 참조하는 인스턴스의 정보가 저장된다.
6. 메서드가 실행이 종료됨에 따라, StackFrame 은 Stack 영역에서 사라진다.
7. Heap 영역의 할당된 자원도 자동적으로 반환된다.(GC)

> **스택프레임 ( Stack Frame)**
>
> 하나의 메서드에 필요한 메모리 덩어리를 묶어서 스택 프레임(Stack Frame) 이라고 한다.\
> 하나의 메서드당 하나의 스택 프레임이 필요하며, 메서드 호출하기 직전 스택프레임을 자바 Stack 에 생성한 후 메서드를 호출하게 된다.
>
> 스택 프레임이 쌓이는 데이터는 지역변수, 리턴값 등이 있다.\
> 만약 메서드 호출 범위가 종료되면 스택에서 제거된다.

<figure><img src="../../../../../.gitbook/assets/image (67).png" alt=""><figcaption><p>스택프레</p></figcaption></figure>

## 자바의 메모리

<figure><img src="../../../../../.gitbook/assets/스크린샷 2023-09-11 19.07.26.png" alt=""><figcaption></figcaption></figure>

* 위에서 클래스 수준의 정보가 저장되는 메모리 영역을 메서드 영역이라 불렀는데, 버전에 따라 저장되는 영역이 달라졌다.
  * **자바 7 : PermGen(Java Heap)**
  * **자바 8 : Metaspace(Native Memory)**\
    **-> Native Memory : 운영체제 수준에서 관리하는 메모리**\
    **-> PermGen 은 고정 크기 문제가 있는데, Metaspace 는 동적 메모리 할당으로 이 문제를 해결했다.**
