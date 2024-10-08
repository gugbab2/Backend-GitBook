# Servlet

## 1. 서블릿 기초

* 서블릿 클래스를 구현하려면, HttpServlet 클래스를 상속받은 클래스를 작성해야 한다.
* 그 안에서 doGet(HttpServletRequest, HttpServletResponse) or doPost.. 등의 메서드를 사용해서 요청과 응답을 처리해 주어야 한다.
* 서블릿 2.5 버전까지는 web.xml 파일을 통해 설정을 해주어야 했지만, @WebServlet 어노테이션을 통해서 url 패턴을 매칭시킬 수 있다.

## 2. 서블릿 컨테이너

* 동적인 요청을 처리하기 위한 서블릿을 관리해주는 역할을 하는 것이 서블릿 컨테이너이다.
* 서블릿 컨테이너는 Clinet의 Request를 받아주고 Response할 수 있게, 웹 서버와 소켓을 만들어 통신한다.\
  \-> 대표적으로 톰캣이 있다.
* 서블릿 컨테이너의 역할은 다음과 같다.
  * **웹 서버와의 통신을 지원**
  * **서블릿의 생명주기를 관리한다.**
    * 서블릿 클래스를 로딩하여 인스턴스화
    * 초기화 메서드(init) 호출
    * 요청이 들어오면 적절한 서블릿 클래스 실행
    * 서블릿 소멸 시 GC 를 진행
  * **멀티쓰레드 지원 및 관리**
    * 요청이 들어올 때마다 쓰레드 생성
    * 쓰레드가 다 사용되면 자동적으로 소멸
    * 즉, 서블릿을 사용하는 것은 JVM이 각 요청을 _**분리된 자바 스레드 내부에서 처리하도록**_ 하는 것\
      \-> 서블릿의 장점이다.

## 3. 서블릿 로딩과 초기화

* 경로 지정을 통해서 해당 url 로 접근하게 되면, 해당 서블릿 클래스를 실행하게 된다.
* 서블릿 컨테이너는 **처음** 서블릿 클래스를 실행할 때 **서블릿 객체를 생성**한다.\
  \-> 이후 요청에 대해서는 기존에 만들어둔 서블릿 객체를 재사용한다.
* 웹 컨테이너가 서블릿 객체를 생성하고 init() 메서드를 호출하는 과정을 서블릿 로딩이라고 한다.\
  \-> init() 메서드를 통해서 필요한 초기화 작업을 수행한다.
* 서블릿 컨테이너는 서블릿을 초기화하기 위해서 ServletConfig 파라미터를 갖는 init() 메서드를 실행한다.
* **보통 초기화 작업은 상대적으로 시간이 오래 걸리기 때문에, 처음 서블릿을 사용하는 시점 보다는 웹 컨테이너를 처음 구동하는 시점에 초기화를 진행하는 것이 좋다.**
