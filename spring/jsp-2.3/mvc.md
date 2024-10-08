# MVC 패턴

## 1. 모델 1 구조

* 모델 1의 구조는 웹 브라우저의 요청을 JSP 가 직접 처리한다.
* 웹 브라우저의 요청을 받은 JSP 는 자바빈이나 서비스 클래스를 사용해서 웹 브라우저가 요청한 작업을 처리하고 그 결과를 클라이언트에 출력한다.
* 이 상황은 JSP 페이지에 비지니스 로직을 처리하기 위한 코드와 뷰의 코드가 섞인다는 것을 의마 한다.\
  \-> 유지보수가 어려워진다.

## 2. 모델 2 구조

* 모델 1과 달리 웹 브라우저의 요청을 하나의 서블릿이 받는다. 서블릿은 웹 브라우저의 요청에 알맞게 처리한 후 그 결과를 보여줄 JSP 페이지로 포워딩한다.
* 포워딩 처리를 통해서 요청 흐름을 받은 JSP 페이지는 결과 화면을 클라이언트에게 전달한다.
* 가장 큰 특징은 비지니스 로직을 처리하기 위한 코드와 뷰의 코드가 분리된다는 것이다.

## 3. MVC(Model-View-Controller) 패턴

* 모델 : 비지니스 영역의 로직을 처리한다.
* 뷰 : 비니지스 영역에 대한 뷰를 담당한다.
* 컨트롤러 : 사용자의 입력 처리와 흐름 제어를 담당한다.
* MVC 패턴의 핵심은 다음과 같다.
  * 비지니스 로직을 처리하는 모델과 결과 화면을 보여주는 뷰를 분리한다.
  * 어플레케이션 흐름 제어나 사용자의 처리 요청은 컨트롤러에 집중된다.

## 4. MVC 패턴과 모델 2 구조의 매핑

* 컨트롤러 = 서블릿
* 모델 = 로직처리 클래스, 자바빈
* 뷰 = JSP
* 사용자 = 웹 브라우저 내지 휴대폰과 같은 다양한 기기

## 5. MVC 컨트롤러 : 서블릿

* 서블릿은 웹 브라우저의 요청과 웹 어플리케이션의 전체적인 흐름을 제어한다.
* 컨트롤러 역할을 하는 서블릿은 다음과 같은 과정을 거쳐 웹 브라우저의 요청을 처리하게 된다.
  * HTTP 요청을 받음
  * 클라이언트가 요구하는 기능을 분석
  * 요청한 비지니스로직을 처리하는 모델을 사용
  * 결과를 request 또는 session 에 저장
  * 알맞은 뷰 선택 후, 뷰로 포워딩

## 6. MVC 뷰 : JSP

* 비지니스 역할과 관련 없는 온전히 뷰의 역할을 담당한다.

## 7. MVC 모델 : 자바 클래스
