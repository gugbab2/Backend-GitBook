# 영속성 관리 - 내부 동작 방식

## 1. 영속성 컨텍스트

#### JPA 에서 가장 중요한 2가지&#x20;

* 객체와 관계형 데이터베이스 매핑하기
* **영속성 컨텍스트**&#x20;

#### 엔티티 매니저 팩토리와 엔티티 매니저&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.09.43.png" alt="" width="563"><figcaption></figcaption></figure>

* 엔티티 매니저 팩토리는 요청이 올 때마다 엔티티 매니저를 생성하게 되고,
* 엔티티 매니저는 커넥션 풀을 사용하여 DB 를 사용하게 된다.&#x20;

### 영속성 컨텍스트?&#x20;

* JPA 를 이해하는데 가장 중요한 용어&#x20;
* "엔티티를 영구 저장하는 환경" 이라는 뜻&#x20;
* `EntityManager.persist(entity);`

#### 엔티티 메니저? 영속성 컨텍스트?&#x20;

* 영속성 컨텍스트는 논리적인 개념
* 눈에 보이지 않는다.&#x20;
* 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.&#x20;

### 엔티티의 생명주기&#x20;

* 비영속 (new/transient)
  * 영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태&#x20;
* 영속 (managed)
  * 영속성 컨텍스트에 **관리**되는 상태&#x20;
* 준영속 (detached)&#x20;
  * 영속성 컨텍스트에 저장되었다가 **분리**된 상태&#x20;
* 삭제 (removed)&#x20;
  * **삭제**된 상태&#x20;

#### 비영속&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.15.28.png" alt="" width="563"><figcaption></figcaption></figure>

```java
// 객체를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername("회원1"); 
```

* 영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태&#x20;
* 엔티티 객체를 생성하기만 한 상태&#x20;

#### 영속&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.16.56.png" alt="" width="479"><figcaption></figcaption></figure>

```java
// 객체를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1"); 
member.setUsername("회원1"); 

EntityManager em = emf.createEntityManager(); 
em.getTransaction().begin(); 

// 객체를 저장한 상태(영속) 
em.persist(member); 
```

* 영속성 컨텍스트에 **관리**되는 상태&#x20;
* 엔티티 매니저에 엔티티를 전달한 상태&#x20;

#### 준영속, 삭제&#x20;

```java
// 회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태 
em.detach(member); 

// 객체를 삭제한 상태(삭제) 
em.remove(member); 
```

### 영속성 컨텍스트의 이점&#x20;

* **1차 캐시**
* **동일성(identity) 보장**&#x20;
* **트랜잭션을 지원하는 쓰기 지연(transactional write-behind)**&#x20;
* **변경 감지(Dirty Checking)**&#x20;

#### 1차 캐시  (엔티티 조회)

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.27.48.png" alt="" width="563"><figcaption></figcaption></figure>

```java
// 엔티티를 생성한 상태(비영속) 
Member member = new Member(); 
member.setId("member1");    
member.setUserName("회원1");     

// 엔티티를 영속 && 1차 캐시에 저장됨 
em.persist(member); 

// 1차 캐시에서 조회 
Member findMember = em.find(Member.class, "member1"); 

// 데이터베이스에서 조회 
Member findMember2 = em.find(Member.class, "member2"); 
```

* JPA 는 기본적으로 1차 캐시에 저장하기 때문에, 1차 캐시에 데이터가 있다면 DB 에 `SELETE` 쿼리를 날리지 않는다.&#x20;
* 기본적으로는 1차 캐시에서 조회하고, 1차 캐시에 데이터가 없을때 DB 에 `SELECT` 쿼리를 날린다.&#x20;
  * DB 에 `SELECT` 쿼리를 날린 후, 쿼리 결과를 1차 캐시에 저장한다.&#x20;
* 조회시 이점을 얻을 수 있다. \
  (사실 고객의 요청이 끝나면 영속성 컨텍스트를 지우기 때문에, 큰 도움은 안된다..)

#### 영속 엔티티의 동일성 보장

```java
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member2");

System.out.println(a == b); // 동일성 비교 true
```

* 1차 캐시로 반복 가능한 읽기(Repeatable Read) 등급의 트랜잭션 격리 수준을 데이터베이스가 아닌 애플리케이션 차원에서 제공한다.&#x20;

#### 트랜잭션을 지원하는 쓰기 지연 (엔티티 등록)

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.47.27.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 15.49.01.png" alt=""><figcaption></figcaption></figure>

```java
EntityManager em = emf.createEntityManager(); 
EntityTransaction transaction = em.getTransaction(); 

// 엔티티 매니저는 데이터 변경시 트랜잭션을 시작해야 한다. 
transaction.begin(); 

em.persist(memberA);
em.persist(memberB);
// 여기까지 INSERT 쿼리를 보내지 않는다. 

// 커밋하는 순간 데이터베이스에 INSERT 쿼리를 보낸다. 
transaction.commit(); 
```

