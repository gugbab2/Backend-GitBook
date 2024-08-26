# AOP

## 1. AOP(Aspect-Oriented Programming) 란?

* 관점 지향 프로그래밍\
  \-> 어떤 로직을 기준으로 핵심적인 관점, 부가적인 관점으로 나누어 그 관점을 기준으로 각각 모듈화를 하겠다는 의미\
  \-> DI 가 의존성 주입이라면, AOP 는 로직 주입이라고 할 수 있다.
* 횡단(부가적인) 관심사 : 다수의 모듈(입금, 출금, 이체) 에서 **공통적으로 나타나는 부분(로깅, 보안, 트랜잭션)을** 의미한다.
* 핵심 관심사 : 로직에서 횡단 관심사를 제외한 부분으로, 모듈마다 다르다.

<figure><img src="../../.gitbook/assets/image (63).png" alt=""><figcaption></figcaption></figure>

## 2. OOP / AOP

* OOP, AOP 는 대칭되는 개념이 아닌, 상호 보완관계의 있는 패러다임이다.
* AOP 는 패러다임으로 각 언어마다 구현체가 존재한다.\
  \->Java : AspectJ

## 3. AOP 주요 개념

<figure><img src="../../.gitbook/assets/image (52).png" alt="" width="375"><figcaption></figcaption></figure>

### 3-1. 핵심 관심사 영역

<figure><img src="../../.gitbook/assets/image (55).png" alt="" width="375"><figcaption></figcaption></figure>

* Target Object
  * **횡단 기능이 적용될 객체로**, 핵심 기능을 가지고 있다.
* JoinPoint
  * **타겟 안에서 횡단 기능(Advice)이 적용될 수 있는 여러 위치**를 의미한다.\
    \-> 스프링 AOP 에서는 메서드로 한정하고 있다.

### 3-2. 횡단 관심사 영역

<figure><img src="../../.gitbook/assets/image (59).png" alt="" width="375"><figcaption></figcaption></figure>

* Aspect
  * 횡단 관심사를 모듈화한 것이다.
* Advice
  * **JoinPoint 에 적용할 횡단 코드이다.**
  * **어떤 부가기능?**
* Point cut
  * 실제 advice 가 적용될 지점, Spring AOP 에서는 advice 가 적용될 메서드를 선정한다.\
    \-> 어노테이션으로 설정할수도 있다.

## 4. AOP 특징

* 프록시 패턴 기반의 AOP 구현체이다.
  * 프록시 객체를 쓰는 이유는 기존 코드 변경 없이 접근 제이 및 부가 기능을 추가하기 위함.
* 스프링 빈에만 AOP 를 적용할 수 있음.

### 4-1. AOP 구현 방법

* 컴파일 시점에 코드에 공통 기능을 삽입하는 방법
* 클래스 로딩 시점에 바이트 코드에 공통 기능을 삽입하는 방법
* **런타임에 프록시 객체를 생성해 공통 기능을 삽입하는 방법(프록시 패턴)**\
  **-> 스프링이 채택한 방법**

### 4-2. 프록시 패턴

* 실제 기능을 수행하는 객체 대신 가상의 객체를 사용하여 로직의 흐름을 제어하는 디자인 패턴
* 특징
  * 원래의 기능보다 그 외의 부가적인 작업(로깅, 인증, 트랜잭션 등..) 을 별도로 수행할 수 있다.
  * 비용이 많이 드는 연산(DB 쿼리, 대용량 텍스트 파일 등) 을 실제로 필요한 시점까지 미룰 수 있다.

## 5. Spring AOP

### 5-1. AOP 동작 원리

<figure><img src="../../.gitbook/assets/image (39).png" alt=""><figcaption></figcaption></figure>

* AOP 는 프록시 패턴을 사용한다.
* 프록시는 타겟을 감싸서 타겟의 요청을 대신 받아주는 Wrapping 오브젝트이다.
* 클라이언트에서 타겟을 호출하게 되면, 타겟이 아닌 프록시가 호출되고, 이 때 프록시는 타겟 메소드 실행 전후로 부가 기능을 실행하도록 구성되어 있다.
* 다만, 프록시 패턴은 타겟 하나하나마다 프록시 객체를 정의해야 하기에, 너무 번거롭다(OOP).. && 코드의 중복..
* 그래서 Spring AOP 에서는 런타임시에 JDK Dynamic Proxy 또는 CGLIB 를 활용하여 프록시를 생성해준다.\
  \-> 타겟이 하나 이상의 인터페이스를 구현한다면 JDK Dynamic Proxy / 아니라면, CGLIB 방식 채택
