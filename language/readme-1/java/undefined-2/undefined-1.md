# 캡슐화

## 캡슐화&#x20;

* 캡슐화(Encapsulation) 는 객체 지향 프로그래밍의 중요한 개념 중 하나이다.&#x20;
* 캡슐화는 데이터와 해당 데이터를 처리하는 메서드를 하나로 묶어서 외부에서의 접근을 제한하는 것을 말한다.
* 캡슐화를 통해서 데이터의 직접적인 변경을 방지하거나 제한할 수 있다.&#x20;
* **캡슐화는 쉽게 이야기해서 속성과 기능을 하나로 묶고, 외부에 꼭 필요한 기능만 노출하고 나머지(변수, 내부에서 사용하는 메서드)는 모두 내부로 숨기는 것이다.**&#x20;

## 그럼 어떤 것을 숨기고 어떤 것을 노출해야 할까?

### 1. 데이터는 숨겨라

* 객체에는 속성(데이터) 과 기능(메서드) 이 있다. 가장 필수로 숨겨야 하는 것은 속성(데이터) 이다.&#x20;
* 객체 내부의 데이터를 외부에서 함부로 접근하게 두면, 클래스 안에서 데이터를 다루는 모든 로직을 무시하고 데이터를 변경할 수 있게 되고, 따라서 캡슐화가 깨진다.
* **객체의 데이터는 객체가 제공하는 기능인 메서드를 통해서 접근해야 한다.**

#### 일상생활의 예를 생각해보자

* 우리가 자동차를 운전할 때 자동차 부품을 다 열어서 그 안에 있는 속도계를 직접 조절하지 않는다. 단지 자동차가 제공하는 엑셀 기능을 사용해서, 엑셀을 밟으면 자동차가 나머지는 다 알아서 하는 것이다.&#x20;

### 2. 내부에서만 사용하는 기능을 숨겨라

* 객체의 기능 중에서 외부에서 사용하지 않고 내부에서만 사용하는 기능들이 있다. 이런 기능도 모두 감추는 것이 좋다. \
  \-> ex, 속도계가 올라가고 내려가는 기능 \
  \-> 해당 기능은 사용자가 직접 조절하는 것이 아닌, 엑셀을 밟으면 자동으로 실행되는 기능이다.&#x20;
* 내부에서만 사용하는 기능들을 외부에서 알게 되면, 사용자는 객체에 대해 너무 많은 것을 알아야 한다.
* 사용자 입장에서 꼭 필요한 기능만 외부에 노출하고, 나머지 기능은 내부로 숨기자..&#x20;
