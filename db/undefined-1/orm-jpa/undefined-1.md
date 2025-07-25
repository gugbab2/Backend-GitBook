# 엔티티 매핑

## 1. 객체와 테이블 매핑&#x20;

### @Entity

* `@Entity` 가 붙은 클래스는 JPA 가 관리, 엔티티라 한다.&#x20;
* JPA 를 사용해서 테이블과 매핑할 클래스는 `@Entity` 가 필수&#x20;

#### 주의점&#x20;

* **기본 생성자 필수**(파라미터가 없는 `public`, `protected` 생성자)
* `final` 클래스, `enum`, `interface`, `inner` 클래스 사용X&#x20;
* 저장할 필드에 `final` 사용 X&#x20;

#### 속성 : name&#x20;

* JPA 에서 사용할 엔티티 이름을 지정한다.&#x20;
* 기본값 : 클래스 이름을 그대로 사용&#x20;
* 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.&#x20;

### @Table

* `@Table` 은 엔티티와 매핑할 테이블 지정&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 16.48.02.png" alt="" width="563"><figcaption></figcaption></figure>

## 2. 데이터베이스 스키마 자동 생성&#x20;

* DDL 을 애플리케이션 실행 시점에 자동 생성&#x20;
  * 개발 시점에 굳이 테이블을 만들고 시작하지 않아도 된다.&#x20;
  * JPA 가 귀찮은 테이블 생성을 자동으로 진행해준다.&#x20;
* 테이블 중심 -> 객체 중심&#x20;
* 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 을 생성&#x20;
* **이렇게 생성된 DDL 은 개발 서버에서만 사용해야 한다.**
  * 운영 환경에서는 사용해서는 안된다!&#x20;
* 생성된 DDL 은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

#### 데이터베이스 스키마 자동 생성 - 속성&#x20;

`hibernate.hbm2ddl.auto`

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 16.55.58.png" alt="" width="563"><figcaption></figcaption></figure>

#### 데이터베이스 스키마 자동 생성 - 주의&#x20;

* **운영 장비에는 절대 `create`, `create-drop`, `update` 사용하면 안된다!**
  * **운영 서버에는 약 1억건 가까운 데이터를 가지고 있는 경우도 많다.**&#x20;
  * **운영 장비에서 애플리케이션 로딩 시점에 update 를 한다는 것 자체가 매우 위험성이 높다.**
* 개발 초기 단계는 `create` 또는 `update`&#x20;
* 테스트 서버는 `update` 또는 `validate`&#x20;
* 스테이징과 운영 서버는 `validate` 또는 `none`&#x20;

#### DDL 생성 기능&#x20;

* 제약조건 추가 : 회원 이름 필수, 10자 초과X&#x20;
  * `@Column(nullable = false, length = 10)`
* 유니크 제약 조건 추가
  * `@Table(uniqueConstraints = {@UniqueConstraint( name = "NAME_AGE_UNIQUE", columnNames = {"NAME", "AGE"} )})`
* DDL 생성 기능은 DDL 을 자동 생성할 때만 사용되고 JPA 의 실행 로직에는 영향을 주지 않는다.&#x20;

## 3. 필드와 컬럼 매핑&#x20;

### 요구사항 추가&#x20;

1. 회원은 일반 회원과 관리자로 구분해야 한다.&#x20;
2. 회원 가입일과 수정일이 있어야 한다.&#x20;
3. 회원은 설명할 수 있는 필드가 있어야 한다. 이 필드는 길이 제한이 없다.&#x20;

```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;

    public Member() {
    }
}
```

#### 매핑 어노테이션 정리&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 17.15.42.png" alt="" width="563"><figcaption></figcaption></figure>

### @Column

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 17.16.35.png" alt=""><figcaption></figcaption></figure>

### @Enumerated&#x20;

* 자바 enum 타입을 매핑할 때 사용&#x20;
* **주의! ORDINAL 사용 X**&#x20;
  * **enum 타입을 순서(숫자) 로 저장하게 되면 이후 enum 타입을 추가하거나 삭제했을 때 정합성 문제가 발생!**
  * **운영에서는 정말 큰 문제다!**

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 17.21.00.png" alt=""><figcaption></figcaption></figure>

### @Temporal&#x20;

* 날짜 타입(`java.util.Date`, `java.util.Calendar`) 을 매핑할 때 사용&#x20;
* 참고 : `LocalDate`, `LocalDateTime` 을 사용할 때는 생략 가능&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 17.24.58.png" alt=""><figcaption></figcaption></figure>