* 커밋 시점에 한번에 쿼리를 보냄으로써 쿼리를 쓰는 과정을 지연할 수 있다.

```xml
<property name="hibernate.jdbc.batch_size" value="10"/>
```

* `hibernate.jdbc.batch_size` 설정을 사용하면 `flush` 시 IO 단위를 설정할 수 있다.&#x20;
* 설정하지 않는다면 기본값인 1이 설정된다.&#x20;
* **작은 설정으로 큰 효과를 낼 수 있다는 건 매우 큰 장점이다!**&#x20;

#### 변경 감지(Dirty Checking)

<figure><img src="../../../.gitbook/assets/스크린샷 2025-07-01 16.00.57.png" alt=""><figcaption></figcaption></figure>

```java
EntityManager em = emf.createEntityManager(); 
EntityTrasaction transaction = em.getTransaction(); 
transaction.begin(); 

// 영속 엔티티 조회 
Member memberA = em.find(Member.class, "memberA"); 

// 영속 엔티티 데이터 수정 
memberA.setUsername("hi"); 
memberA.setAge(10); 

// em.update(member) 이런 코드가 있어야 할 것 같지만, 필요없다!

transaction.commit();  
```

* 커밋 시점에 내부적으로 `flush()` 가 발생한다.&#x20;
* `flush()` 시점에 엔티티와 스냅샷을 비교하게 된다.&#x20;
* **이때, 엔티티와 스냅샷 값이 같지 않다면 수정 쿼리를 날리게 된다.**&#x20;
* **JPA 의 목적이 자바 컬렉션 처럼 DB 를 사용하는 것이다.**&#x20;
  * 생각해보면, 우리는 자바에서 컬렉션 내부 데이터를 변경하고 따로 update 를 하지 않는다.&#x20;
  * 그냥 내부 데이터를 변경하는 것으로 끝인 것이다.
  * **JPA 또한 동일하게 동작하게 하는 것이다.**&#x20;

## 2. 플러시&#x20;

영속성 컨텍스트의 변경내용을 데이터베이스에 반영하는 작업&#x20;

#### 플러시 발생하면 다음과 같은 동작이 발생한다.

1. 변경 감지&#x20;
   1. 엔티티와 스냅샷을 비교 후, 엔티티가 스탭샷이 같지 않다면 수정 쿼리를 만든다.&#x20;
2. 수정된 엔티티 쓰기 지연 SQL 저장소에 등록&#x20;
3. 쓰기 지연 SQL 저장소의 쿼리를 데이터베이스에 전송 (등록, 수정, 삭제 쿼리)&#x20;

#### 영속성 컨텍스트를 플러시 하는 방법

* `em.flush()` - 직접 호출
* 트랜잭션 커밋 - 자동 호출&#x20;
* JPQL 쿼리 실행 - 자동 호출

> 참고&#x20;
>
> * 플러시는 영속성 컨텍스트를 비우지 않음
>   * 1차 캐시는 유지하지만,&#x20;
>   * 쓰기 지연 SQL 저장소는 비운다.&#x20;
> * 플러시는 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화
> * 트랜잭션이라는 작업 단위가 중요 -> 커밋 직전에만 동기화하면 됨

#### JPQL 쿼리 실행시 플러시가 자동으로 호출되는 이유&#x20;

```java
em.persist(memberA); 
em.persist(memberB); 
em.persist(memberC); 

// 중간에 JPQL 실행 
query = em.createQuery("select m from Member m", Member.class); 
List<Member> members = query.getResultList(); 
```

* JPQL 은 코드 실행과 동시에 쿼리가 실행된다.&#x20;
* 만약 위 코드에서 JPQL 실행 시점에 플러시 되지 않는다면, 조회할 데이터가 없다..&#x20;
* 때문에, JPQL 은 코드 실행과 동시에 플러시가 실행 된다.&#x20;

#### 플러시 모드 옵션&#x20;

```java
em.setFlushMode(FlushModeType.COMMIT)
```

* `FlushModeType.Auto`
  * 커밋이나 쿼리를 실행할 때 플러시 (기본값)
* `FlushModeType.COMMIT`
  * 커밋할 때만 플러시&#x20;

## 3. 준영속 상태&#x20;

* 영속(1차 캐시에 올라간 상태) -> 준영속&#x20;
* 영속 상태의 엔티티가 영속 컨텍스트에서 분리 (detached)&#x20;
* 영속성 컨텍스트가 제공하는 기능을 사용 못함
  * 쓰기 지연, 변경 감지 등등..

#### 준영속 상태로 만드는 방법

* `em.detach(entity)`&#x20;
  * 특정 엔티티만 준영속 상태로 전환&#x20;
* `em.clear()`
  * 영속성 컨텍스트를 완전히 초기화
* `em.close()`
  * 영속성 컨텍스트를 종료
