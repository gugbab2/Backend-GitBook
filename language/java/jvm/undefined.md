# 클래스 로더

<figure><img src="../../../.gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

* 클래스 로더는 로딩 -> 링크 -> 초기화 순으로 진행된다.

## 1. 로딩

* **클래스 로더가 .class 파일을 읽고 그 내용에 적절한 바이너리 데이터를 **_**"메소드" 영역**_**에 저장.**\
  \-> 메소드 영역에 저장되는 내용은 **클래스 수준의 정보이다.**
* 이때 메소드 영역에 저장하는 데이터
  * FQCN(Full Qualify Class Name) -> Full 패키지 이름 + 클래스 이름
  * Class / Interface / Enum
  * 메소드와 변수(인스턴스 변수?)
* 로딩이 끝나면, 해당 클래스 타입의 객체를 생성하여 "힙" 영역에 저장한다.
* **우리가 작성하는 클래스 대부분은 Application ClassLoader 가 읽게 된다.**\
  \-> Bootstrap, Extension, Application ClassLoader 가 순서대로 읽게 되고 클래스를 읽을 수 없을 시 _**ClassNotFoundException**_ 오류를 만나게 된다.

## 2. 링크

* Verify : .class 파일의 형식이 유효한지 체크한다.\
  \-> .class 파일(바이너리 코드) 를 임의로 변경하게 되면 오류가 발생한다.
* Prepare : 클래스 변수(static 변수) 와 기본값에 필요한 메모리를 할당한다.
* Resolve : 논리적 메모리 레퍼런스를 메소드 영역에 있는 실제 레포런스로 교환한다.\
  \-> Optional 로 이때 연결될 수도, 아닐수도 있다.

## 3. 초기화

* Static 변수의 값을 할당한다.(static 블럭이 있다면 이때 실행된다)
