# 엔티티 매핑

## 1. 객체와 테이블 매핑

### @Entity

* @Entity 가 붙은 클래스는 JPA 가 관리, 엔티티라 한다.&#x20;
* JPA 를 사용해서 테이블과 매핑할 클래스는 @Entity 필수
* **주의**
  * 기본생성자 필수&#x20;
  * 불변객체 사용 X
  * 클래스 변수에 final 사용 금지&#x20;

### @Table&#x20;

## 2. 데이터베이스 스키마 자동 생성&#x20;

* JPA 는 어플리케이션 로딩 시점에, 테이블이 자동 생성 되도록 해준다. \
  \-> 객체 매핑만 해두면 된다.&#x20;
* 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 을 생성한다.&#x20;
* **이렇게 생성된 DDL 은 개발 장비에서만 사용해야 한다!**

#### &#x20;데이터베이스 스키마 자동 생성 - 속성

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 10.13.24.png" alt="" width="563"><figcaption></figcaption></figure>

* **운영 장비에는 절대 create, create-drop, update 사용하면 안된다!!**
  * 개발 초기 단계는 create, update
  * 테스트 서버는 update, validate
  * 스테이징과 운영 서버는 validate 또는 none

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

## 3. 필드와 컬럼 매핑&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 10.31.26.png" alt="" width="563"><figcaption></figcaption></figure>

### @Column

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 10.32.36.png" alt="" width="563"><figcaption></figcaption></figure>

### @Enumerated

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 10.39.23.png" alt="" width="563"><figcaption></figcaption></figure>

* EnumType.ORDINAL : **defalut 타입으로, 이 후 요구사항이 늘어날 때를 생각해서 운영에서는 절대로 사용해서는 안된다!**
* EnumType.STRING

## 4. 기본 키 매핑

* 직접 할당 : @Id만 사용
* 자동 생성 : @GeneratedValue&#x20;
  * IDENTITY : 데이터베이스에 위임, MYSQL
  * SEQUENCE : 데이터베이스 시퀀스 오브젝트 사용
  * TABLE : 키 생성용 테이블 사용, 모든 DB 사용
  * AUTO : 방언에 따라서 자동 기정, default 값
  *

### IDENTITY 전략&#x20;

* 기본 키 생성을 데이터베이스에 위임! -> auto\_increment
* 주로 MySQL, PostgreSQL, SQL Server, DB2 에서 사용
* **auto\_increment 는 데이터베이스에 INSERT SQL 을 실행 한 이후에 ID 값을 알 수 있음 ..**\
  **-> 영속성 컨텍스트에서 관리되려면 PK 를 무조건 가져야 하는데, 쓰기 지연으로 인해서 ID 값을 모를수도 있다 ..**\
  **-> 때문에, 예외적으로! IDENTITY 전략에서만, em.persist() 를 실행하는 시점에 DB 에 쿼리를 날린다.**\
  **-> 쓰기 지연의 장점을 가지지는 못하지만, 성능상 큰 차이가 있지는 않다;**&#x20;

### SEQUENCE 전략&#x20;

* 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(ex, 오라클)
* 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
* **IDENTITY 전략과 마찬가지고 영속성 컨텍스트에서 관리하려면 PK 값(Sequence)을 가져와야 하는데, 마찬가지로 네트워크를 왔다 갔다 하기 때문에, 최적화 문제가 생길 수 있다.** \
  **-> 그래서 @SequenceGenerator allocationSize 의 default 값이 50 인 것을 볼 수 있다!**\
  **-> 이것은 한번의 요청 때 50까지의 Sequence 를 서버 메모리에 다시 가져오고, 모두 사용 후 다시 Sequence 를 요청하는 형식이다.** \
  **-> 최적화라고 할 수는 있지만, 너무 큰 숫자를 설정하면, 서버가 내려갔다 올라갔을 때, 구멍이 생긴다..**&#x20;

### TABLE 전략

* 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략
* 장점 : 모든 데이터베이스에 적용 가능하다.&#x20;
* 단점 : 성능 ;;

### 그럼, 권장하는 식별자 전략은?

* **기본 키 제약 조건 : null 아님, 유일, 변하면 안된다.**\
  \-> 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대체키(UUID)를 사용하자.
* 예를 들어 주민등록번호도 기본 키로 절적하지 않다 ..
* **권장 : Long 형 + 대체키 + 키 생성전략 사용!**
