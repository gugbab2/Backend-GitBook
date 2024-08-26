---
description: 빈
---

# 스프링 컨테이너와 스프링 빈

## 1. 스프링 컨테이너 생성 과정

### 1-1. 스프링 컨테이너 생성

<figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 19.53.46.png" alt=""><figcaption></figcaption></figure>

* new AnnotationConfigApplicationContext(AppConfig.class) 등의 코드로 스프링 컨테이너를 생성할 때 구성 정보를 지정해 주어야 한다.

### 1-2. 스프링 빈 등록

<figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 19.54.40.png" alt=""><figcaption></figcaption></figure>

* 스프링 컨테이너는 파라미터로 넘어온 설정 클래스 정보를 사용해서 스프링 빈을 등록한다.\
  \-> 빈 이름은 보통 빈 메서드의 이름을 사용한다.

### 1-3. 스프링 빈 의존관계 설정

<figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 19.56.12.png" alt=""><figcaption></figcaption></figure>

* 스프링 컨테이너는 설정 정보를 참고해서 의존관계를 주입한다.
* 단순히 자바 코드를 호출하는 것 같지만, 이 차이는 뒤에 싱글톤 컨테이너에서 설명한다.

### 결론적으로 스프링 빈을 생성하고, 의존관계를 주입하는 단계가 나누어져 있다!

## 2. 스프링 빈 조회 - 상속관계

* 부모 타입으로 조회하면, 자식 타입도 함께 조회한다.
* 그래서 모든 자바 객체의 최고 부모인, Object 타입으로 조회하면 모든 스프링 빈을 조회한다.

### 2-1. BeanFactory 와 ApplicationContext

<figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 19.59.24.png" alt=""><figcaption></figcaption></figure>

* BeanFactory
  * 스프링 컨테이너의 최상위 인터페이스이다.
  * 스프링 빈을 관리하고 조회하는 역할을 담당한다.
  * getBean() 을 제공한다.\
    \-> 빈에 등록된 객체를 사용하고 싶을 때 사용하는 메서드!
* ApplicationContext
  * BeanFactory 의 기능을 모두 상속받아서 사용한다.
  * 애플리케이션을 개발할때는 빈을 관리하고 조회하는 기능 외에도 수 많은 부가기능이 필요한데 이 기능을 ApplicationContext 가 제공한다.
  * 메시지 소스를 활용한 국제화 기능
  * 환경변수 : 로컬, 개발, 운영 등을 구분해서 처리
  * 애플리케이션 이벤트
  *   편리한 리소스 조회 등 ..

      <figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 20.01.55.png" alt=""><figcaption></figcaption></figure>

### 2-2. BeanDefinition

* 스프링이 지원하는 다양한 컨테이너 설정 방식의 중심에는 BeanDefinition 이라는 추상화가 있다.
* 쉽게 말해 역할과 구현의 개념으로 명확하게 구분한 것이다.
  * XML 을 읽어서 BeanDefinition 을 만들면 된다.
  * 자바 코드를 읽어서 BeanDefinition 을 만들면 된다.
* BeanDefinition 을 빈 설정 메타정보라 말한다.
*   스프링 컨테이너는 해당 메타정보를 바탕으로 스프링 빈을 생성한다.

    <figure><img src="../../.gitbook/assets/스크린샷 2023-05-29 20.06.33.png" alt=""><figcaption></figcaption></figure>
