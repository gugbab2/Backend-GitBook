# 컴포넌트 스캔과 의존관계 자동 주입하기

## 1. 컴포넌트 스캔과 의존관계 자동 주입하기

* `@Bean` or XML 설정 등을 통해서 의존관계를 지정할 수 있지만, 그 수가 수백개가 된다면 설정정보를 완벽하게 관리하는 것이 쉬운일은 아니다..
* 때문에 스프링은 설정 정보가 없어도 자동으로 스프링 빈을 등록하는 컴포넌트 스캔이라는 기능을 제공한다.
* 컴포넌트 스캔을 사용하려면 `@ComponentScan` 이라는 어노테이션을 사용하면 된다.
* 기존 설정과 달리 `@Bean` 으로 등록한 클래스가 하나도 없다.
* 추가적으로 컴포넌트 스캔은 `@Component` 이 붙은 클래스를 스캔해서 스프링 빈으로 등록한다.\
  &#xNAN;**(스프링 컨테이너 등록)**
* 마지막으로 `@Autowired` 를 생성자, 메서드, 필드 등에 붙여서 의존관계를 설정해준다.\
  &#xNAN;**(의존관계 설정)**

### 1-1. 탐색 위치와 기본 스캔 범위

* 탐색할 패키지를 지정할 수도 있지만 권장하는 방법은 아래와 같다.
  * 패키지 위치를 지정하지 않고, 설정 정보 클래스의 위치를 프로젝트 최상단에 위치시키자!
  * 최근 스프링 부트도 이 방법을 기본으로 제공한다.
* 컴포넌트 스캔의 용도 뿐만 아니라 다음 애노테이션이 있으면 스프링은 부가 기능을 수행한다.
  * `@Controller` : 스프링 MVC 컨트롤러로 인식
  * `@Repository` : 스프링 데이터 접근 계층으로 인식하고, 데이터 계층의 예외를 스프링 예외로 변환해준다.
  *   `@Configuration` : 앞서 보았듯이 스프링 설정 정보로 인식하고, 스프링 빈이 싱글톤을 유지하도록 추가

      처리를 한다.
  *   `@Service` : 사실 `@Service` 는 특별한 처리를 하지 않는다. 대신 개발자들이 핵심 비즈니스 로직이 여기에

      있겠구나 라고 비즈니스 계층을 인식하는데 도움이 된다.

### 1-2. 중복 등록과 충돌

* 자동 빈 등록 & 자동 빈 등록
  * 컴포넌트 스캔에 의해 자동으로 스프링 빈이 등록되는데, 이름이 같은 경우 스프링은 다음 오류를 발생시킨다.
  * `ConflictingBeanDefinitionException` 예외 발생
* 수동 빈 등록 & 자동 빈 등록
  * 스프링은 기본적으로 구체적인 것이 높은 우선순위를 갖는다.
  * _하지만 이런 경우 대부분 정말 잡기 어려운 버그를 만들어낸다...!!_
  * **때문에, 최근 스프링 부트에서는 해당 경우에 있어서 오류값을 발생시킨다.**
