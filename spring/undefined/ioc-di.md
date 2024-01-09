# IoC / DI

## 1. IoC(Inversion of Control)

* 제어의 역전이라는 의미로, **스프링에서 오브젝트(빈)의 생성과 의존 관계 설정, 사용, 제거 등의 작업을 코드 대신 스프링 컨테이너가 담당한다.**\
  \-> 이를 **스프링 컨테이너가 코드 대신 오브젝트에 대한 제어권을 가지고 있다고 해서 IoC 라고 부른다.**
* 따라서, 스프링 컨테이너를 IoC 컨테이너라고 부른다.\
  \-> 빈을 IoC 컨테이너에서 관리하므로, 개발자는 비지니스 로직에 집중할 수 있다.

### 1-1. IoC 컨테이너란?

* 스프링에서는 IoC 를 담당하는 컨테이너를 빈 팩토리, DI 컨테이너, 애플리케이션 컨텍스트 라고 다양하게 부른다.
* 오브젝트 생성과 오브젝트 사이의 런타임 관계를 설정하는 **DI 관점**으로 바라볼 때, 컨테이너를 **빈 팩토리, 또는 DI 컨테이너**라 부른다.
* 그러나 스프링 컨테이너는 **단순한 DI 작업보다 더 많은 일을 하는데**, DI 를 위한 빈 팩토리의 여러가지 기능을 추가한 것을 **애플리케이션 컨텍스트**라고 부른다.
* 정리하면, 애플리케이션 컨텍스트는 그 자체로 IoC와 DI 그 이상의 기능을 가졌다고 생각하면 된다.

### 1-2. 빈 팩토리와 애플리케이션 컨텍스트의 관계

<figure><img src="../../.gitbook/assets/image (36).png" alt=""><figcaption></figcaption></figure>

* 빈 팩토리
  * 스프링 컨테이너의 최상위 인터페이스
  * 스프링 빈을 관리하고 조회하는 역할을 담당한다.
  * 대표적으로 getBean() 메서도를 제공한다.
* 애플리케이션 컨텍스트
  * 애플리케이션 컨텍스트는 빈 팩토리의 기능을 모두 상속받아서 사용한다.
  * 때문에 빈 팩토리의 없는 다음의 기능들을 가지고 있다.
    * 메시지 소스를 활용한 국제화 기능
    * 환경변수
    * 애플리케이션 이벤트
    * 편리한 리소스 조회
    * ...

### 1-3. 설정 메타 정보

* IoC 컨테이너의 가장 기초적인 역할은 오브젝트를 생성하고 관리하는 것이다.\
  \-> 스프링 컨테이너가 관리하는 이러한 오브젝트를 빈이라고 한다.
* 스프링 컨테이너는 자바코드(생성자, 수정자...), XML, Groovy 등 다양한 설정 정보를 받아들일 수 있도록 유연하게 설계되어 있다.

<figure><img src="../../.gitbook/assets/image (61).png" alt=""><figcaption></figcaption></figure>

### 1-4. 스프링 빈 설정 메타 정보(BeanDefinition)

<figure><img src="../../.gitbook/assets/image (65).png" alt=""><figcaption></figcaption></figure>

* 스프링이 이러한 다양한 설정을 제공하는데 있어 그 중심에는 BeanDefinition 이라는 추상화가 있다.
* 쉽게 말하면, 설정정보(XML, 자바코드, 어노테이션) 를 읽어서 BeanDefinition 을 만든다.\
  \-> 따라서 스프링 컨테이너는 오직 BeanDefinition 만 알면 된다.(이상적인 캡슐화) 로 볼 수 있다.

## 2. DI(의존관계 주입)

### 2-1. 의존관계(Dependency)

