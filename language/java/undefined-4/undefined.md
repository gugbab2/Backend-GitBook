# 어노테이션 기본

## Annotation 이란?

* 컴파일러에게 정보를 알려주거나,&#x20;
* 컴파일할 때와 설치시의 작업을 지정하거나,
* 실행할 때 별도의 처리가 필요할 때 사용한다.
* 클래스와 인터페이스처럼 상속은 불가하다..&#x20;

## 미리 정해져 있는 어노테이션은 3개뿐(JDK 6까지)

* @Override : 해당 메서드가 부모 클래스에 있는 메서드를 오버라이딩 했다는 것을 명시적으로 선언한다.\
  \-> 부모의 메서드를 오버라이딩 했다는 것을 컴파일러에게 알려주는 것이기 때문에, 시그니처가 문제가 있을 시 컴파일 에러를 잡아준다.
* @Deprecated : 해당 클래스, 메서드가 더이상 사용되지 않는다는 것을 의미한다.&#x20;
* @SupressWarning : 간혹 코딩을 할때 컴파일러가 warning 을 알리는 경우가 있는데, 해당 상황에서 컴파일러에게 "개발자가 일부러 코딩하는 것이기 때문에, warning 할 필요가 없다." 고 컴파일러에게 미리 알려주는 것.

## 어노테이션을 선언하기 위한 메타 어노테이션

* @Target : 적용 대상을 지정한다.
  * CONSTROCTOR, FIELD, LOCAL\_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE 가 있다.

```java
@Target(ElementType.METHOD)
```

* @Retention : 얼마나 오래 어노테이션 정보가 유지되는지를 다음과 같이 선언한다.
  * SOURCE(컴파일시 사라짐), CLASS(클래스 파일에 존재), RUNTIME(실행시에 참조)

```
@Rentention(RetentionPoilcy.RUNTIME)
```

* @Documented : 해당 어노테이션에 대한 정보가 Javadocs(API) 문서에 포함된다는 것을 선언한다.
* @Inherited : 모든 자식 클래스에서 부모 클래스의 어노테이션을 사용 가능하다는 것을 선언한다.
* @Interface : 어노테이션을 선언할 때 사용한다.&#x20;
