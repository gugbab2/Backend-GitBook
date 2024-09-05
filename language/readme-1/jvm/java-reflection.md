---
hidden: true
---

# Java Reflection

## 1. Java Reflection?

* Java 프로그래밍을 할 때, 보통은 클래스를 직접 선언하고 만들어 사용해왔다.
* **하지만, 어떤 경우에는 애플리케이션 실행 중 클래스를 동적으로 불러와야 할 경우가 생긴다.**\
  **-> 컴파일 단에서 개발자가 직접 클래스를 사용하는 것이 아닌, 런타임에 클래스를 사용하는 것이다.**
* 이 방법을 Reflection 이라 부르며, **Class 클래스**가 해당 기능을 수행한다.

## 2. Reflection 은 어떠한 원리로 이루어지는가?

* 자바의 모든 클래스와 인터페이스는 컴파일 후 .java -> .class 파일로 변환된다.
* 해당 .class 파일에는 멤버변수, 메서드, 생성자 등 객체의 정보들이 들어 있는데,
* Reflection 은 런타임에 JVM 내 클래스로더에 의해서 클래스 파일이 메모리에 올라갈 때, Class 클래스는 이 .class 파일의 정보를 바탕으로 힙 영역에 자동적으로 인스턴스화 시킨다.
* 때문에, 따로 new 인스턴스화 하지 않고 바로 사용하면 된다.

> JVM 클래스 로더는 런타임에 필요한 클래스를 메모리(힙)에 로드하는 역할을 한다.
>
> 로드할 때 기존에 생성된 객체가 있다면, 해당 객체의 참조를 반환하고, 없으면 CLASSPATH 에 지정된 경로를 따라 .class 파일을 읽어 인스턴스화 시킨다
>
> 만약 .class 파일을 찾지 못하면 ClassNotFoundException 예외를 띄우게 된다.

<figure><img src="../../../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

## 3. Class 클래스 객체 얻는 3가지 방법

### 3-1. Object.getClass() 로 얻기

* 모든 클래스의 최상위 클래스인 Object 클래스에서 제공하는, getClass() 메서드를 통해서 가져온다.
* 단, 해당 클래스가 인스턴스화 된 상태 이어야 한다는 제약사항이 있다.

```java
public static void main(String[] args) {

    // 스트링 클래스 인스턴스화
    String str = new String("Class클래스 테스트");

    // getClass() 메서드로 얻기
    Class<? extends String> cls = str.getClass();
    System.out.println(cls); // class java.lang.String
}
```

### 3-2. .class 리터럴로 얻기

* 인스턴스가 존재하지 않고, 컴파일된 .class 파일만 있다면, 리터럴(상수)로 Class 객체를 곧바로 얻을 수 있다.
* 가장 심플하게 Class 객체를 가져오는 방법이다.

```java
public static void main(String[] args) {

    // 클래스 리터럴(*.class)로 얻기
    Class<? extends String> cls2 = String.class;
    System.out.println(cls2); // class java.lang.String
}
```

### 3-3. Class.forName() 으로 얻기

* 위 방식과 같이 컴파일 된 .class 파일이 있다면 클래스 이름만으로 Class 객체를 반환 받을 수 있다.
* 단 이때는, FULL PATH 를 적어주어야 한다.
* .class 파일을 찾지 못한다면, ClassNotFoundException 이 발생한다.
* 위 두가지 방법보다 가장 성능이 뛰어나다.

```java
public static void main(String[] args) {
    try {
        // 도메인.클래스명으로 얻기
        Class<?> cls3 = Class.forName("java.lang.String");
        System.out.println(cls3); // class java.lang.String
        
    } catch (ClassNotFoundException e) {}
}
```

> **\[INFO]**
>
> 대부분의 클래스 정보는 프로그램이 로딩될 때, 이미 Method Area 메모리에 적재된다.\
> 그런데 어떤 시스템은 모든 DB 데이터베이스와 연동 가능하다고 말하는데, 그렇다고 이 시스템을 컴파일 할 때, 모든 DB 드라이버를 컴파일 할 필요는 없다.\
> 시스템을 구동할 때 사용하는 DB 드라이버만 로딩하면 된다.\
> 이 때 forName() 을 통해서 메모리를 아끼면서 동적 로딩하면 좋다.
>
> **\[TIP]**
>
> Class 클래스 객체를 forName() 메서드를 통해서 가져오는 방법을 '동적 로딩' 이라고 부른다.\
> 보통 다른 클래스 파일을 불러올 때 JVM Method Area 에 클래스 파일이 같이 바인딩 되지만, forName() 으로 .class 파일을 불러올 때는, 컴파일에 바인딩되지 않고 런타임 때 불러오게 되기 때문에, 동적 로딩이라고 부른다.\
> 컴파일 타임에 클래스 유무가 확인되지 않아 예외를 처리해주어야 하는 이유이기도 하다.

## 4. Reflection API 기법

* Class 객체를 이용하면 클래스에 대한 모든 정보를 런타임 단에서 코드 로직으로 얻을 수 있다.
* 클래스 정보를 런타임에 얻을 수 있는 것은 매우 매력적이다.\
  \-> Class 객체만을 통해 본 클래스를 인스턴스화 하고, 메서드를 호출하는 등 동적인 코드를 작성할 수 있다.\
  \-> 이처럼 구체적인 클래스 정보를 알지 못해도, 그 클래스 인스턴스에 접근할 수 있게 해주는 방법을 Reflection API 라고 부른다.\
  \-> Reflection API 는 클래스 파일의 위치나, 이름만 있다면 해당 클래스의 정보를 얻어내고, 객체를 생성하는 것 또한 가능하게 해주어 유연한 프로그래밍을 가능하게 해준다.
* **Reflection 은 애플리케이션 개발보다 프레임워크, 라이브러리에서 많이 사용된다. 왜냐면 프레임워크 라이브러리는 사용하는 사람이 어떤 클래스명과 멤버들을 구성할지 모르는데, 이러한 사용자 클래스들을 기존 기능과 동적으로 연결시키기 위해서 리플렉션을 사용한다.**