* 프로그래밍에서 의존이란 A 객체를 수정할 때 B 의 기능이 추가되거나 변경되면 두 객체는 서로 의존관계에 있다고 볼 수 있다.
* 객체지향 프로그래밍에서는 구체화된 객체를 의존하는 것이 아닌, 인터페이스를 의존하게 되면, 확장성 있는 의존관계를 맺을 수 있다.

### 2-2. 의존관계 주입(Dependency Injection)

* 의존 관계를 외부에서 결정하는 것을 DI 라고 한다.\
  **-> DI 를 통해서 객체간의 의존성이 줄어든다.(요구사항이 변경되었을 때 구현체만 변경하면 된다)**\
  **-> 이를 통해 가독성 또한 높아진다.**
* 스프링에서는 외부의 대상이 IoC 컨테이너가 되어, 빈을 알아서 주입해 준다.

<figure><img src="../../.gitbook/assets/image (64).png" alt=""><figcaption></figcaption></figure>

### 2-3. 의존관게 주입의 방법

* 필드 주입
* 수정자 주입
* 생성자 주입

### 2-4. 순환 참조

* 순환 참조란 서로 다른 여러 빈들이 서로를 참조하고 있음을 의미한다.
* **필드 주입, 수정자 주입(하지마!)**
  * **각각의 빈들이 서로의 객체를 사용하며, 호출을 반복하다가, StackOverflowError 를 발생시키고 죽는다..**\
    **-> 이처럼 필드 주입이나, 수정자 주입은, 객체 생성 후 비지니스 로직 상에서 순환 참조가 일어나기 때문에, 컴파일 단계에서는 순환참조를 잡아낼 수 없다.**

```java
@Service
public class CourseServiceImpl implements CourseService {

    @Autowired
    private StudentService studentService;

    @Override
    public void courseMethod() {
        studentService.studentMethod();
    }
}

@Service
public class StudentServiceImpl implements StudentService {

    @Autowired
    private CourseService courseService;

    @Override
    public void studentMethod() {
        courseService.courseMethod();
    }
}
```

* **생성자 주입(권장)**
  * **반대로 생성자 주입의 경우, 초기의 빈을 등록하는 과정에서 순환 참조를 발견 할 수 있다.**\
    **-> 컴파일 타임!**

```java
@Service
public class CourseServiceImpl implements CourseService {

        private final StudentService studentService;

    @Autowired
    public CourseServiceImpl(StudentService studentService) {
        this.studentService = studentService;
    }

    @Override
    public void courseMethod() {
        studentService.studentMethod();
    }
}

@Service
public class StudentServiceImpl implements StudentService {

    private final CourseService courseService;

    @Autowired
    public StudentServiceImpl(CourseService courseService) {
        this.courseService = courseService;
    }

    @Override
    public void studentMethod() {
        courseService.courseMethod();
    }
}
```

### 2-5. @Autowired

* DI 를 할 때 사용하는 어노테이션으로, 의존 관계의 타입에 해당하는 빈을 찾아 주입하는 역할을 한다.
* 스프링 서버가 올라갈 때 애플리케이션 컨텍스트가 @Bean, @Service, @Controller 등 어노테이션을 이용하여 등록한 빈을 생성하고, @Autowired 어노테이션이 붙은 위치에 의존관계 주입을 실행하게 된다.
* 해당 어노테이션에 빈을 주입하는 것은 BeanPostProcessor 라는 내용을 찾을 수 있고, 그것의 구현체는 AutowiredAnnotationBeanPostProcessor 인 것을 확인할 수 있다.

## 3. 결론(DI 와 IoC 의 차이는?)

* DI : 의존관계를 어떻게 가질 것인가?
* IoC : 누가 소프트웨어의 제어권을 가지고 있는가?\
  \-> 스프링 컨테이너가 빈을 생성할 때 빈들간의 의존관계를 DI 를 통해서 해결한다.
* DI 는 IoC 를 필수적으로 사용하지 않는다.\
  \-> 자바 코드(생성자, 수정자) 를 통해서 의존성을 직접 주입할 수도 있다.
