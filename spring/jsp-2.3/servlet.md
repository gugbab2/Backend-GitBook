# Servlet

## 1. 서블릿이란?

* 자바를 기반으로 하는 웹페이지를 동적으로 만들어줄 수 있는 일종의 프로그램을 말한다.
* **사실 좁게보면 서블릿이란 위와 같은 기능을 하는 자바의 클래스를 뜻한다.**&#x20;
* **넓게 보면 위 기능을 수행하기 위한 자바의 패키지를 뜻한다.**

### 서블릿의 등장배경&#x20;

* 서버 부분에서 이야기했다시피 과거 서버는 정적인 자료(주로 HTML 문서)만 주고받을 수 있었다.
* 초기의 클라이언트가 자료를 요청하면 서버는 미리 만들어진 자료(정적)를 저장하고 있다가 반환했다.
* 하지만 인터넷 사용자가 많아지고 다양한 기능을 웹을 통해 구현하고자 하는 움직임이 많아졌다.
* 사용자는 정적인 자료가 아닌 자기 필요에 맞는 자료를 웹페이지를 통해 제공 받고 싶어했다.
* 그리고 사용자 요구에 맞춰 동적으로 반응하는 페이지을 만들기 위해 만들어진 것이 서블릿이다.

### 서블릿과 서버&#x20;

#### 과거의 서버 구조&#x20;

* 과거 정적인 웹페이지를 제공할 때는 아래와 같이 서버 구성이 단순했다.&#x20;
* 사용자가 요청하면 이미 저장된 페이지를 반환하기만 했다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 16.05.26.png" alt=""><figcaption></figcaption></figure>

#### 현대의 서버 구조&#x20;

* 동적인 웹페이지의 필요성이 강조 되면서 서버는 연산 기능까지 가지게 되었다.&#x20;
* 그러면서 과거의 서버라고 불리던 것으로 크게 WEB, WAS(Wep Application Server) 로 나뉘게 되었다.&#x20;
* 아래 그림에서 볼 수 있듯 WEB 서버는 사용자의 요청에 따라 정적인 페이지를 제공한다. 그리고 WAS는 사용자의 요청 중 연산이 필요한 부분을 맡아서 그 결과를 계산한다. 그리고 WAS는 연산 결과를 웹서버로 제공하고 웹서버 정적 페이지를 만들어 사용자에게 전달한다.
* 이때 WAS에서 연산을 담당하는 것이 서블릿이다.
* 서블릿은 WAS 안에 있는 **서블릿 컨테이너** 또는 **웹 컨테이너**라고 불리는 공간에서 활용하게 된다.

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 16.07.02.png" alt=""><figcaption></figcaption></figure>

### 웹 컨테이너란(서블릿 컨테이너)?

* 서블릿 컨테이너라고도 불리는 웹 컨테이너는 서블릿을 이용해 사용자 요청을 처리하는 역할을 한다.
* 보통 일반적으로 홈페이지를 들어가면 한 가지 기능만 제공하는 홈페이지는 거의 없다. 쇼핑몰 하나만 생각해도 상품 등록, 장바구니, 게시판, 회원 가입 등 다양한 기능을 한다.
* 그렇다는 이야기는 각각의 기능을 구현할 다양한 서블릿이 한 서버 안에서 작동한다는 것이다. 하지만 이러한 서블릿이 서버에서 잘 운용되도록 일일이 컨트롤하는 것은 쉬운 일이 아니다.
* 웹 컨테이너는 이러한 다양한 서블릿이 고객의 요청에 따라 작동하도록 서블릿을 제어한다. 이를 통해 효율적으로 서버 관리 및 사용자 요청을 처리할 수 있다.

## 2. 서블릿 패키지 구조

* 우선 서블릿은 클라이언트의 동적인 동작을 처리하기 위해서 만들어졌고, 자바 클래스의 형태를 가지고 있다.
* 서블릿은 `javax.servlet.http` 라는 패키지 안에 포함되어있다.
* 선언부를 보면 알 수 있듯이 `HttpServlet` 클래스를 상속받아 사용이 가능하다.

```java
public class Test extends HttpServlet {

	protected void doGet(HttpServletRequest request, HttpServletResponse response) 
		throws ServletException, IOException {
		response.getWriter().append("Served at: ").append(request.getContextPath());
	}
	
	protected void doPost(HttpServletRequest request, HttpServletResponse response) 
		throws ServletException, IOException {
		doGet(request, response);
	}
	
}
```

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 16.20.21.png" alt="" width="563"><figcaption></figcaption></figure>

### Servlet 인터페이스&#x20;

* 각종 서블릿 클래스를 구현하는 가장 기본 토대가 되는 인터페이스. 해당 인터페이스를 구현하면 서블릿 클래스를 구현하여 사용할 수 있다.
* 해당 인터페이스를 구현하기 위해서는 아래 5개의 메서드를 구현해야 한다.
* 이때 `init()`, `service()`, `destroy()` 메서드는 "**life-cycle 메서드"** 라고 한다.

