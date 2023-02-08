# REST API

## API (Application Programming Interface)

* ### Interface
  * 인터페이스 : 각각의 엔드포인트 간의 연결(매개체)이라 생각할 수 있다.
    * API(Application Programming Interface) : 컴퓨터나 컴퓨터 프로그램 간의 연결이라 볼 수 있다.
    * UI(User Interface) : 사람과 컴퓨터 간의 연결이라 볼 수 있다.
    * 객체 기준의 인터페이스 : 객체가 정의하는 연산의 모든 시그니처(메서드의 선언부)들을 일컫는 말로 인터페이스는 객체가 받아서 처리할 수 있는 연산의 집합이다.\
      \-> 구현을 다루지 않고 명세에 대한 부분이다(이를 통해서 외부에 객체 내부에서 어떤일이 일어나는지를 감출 수 있다)
  * 인터페이스의 특징&#x20;
    * Communication : 각각의 엔드포인트 간의 소통을 위함이다.
    * Specification : 소통을 위한 명세이다.
    * Information Hiding (Principle) : 연결점만 존재하지 내부적으로 어떤 일이 일어나는지는 알 수 없다.\
      \-> 정보 은닉이 부족하다면 인터페이스라 보기 애매하다;;
    * Encapsulation (Technique) : 캡슐화는 정보를 내부에 넣어 관리한다는 느낌으로 생각하고 있자.. \
      \-> 정보 은닉과 캡슐화는 상당히 밀접한 관계를 맺고 있다.
    * Implementation : 인터페이스와 별개로 구현이 이루어진다.

## REST(Representation State Transfer)

로이필딩 논문 : [Representational State Transfer (REST)](https://www.ics.uci.edu/\~fielding/pubs/dissertation/rest\_arch\_style.htm)

\-> REST 는 API 를 위한 아키텍처 스타일은 아니다. WEB 에 특화된 방법으로, API 를 만들 때 유용하게 사용할 수 있다.\
\-> API 를 사용하는 방법론 중 하나(현재 가장 일반적으로 사용되는 방법론이다)\
\-> REST 는 아키텍처(설계된 것)를 위한 아키텍처 스타일(설계의 방법)이다!

*   ### &#x20;제약조건&#x20;

    1️⃣ **Starting with the Null Style**&#x20;

    2️⃣ **Client-Server**

    3️⃣ **Stateless -> HTTP 프로토콜의 가장 큰 특징**

    4️⃣ **Cache**

    5️⃣ **Uniform Interface → 핵심!**

    1. 다른 아키텍처 스타일과 다른점은 **균일한 인터페이스를 강조**한다는 것이다.
    2. 일반성 이라는 원리를 적용해 **전체 시스템 아키텍처를 단순화하고 상호작용 가시성을 개선**했다.
    3. **구현은 제공하는 서비스와 분리되어 독립적인 진화 가능성을 장려한다.**
    4. 하지만 단점은, 애플리케이션의 요구에 특화 된 것이 아니라 표준화된 형태로 전송되기 때문에 균일한 인터페이스가 효율성을 저하 시킨다는 것입니다.(**컨트롤 URI??**) -> 모든 상황을 REST 로 컨트롤 할 수 없다.
    5. **REST 인터페이스는 대용량 하이퍼미디어 데이터 전송에 효율적으로 설계되어 웹의 일반적인 경우에 최적화되지만** 결과적으로 다른 형태의 아키텍처 상호 작용에는 최적화되지 않습니다 \
       \-> 웹 말고 다른  형태의 아키텍처와 상호작용하기 쉽지 않다;;
    6. **필딩 제약 조건 -> 범용적으로 REST 를 말할 때 중요시 되는 부분이다.**
       1. Four Interface Constraints
          1. URI 등으로 리소스를 식별할 수 있다. -> 리소스 : 자원(명사)
          2. 표현으로 리소스를 조작한다 -> 표현 : 행위(동사)
          3. 메시지(표현)는 자기 서술적이기 때문에, 여러 레이어에서 처리/변환 가능하다...??\
             \-> 여러 레이어에서 처리 변환이 가능하기 위해서는 범용적이어야 한다.\
             (모두가 알아 들어 먹어야 한다)
          4. HATEOAS(Hypermedia as the Engine of Application State)\
             \-> REST 저자는 강조하는 부분 중 하나다..
       2. 아키텍처 요소(5.2)에서 리소스와 표현을 구분
          1. Resource -> 추상, 실체가 아니라 모든 시간에 통용되는 엔티티의 집합이다.
          2. Representation(Meta Data + Data) -> 사실 상 HTTP METHOD
       3. URI 파트(6.2)에서 리소스에 대해 다시 강조
       4. 아키텍처 데이터 뷰(5.3.3)에서 HATEOAS에 대해 언급.
          1. 표현에 선택 가능한 상태 전환이 포함돼야 한다. -> 이게 바로 하이퍼미디어 링크
          2. 대부분은 효율 문제로 표현에 링크를 넣지 않고, 클라이언트 개발자가 API 문서를 활용해 처리한다. 표현에서 상태 전환을 선택하는 게 아니라, API 문서를 참조해서 상태 전환을 강제하는 것.
          3.  [Richardson Maturity Model](https://martinfowler.com/articles/richardsonMaturityModel.html)

              \<RESTful Web API>의 공저자인 레오나르드 리처드슨은 Hypermedia Control(대표적인 게 바로 링크)을 강조.

              \-> 성숙한 REST라면 표현(API Response value)에 하이퍼미디어 컨트롤(링크)이 포함되어야 한다.\
              \-> **일반적(업계)으로는 2단계(리소스, 표현) 까지만 해도 RESTFUL 하다고 부른다.**



    6️⃣ **Layered System**

    7️⃣ **Code-On-Demand**
