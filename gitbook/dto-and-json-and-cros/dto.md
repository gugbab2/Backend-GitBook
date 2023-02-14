# DTO

### DTO (Data Transfer Object)

> [DTO](https://martinfowler.com/eaaCatalog/dataTransferObject.html)

> ~~~~[~~Remote Facade~~](https://martinfowler.com/eaaCatalog/remoteFacade.html)~~~~

#### **제약 조건**

* “between processes”
* ~~“working with a remote interface, such as Remote Facade”~~
* 다른 프로세스와 통신, 그것도 원격(네트워크를 통해)으로!\
  \-> 아래 나오는 IPC 라고 부른다

#### [**IPC (Inter-Process Communication)**](https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4\_%EA%B0%84\_%ED%86%B5%EC%8B%A0)

* 서로 다른 프로세스, 거칠 게 이야기하면 서로 다른 프로그램이 서로 통신.
* B/E와 F/E로 Tier를 나누면 IPC가 필수적이다.
* IPC에서 쓸 수 있는 기술
  * File → 가장 기본적인 접근. 원격 환경에서 활용하기 어렵다.(너무 원시적으로 보인다..)
  * Socket → 파일과 유사하게 읽고 쓸 수 있지만, 원격 환경에서도 활용할 수 있다.
    * HTTP 같은 고수준 프로토콜을 활용하면 어느 정도 정해진 틀이 있기 때문에 상대적으로 쉬워진다.
    * REST를 활용하면 리소스에 대한 CRUD로 정리해야 함.\
      \-> 리소스를 표현을 기준으로 API 를 명확하게 정의함.&#x20;

REST 에선 표현(HTTP 메서드)을 다뤄야 하고, 이를 위해 사실상 데이터를 담는 것 외엔 아무 것도 하지 않아서 제대로 된 객체라고 볼 수 없는 특별한 객체를 사용하게 된다.(객체 지향에서 지양해야만 하는 데이터 덩어리 객체..)\
\->  객체지향 프로그래밍에서는 캡슐화를 통해서 데이터를 외부에 공개해서는 안된다!

* [“무기력한 도메인 모델” 안티패턴](https://martinfowler.com/bliki/AnemicDomainModel.html)

#### **DTO**

* 아주 단순하게 보면 setter와 getter로만 이뤄짐.\
  \-> 객체 지향에서는 setter, getter 를 통해서 객체 내부의 값을 드러내서는 안된다.. (중요하기 때문에 반복!)
* 제대로 된 객체가 아니라 그냥 무기력한 데이터 덩어리. C/C++ 등에선 구조체(struct)로 구분할 수 있지만, Java에선 불가능. 최신 Java에선 record를 활용할 수 있지만, 오래된 Bean 관련 라이브러리에선 지원하지 않음.

#### **Tier간 통신(완전하게 분리된 상황을 가정)**

* F/E와 B/E 사이
  * DTO 자체를 그대로 전송할 수는 없고, 직렬화(마샬링)를 통해야 한다.
  * 어떤 직렬화 기술을 사용할 건지도 결정해야 함. → XML, JSON \
    \-> 최근 거의 표준인 JSON 은 직렬화 기술 중 하나이다
* B/E와 DB 사이
  * 아주 옛날에는 Value Object를 DTO란 의미로 썼지만, 재빨리 Transfer Object라고 정정함. 아직도 한국의 오래된 SI 기업에서는 VO(Value Object)를 DTO란 의미로 사용(DAO와 VO를 쓰고 있다면 대부분 여기에 속함). -> 우리회사..

“Data Transfer”란 측면에 집중하면, “원격(remote)”이 아닌 경우에도 DTO를 사용할 수 있다.\
\-> 애플리케이션 안에 레이어 안에서도 DTO 를 사용할 수 있다.