#### init()&#x20;

* 웹 컨테이너는 서블릿 객체를 생성하며 서블릿 생명 주기 중 단 한번 `init()`메서드를 호출한다.
  * **즉 서블릿 객체 생성 이후에 서블릿의 초기화는 단 한번만 이뤄진다는 것이다.**
* 서블릿은 `init()` 메소드가 완료되어야 요청 처리가 가능하다.
* 이때 초기화 중 필요한 정보는 `ServletConfig` 인터페이스 타입의 객체를 매개변수로 받는다.
* **그리고 `init()` 메서드는 일반적으로 클라이언트가 서블릿을 처음 요청할 때 호출된다.**
  * **하지만 설정으로 요청 없이 서버 시작 시 서블릿을 로드하고 `init()`메서드를 호출할 수도 있다.**\
    **-> 이 경우 서버 시작 시간은 오래 걸리겠지만, 첫 요청부터 빠른 응답을 받을 수 있다.**&#x20;

#### service()

* 웹 컨테이너에 의해 호출되어 서블릿이 클라이언트의 요청에 응답할 수 있도록 하는 메서드.
* `service()` 메서드는 `init()` 메서드가 성공적으로 실행된 이후 호출이 가능하다.
* 클라이언트의 요청을 전달하는 `ServletRequest` 객체를 통해 웹 컨테이너에 정보가 전달되면 `service()` 메서드가 실행되고 service() 메서드는 요청에 따른 메서드를 실행한다.
  * 예를 들면, `doGet()`, `doPost()`, `doPut()`, `doDelete()` 와 같은 메서드가 있다.
* 그 후 실행에 따른 결과물을 클라이언트에게 요청을 전달하는 `ServletResponse`로 응답한다.
* 이때 중요한 점은 다음과 같다.&#x20;
  * **웹 컨테이너는 다중 요청이 들어와도 매번 서블릿 클래스를 로드하고 초기화하지 않고, 한번 로드 되었다면 로드된 서블릿 인스턴스를 사용한다.**&#x20;
  * **때문에, 서블릿 인스턴스가 한번 로드되고 사용되었다면 `init()` 메서드는 2번 호출되지 않는다!**
  * **그리고 사용자 요청이 올 때마다 스레드를 만들고 사용한다. 일반적으로 서블릿 내 스레드 풀을 관리하여 사용한다.**&#x20;

#### destroy()

* 웹 컨테이너에 의해 호출되어 수명이 다한 서블릿을 서비스에서 제외되도록 하는 메서드. 일반적으로 개발자가 아닌 웹 컨테이너에 의해 자동으로 수행된다.
* 이 또한 `init()` 메서드와 같이 단 한번만 실행된다.
* 만약 서버를 끄거나 재시작할 경우에도 해당 메서드가 실행된다.

#### getServletConfig()&#x20;

* 서블릿의 초기화를 위한 정보를 포함하는 `ServletConfig`의 객체를 반환하는 메서드.
* 참고로 이때 반환되는 객체는 이미 한번 `init()` 메서드로 전달된 상태의 객체다.

#### getServletInfo()&#x20;

* 서블릿에 관한 다양한 정보(버전, 저작권자, 작성자)를 반환함.

### ServletConfig 인터페이스&#x20;

* 초기화 중에 서블릿에 정보를 전달하기 위해 서블릿 컨테이너에서 사용하는 서블릿 구성을 위한 객체.

### GenericServlet 추상 클래스&#x20;

* `Servlet` 인터페이스와 `ServletConfig` 인터페이스를 구현하여 만든 추상 클래스다.
* 기존 `Servlet` 인터페이스를 구현하기 위해서는 일일이 메서드를 따로 만들어줘야했다. 게다가 `ServletConfig` 인터페이스까지 같이 구현해야 했다.
* 이를 조금 더 수월하게 하기 위해 만든 것이 `GenericServlet` 추상 클래스다.
* `GenericServlet`은 `Servlet`의 라이프사이클 메서드 중 `init()`, `destroy()`를 간단하게 제공한다.
* 즉, 사용자는 `service()`만 구현하여 서블릿을 간편하게 실행할 수 있다.

### HttpServlet 클래스&#x20;

* `GenericServlet`를 상속하여 만든 추상 클래스다.
* `HttpServlet`는 `GenericServlet`과 마찬가지로 상속받으면 간단하게 서블릿을 실행할 수 있다. 그리고 우리가 대부분 사용하는 서블릿은 `HttpServlet` 추상 클래스를 상속받은 클래스다.
* `GenericServlet` 이 `init()`, `destroy()`까지 구현했다면 `HttpServlet`는 `service()`까지 구현되있다.
* 다만 이름에서 알 수 있듯이 `GenericServlet`은 Http뿐 아니라 다양한 프로토콜에 대응이 가능하다. 즉 Http 프로토콜을 이용하지 않을 때도 사용이 가능하다.
* 하지만 `HttpServlet`는 웹개발서 주로 쓰는 프로토콜이 http라서 이를 편하게하려고 만들었다. 그래서 `HttpServlet`는 Http 프로토콜에만 한정적으로 사용할 수 있다.

