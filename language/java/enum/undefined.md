---
description: 스
---

# 추상클래스, 인터페이스

## 1. 추상클래스

### 1-1. 추상클래스란?

* 객체지향 프로그래밍에서 클래스가 설계도라면, 추상클래스는 미완성 설계도로 비유할 수 있다.
* 추상클래스는 미완성 클래스로 인스턴스를 생성할 수 없고, 상속을 통해서 자손 클래스에서 완성된다.
* 클래스 선언부에 abstract 키워드가 있다면 "추상메서드가 포함되어 있는 추상클래스다." 라고 생각할 수 있다.
* **이를 통해 클래스간 공통점을 추상클래스에서 구현하고, 차이점은 이를 상속 받은 클래스에서 추상메서드를 구현함으로 확장성 있는 코드를 작성할 수 있다.**\
  **-> 이를 통해서 자손 클래스에서 추상 메서드의 구현을 강요할 수 있다.**\
  **-> default 키워드를 통해서 강요하지 않을 수도 있다.**

### **1-2. 추상메서드 작성**

* 선언부만 작성하고 구현부는 작성하지 않은 채로 남겨놓은 메서드이다.
* 추상메서드 역시 선언부 앞에 abstract 키워드를 붙여야 한다.

```java
abstract class Player{
	abstract void play(int pos);
	abstract void stop();
}

class AudioPlayer extends Player {
	void play(int pos){...}
	void stop(){...}
}

```

> * 메서드를 작성할 때 실제 구현부보다 중요한 부분은 선언부이다.\
>   \-> 선언부만 작성해도 메서드의 절반 이상은 완성된 것!

## 2. 인터페이스

### 2-1. 인터페이스란?

* 인터페이스는 일종의 추상클래스로, 추상클래스보다 추상화 정도가 높아서 몸통을 갖춘 일반 메서드 또는 인스턴스변수(상수)를 구성원으로 가질 수 있다.
* 오직 추상메서드와 상수만을 멤버로 가질 수 있다.&#x20;
* 코드를 구현하기 이전에 인터페이스를 구현하는 것은 매우 중요하다. \
  \-> 행위를 규정하는 것은 매우 중요하다.&#x20;

### 2-2. 인터페이스 작성

```java
interface PlayingCard{
	public static final int SPACE = 4;
	public abstract String get CardNumber();
}
```

> * 모든 인스턴스변수는 public static final 이어야 하며, 이를 생략할 수도 있다.
> * 모든 메서드는 public abstract 이어야 하며, 이를 생략할 수도 있다.

### 2-3. 인터페이스를 이용한 다중상속

* 다중상속은 장점도 있지만, 단점이 더 크다고 판단하였기에 자바에서는 다중상속을 지원하지 않는다.\
  \-> 만약 두 조상으로부터 상속 받는 멤버 중 멤버변수의 이름이 같거나, 메서드의 선언부가 일치하고 구현 내용이 다르다면 이 두 조상으로부터 상속받는 자손 클래스는 어느 조상의 것을 상속받게 되는지 알 수 없다.\
  (고민해야 될 상황이 많아진다...)
* 하지만 다른 객체지향 언어인 C++ 에서 다중상속을 허용하기 때문에, 자바에서 다중상속을 지원하지 않는 것은 단점으로 보여진다. 이런 단점의 대응책으로 인터페이스에서 다중상속을 지원하는 것일 뿐..
* 자바에서 보통 인터페이스로 다중 상속을 구현하는 경우는 거의 없다.
* 만약! 두 개의 클래스 기능이 필요할 때는 아래의 프로세스를 따른다.
  * 하나의 클래스를 상속(상속)
  * 나머지 클래스는 클래스 내부에서 인스턴스를 만들어서 사용한다(합성)

### 2-4. 인터페이스의 이해

* 인터페이스를 이해하기 위해서는 다음 두 가지 사항을 반드시 염두해야 한다.
  * 클래스를 사용하는 쪽과, 클래스를 제공하는 쪽이 있다.
  * 메서드를 사용하는 쪽에서는 사용하려는 메서드의 선언부만 알면 된다.\
    \-> 구현부는 알 필요가 없다.\
    \-> 이를 통해 내부 구현을 외부에 감추고 응집도를 높일 수 있다.(객체지향적인 설계)

### 2-5. 인터페이스의 default method(JDK8 에 추가된 메서드)

* 기존 인터페이스는 서브 클래스의 오버라이딩을 강조하기 때문에, 예를들어 인터페이스의 추상메서드를 추가하게 된다면 인터페이스를 구현한 모든 서브클래스의 컴파일 오류가 발생한다.&#x20;
* **해당 문제를 해결하기 위해서 default 메서드는 인터페이스 내에서 구현을 하고 서브클래스의 구현을 강조하지 않는다.**&#x20;
  * **때문에, 서브클래스는 원할 때 오버라이딩 할 수 있다.**

