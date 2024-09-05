# 클래스 로더 시스템

> #### 참고 링크&#x20;
>
> [https://docs.oracle.com/javase/specs/jvms/se20/html/jvms-5.html](https://docs.oracle.com/javase/specs/jvms/se20/html/jvms-5.html)

<figure><img src="../../../.gitbook/assets/image (56).png" alt=""><figcaption></figcaption></figure>

* **클래스 로더 시스템은 프로그램이  실행될 때, 클래스 수준의 정보를 메서드 영역에 올리기 위한 과정이다.**&#x20;
* 클래스 로더 시스템은 로딩 -> 링크 -> 초기화 순으로 진행된다.

## 1. 로딩

* **클래스 로더가 바이트코드(.class)를 읽고 그 내용을 "메소드" 영역에 저장.**
* 이때 메소드 영역에 저장하는 데이터
  * FQCN(Full Qualify Class Name) -> Full 패키지 이름 + 클래스 이름
  * Class / Interface / Enum
  * 메소드와 **변수(초기화 되지 않았다)**&#x20;
* 로딩의 목적은 바이트 코드를 읽고서 그 내용을 메모리로(메소드 영역) 로드하여, **JVM 에서 사용 가능하도록 변환하는 과정.**
* **우리가 작성하는 클래스 대부분은 Application ClassLoader 가 읽게 된다.**\
  \-> Bootstrap, Extension, Application ClassLoader 가 순서대로 읽게 되고 클래스를 읽을 수 없을 시 _**ClassNotFoundException**_ 오류를 만나게 된다.

## 2. 링크

* **검증 (Verification)**
  * 로드된 바이트코드가 JVM 명세에 맞는지 확인합니다.
* **준비 (Preparation)**
  * 클래스 변수(static fields)를 위한 메모리를 할당하고 기본값으로 초기화합니다.
* **해결 (Resolution)**
  * **심볼릭 참조가 가리키는 클래스, 메서드, 필드의 정확한 위치를 찾기 위해 클래스 로더를 사용한다.**&#x20;
  * **참조가 유효한지 확인한 후, 논리적 레퍼런스를 실제 레퍼런스로 변환합니다. 이 단계는 런타임에 동적으로 이루어질 수도 있습니다.(Optional)** \
    \-> 이를 통해서 JVM 은 실행 중 실제 메서드나 필드에 접근할 수 있다.&#x20;

## 3. 초기화

* **클래스 변수(`static`) 값을 할당한다.(`static` 블럭이 있다면 이때 실행된다)**
