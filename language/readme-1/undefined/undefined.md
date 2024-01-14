# 클래스 변수 / 인스턴스 변수 / 지역 변수

* 클래스 변수 (Static Variables)
  * 클래스 수준에서 정의되며 'static' 키워드로 표시한다.
  * 모든 클래스 인스턴스 사이에 공유된다. \
    \-> 따라서 한 클래스 변수의 변경 내용은 모든 인스턴스에 영향을 준다.&#x20;
  * **프로그램 실행 동안 메모리에 유지되며, 프로그램이 종료될 때까지 유지된다.**&#x20;
  * 클래스 이름을 통해서 접근해야 하며, 인스턴스 생성과 상관없이 사용할 수 있다.&#x20;
  * 주로 공통적인 상태나 설정 값을 저장하는 데 사용된다.&#x20;
* 인스턴스 변수 (Instance Variables)
  * 클래스 내부에 선언되며 객체 인스턴스 생성 시에 메모리에 할당 된다.&#x20;
  * 객체의 개별적인 상태를 나타내며, 각 인스턴스마다 별개의 값을 가진다.&#x20;
  * **인스턴스가 생성된 동안 메모리에 유지되며, 해당 인스턴스의 생명 주기가 같다.**
  * 'this' 키워드를 통해 접근하며, 해당 인스턴스의 메서드나 블록 내에서 사용된다.&#x20;
  * 객체의 속성이나 상태를 저장하는 데 사용된다.&#x20;
* 지역변수&#x20;
  * 메서드, 생성자 또는 블록 내에 선언되며, 해당 블록이 실행될 때 메모리에 할당된다.
  * 블록 내에서만 유효하며 블록이 실행을 마치면 메모리에서 해제된다.&#x20;
  * 블록 내에서 메서드나 생성자의 매개변수도 지역 변수의 한 종류이다.&#x20;
  * 블록 내에서 선언된 변수는 해당 블록 내에서만 접근할 수 있다.&#x20;