### @Lob&#x20;

데이터베이스 BLOB, CLOB 타입과 매핑&#x20;

* `@Lob` 에는 지정할 수 있는 속성이 없다.&#x20;
* 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑&#x20;
  * CLOB: String, char\[], java.sql.CLOB
  * BLOB: byte\[], java.sql. BLOB

### @Transient

* 필드 매핑 X&#x20;
* 데이터베이스에 저장 X, 조회 X&#x20;
* 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용&#x20;

```java
@Transient
private Integer temp;
```

## 4. 기본키 매핑

#### 기본기 매핑 어노테이션&#x20;

* `@Id`&#x20;
* `@GeneratedValue`

```java
@Id @GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

#### 기본 키 매핑 방법&#x20;

* 직접 할당 : `@Id` 만 사용&#x20;
* 자동 생성 (`@GeneratedValue`)&#x20;
  * IDENTITY : 데이터베이스에 위임, MySQL&#x20;
  * SEQUENCE : 데이터베이스 시퀸스 오브젝트 사용, ORACLE
    * `@SequenceGenerator` 필요&#x20;
  * TABLE : 키 생성용 테이블 사용, 모든 DB 에서 사용&#x20;
    * `@TableGenerator` 필요&#x20;
  * AUTO : 방언에 따라 자동 지정 : 기본값

### IDENTITY 전략 &#x20;

* 기본 키 생성을 데이터베이스에 위임&#x20;
*   주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용

    (예: MySQL의 AUTO\_ INCREMENT)
* JPA 는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행&#x20;
* **AUTO\_INCREMENT 는 데이터베이스에 INSERT SQL 을 실행 한 이후에 ID 값을 알 수 있음**&#x20;
* **때문에, IDENTITY 전략은 `em.persist()` 시점에 즉시 INSERT SQL 실행하고 DB 에서 식별자를 조회**&#x20;

### SEQUENCE 전략&#x20;

* 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트 (예 : 오라클 시퀀스)&#x20;
* 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용
* `em.persist()` 실행 시 데이터베이스 시퀀스를 조회한다. (INSERT SQL 을 실행하지는 않는다)

#### SEQUENCE 전략 - 매핑&#x20;

```java
@Entity
@SequenceGenerator(name = "member_seq_generator", sequenceName = "member_seq")
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "member_seq_generator")
    private Long id;
```

#### SEQUENCE - @SequenceGenerator

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 18.04.09.png" alt=""><figcaption></figcaption></figure>

> 참고 - 데이터베이스와 IO 가 너무 많은 것 아닌가?&#x20;
>
> * `allocationSize` 속성 만큼 한번에 시퀀스 값을 메모리에 올려놓고 사용한다.&#x20;
> * 때문에, 애플리케이션에서 메모리에 올려둔 시퀀스를 다 사용하기 이전까지 IO 를 하지 않는다.&#x20;
> * **동시성 문제가 발생할 수 있을 것이라 생각하지만, 각 요청마다 새로운 시퀀스를 사용할 것이기 때문에, 문제는 발생하지 않는다.**
>   * 1번 요청 -> 시퀀스 1 \~ 50 확보&#x20;
>   * 2번 요청 -> 시퀀스 51 \~ 100 확보

### TABLE 전략&#x20;

* 키 생성 전용 테이블을 하나 만들어서 데이터베이스 시퀀스를 흉내내는 전략&#x20;
* 장점 : 모든 데이터베이스에 적용 가능&#x20;
* 단점 : 성능&#x20;

#### TABLE 전략 - 매핑

```sql
create table MY_SEQUENCES (
    sequence_name varchar(255) not null,
    next_val bigint,
    primary key ( sequence_name )
)
```

```java
@Entity
@TableGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        table = "MY_SEQUENCES",
        pkColumnName = "MEMBER_SEQ", allocationSize = 1
)
public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
            generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

#### @TableGenerator - 속성

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 18.09.36.png" alt=""><figcaption></figcaption></figure>

### 권장하는 식별자 전략&#x20;

* 기본 키 제약 조건 : null 아님, 유일, **변하면 안된다.**
* **미래까지 이 조건을 만족하는 자연키는 찾기 어렵다..** \
  **대리키(대체키)를 사용하자!**&#x20;
* 예를 들어, 주민등록번호도 기본 키로 적절하지 않다..&#x20;
* **권장 : Long 형 + 대체키 + 키 생성전략 사용**