## 3. 웹 컨테이너에서 서블릿 실행 순서&#x20;

* `HttpServlet` 클래스 사용 예제&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2024-11-02 16.43.20.png" alt=""><figcaption></figcaption></figure>

#### 1. 클라이언트 요청&#x20;

* 클라이언트는 HTTP 프로토콜을 통해서 원하는 데이터를 요청(Request)한다.
* 이때 고객의 요청이 정적 페이지 요청이라면 WEB서버를 통해서 바로 처리해준다.
* 반대로 동적인 페이지에 대한 요청이라면 고객의 요청은 웹 컨테이너(서블릿 컨테이너)로 넘어간다.

#### 2. HttpServletRequest, HttpServletResponse 객체 생성&#x20;

* 서블릿 컨테이너로 요청이 들어오게 되면 스레드를 생성하고 스레드 스택에 요청, 응답을 담당하는 객체인 `HttpServletRequest`, `HttpServletResponse` 의 참조값을 가지고 있는 변수를 생성하게 된다.&#x20;
* `HttpServletRequest` 객체는 고객의 요청 정보를 담은 객체로서 웹 컨테이너에 의해 생성된다. 이 객체는 웹 컨테이너에서 고객 요청 정보를 담고 해당 정보가 처리될때까지 웹 컨테이너를 돌아다닌다.
* 반대로 `HttpServletReponse` 객체는 요청에 따른 처리 정보를 담은 객체로서 웹 컨테이너에 의해 생성된다. 그리고 이 객체는 고객 요청의 처리에 따라 떠돌다가 최종적으로는 고객의 요청에 응답하게 된다.

#### 3.  Web.xml 파싱&#x20;

* **Web.xml 은 서버를 초기화 할때, 서블릿의 위치가 어디있는지 설정을 도와주는 문서다.**
* 즉 어떤 서블릿이 프로젝트 폴더 내에 있다면 어느 위치에 존재하고 어떤 URL로 접속해야하는지 알려준다.
* 컨테이너는 고객 요청 정보를 파악한 이후에 Web.xml 문서에 따라 그에 맞는 서블릿 주소를 찾는다. 그리고 서블릿에 접근을 하게 된다.
* **어노테이션 방식을 사용하면 Web.xml 을 통해 서블릿 클래스를 매핑해주지 않아도 된다.**&#x20;

#### 4. 서블릿 초기화

* 컨테이너가 실행할 서블릿을 찾았다면 이제 해당 서블릿 클래스를 로드하게 된다.
* **그리고 이 객체를 이용해 `init()` 메서드를 호출하여 서블릿을 사용할 수 있도록 초기화한다.**
  * 이때 매우 중요한 점은 위 과정은 한 개의 서블릿이 처음 요청될 때 수행된다는 점이다.\
    &#x20;(Lazy Initialization)
  * 앞서 `service()` 메서드에서 설명했듯 서블릿은 사용자가 요청할 때마다 새로 초기화되지 않는다.
  * 클래스 로드, 서블릿 인스턴스 생성, `init()`은 서블릿 생명 주기에서 단 한번만 수행된다.
  * 이때 만들어지는 인스턴스는 싱글톤 패턴에 따른 것으로 주기 내내 단 한번 만들어진다.
* **많은 요청이 다중으로 들어오면 이 한 개의 인스턴스를 사용해 `service()` 메서드를 실행한다.**
  * **하나의 요청마다 하나의 스레드가 생성된다.**&#x20;
  * **이로 인해 불필요한 자원 낭비를 막을 수 있고, 효율적인 자원 관리가 가능해진다.**&#x20;

#### 5. service() 실행&#x20;

* 초기화가 진행된 이후에는 고객의 요청을 실질적으로 처리하는 `service()` 메서드가 실행된다.
* `service()`는 고객 요청이 get방식이냐 post방식이냐에 따라 거기에 맞는 메서드를 실행한다.
  * 고객의 요청이 get방식으로 전달됐다면 `doGet()` 메서드를 실행한다.
  * 고객의 요청이 post방식으로 전달됐다면 `doPost()` 메서드를 실행한다.

#### 6. destroy() 실행

* 더이상 사용되지 않는다고 판단하거나 서버가 종료될 때 컨테이너는 `destroy()`를 실행한다.

#### 7. HttpServletRequest, HttpServletResponse 객체 소멸&#x20;

* 요청 , 응답이 끝나면 스레드는 스레드 풀로 반납이 된다.&#x20;
* 이후, `HttpServletRequest`, `HttpServletResponse` 객체에 대한 참조가 제거가 되며 GC 대상이 된다.&#x20;
