---
description: 트
---

# 영속성 컨텍스트



<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 07.50.41.png" alt="" width="563"><figcaption></figcaption></figure>

* EntityManagerFactory 는 어플리케이션 단위로 생성이 되고, EntityManager 는 요청 단위로 생성된다. &#x20;
* EntityManager 를 통해서 영속성 컨텍스트에 접근한다.&#x20;
* Application 이 시작될 때 EntityManagerFactory, EntityManager 를 자동으로 Bean 에 등록하고, 우리가 알지 못하는 사이에 가져다 사용하고 있다.&#x20;
* JPA 는 EntityManager 와 영속성 컨텍스트를 통해서 데이터의 상태 변화를 감지하고 필요한 쿼리를 자동적으로 수행한다.&#x20;

## 1. 영속성 컨텍스트(Persistence Context)

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 07.54.52.png" alt="" width="563"><figcaption></figcaption></figure>

* "엔티티를 영구 저장하는 환경" 이라는 뜻!\
  \-> EntityManager.persist(Member); \
  \-> 영구 저장이라는 의미는 디비 저장의 의미가 아닌 영속성 컨텍스트에 저장한다는 것을 의미한다.&#x20;
* ORM 에 사용되는 개념으로 **객체와 데이터베이스간 관리와 상호작용을 담당하는 메모리 영역**이다.\
  영속성 컨텍스트는 객체 지향 프로그래밍의 개념으로, **객체를 데이터베이스에 저장하고, 조회하는 작업을 효율적으로 처리하기 위해 사용**된다. \
  \-> 일반적으로 애플리케이션 실행 중에 유지되며, 객체의 수명 주기를 관리한다. \
  \-> 객체의 변경사항을 추적하고(dirty check), 캐시에 저장함(1차캐시)으로써 데이터베이스와의 효율적인 상호작용을 지원한다.&#x20;

### 1-1. 영속성 컨텍스트 특징

* 1차 캐시&#x20;
  * 영속성 컨텍스트는 객체를 메모리에 캐싱하여 재사용한다. 객체를 데이터베이스로부터 조회하거나 변경할 때, 영속성 컨텍스트는 먼저 자신의 1차 캐시에서 해당 객체를 검색한다. \
    이미 캐시에 존재하는 경우, 추가적인 데이터베이스 접근 없이 객체를 반환하거나, 변경사항을 처리한다. \
    **-> 이를 통해서 데이터베이스 접근을 최소화하여 성능을 향상시킨다.**\
    **-> 하지만, 고객의 요청이 들어와 응답이 나가면 영속성 컨택스트를 지워버린다.. 1차 캐시는 쓰레드 내에서만 공유하기 때문에, 큰 성능 이점을 얻을수는 없다..**&#x20;
* 동일성(Identity) 보장&#x20;
  * 영속성 컨텍스트는 객체를 식별자(주로 데이터베이스 기본키) 와 연결하여 식별자를 기반으로 객체를 고유하게 관리한다. \
    **-> 객체를 중복적으로 생성하지 않는다!** \
    **-> == 비교를 사용할 수 있다.**
*   트랜잭션을 지원하는 쓰기 지연**(입력, 수정, 삭제 쿼리 해당)**



    <figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 08.17.11.png" alt="" width="375"><figcaption></figcaption></figure>

    * **트랜잭션이 시작된 후 JPA 가 생성한 쿼리는 모두 쓰기 지연 저장소(Write-Behind Store)에 저장된다.** \
      **-> 1차 캐시에 저장되는 동시에 쓰기 지연 저장소에 저장된다고 생각하면 된다.**
    * **commit 이 수행되면 저장된 모든 쿼리를 실행한다.**\
      **-> 이전 까지는 디비가 아닌 영속성 컨텍스트에 저장된 상태이다.**&#x20;
* Dirty Checking(변경 감지)
  * 영속성 컨택스트는 Commit 을 하게되면, 1차 캐시에서 Entity, 스냅샷을 비교한다. 해당 시점에 Entity, 스냅샷이 다를 때, 해당 쿼리(Update, Delete)를 쓰기 지연 저장소에 저장을 하게되고 해당 쿼리를 실행하게 된다. &#x20;
  * **JPA 의 사상 자체가 객체지향적인 비지니스 코드를 만드는 것이 목적이기에, 자바 Collection 처럼 동작하도록 되어있다.** \
    **-> 객체의 상태를 변경하면 그것이 전부이다.**&#x20;

```java
tx.begin();

try{
    Member findMember = em.find(Member.class, 6L);
    findMember.setName("zzzzz");

    System.out.println("=================");

    tx.commit();
}catch (Exception e){
    tx.rollback();
}finally {
    em.close();
}
```

* 지연 로딩(Lazy Loading)&#x20;
  *

### 1-2. 생명주기

#### 비영속

* @Entity 어노테이션을 갖는 엔티티 인스턴스를 막 생성 했을 때, 영속성 컨텍스트에서 관리하지 않는다. \
  \-> EntityManager 의 persist 메소드를 사용하여 영속 상태로 바꿀 수 있다.&#x20;

```java
em.persist(someEntity);
```

#### 영속

* EntityManager 를 통해서 데이터를 영속성 컨텍스트에 저장했다. \
  \-> **JPA 는 일반적으로 id 필드가 존재하지 않으면 예외를 뱉어내는데, 영속 상태의 엔티티를 관리하기 위함이다.**\
  \-> id 로 데이터를 관리하기 때문에 꼭 필요한 것이다.&#x20;

#### 준영속

* 원래 영속 상태였으나, 영속성 컨텍스트에서 분리되어 더 이상 관리하지 않는 데이터 상태이다.&#x20;
* 영속 상태의 엔티티를 detach 시키거나, 영속성 컨텍스트 자체가 초기화, 종료되면 내부의 모든 데이터는 준영속 상태가 된다. \
  \-> 관리되지는 않는 상태이지만 JPA의 지원을 받지 못할 뿐, 정상적인 데이터를 갖는 인스턴스이다.

```java
// 1.
em.detach(someEntity);
// 2.
em.close();
// 3.
em.clear();
```

#### 삭제

* 엔티티 영속성 컨텍스트와 DB 양쪽에서 모두 삭제한다.&#x20;

```java
em.remove(someEntity);
```

### 1-3. 플러시&#x20;

* 변경감지(Dirty Checking)
* 1차 캐시의 Entity 와 스냅샷을 비교 후 변경 사항이 확인되었을 때, 쓰기 지연 저장소에 해당 쿼리를 저장한다. \
  (등록, 수정, 삭제 쿼리에 해당)
* 플러시하는 방법&#x20;
  * em.flush();
  * 커밋 시 자동호출
  * JPQL 실행 시 자동호출
* 정리&#x20;
  * 플러시가 영속성 컨텍스트를 비우지는 않는다.&#x20;
  * 영속성 컨텍스트의 변경내용을 데이터베이스에 동기화 하는 것이다.\
    \-> 플러시는 쓰기 지연 저장소에 있는 쿼리들을 저장시키는 것을 생각하면 된다.&#x20;
  * 트랜잭션이라는 작업 단위가 중요하다. \
    \-> 커밋 직전에만 동기화 하면 된다.&#x20;

### 종료

* 영속성 컨텍스트를 종료하려면, EntityManager 의 close 메소드를 호출한다.&#x20;

```java
em.close();
```

### 1-4. 병합(Merge)

* 준영속 상태의 데이터는 병합 기능을 사용해서 다시 영속 상태로 돌릴 수 있다.&#x20;

```java
SomeEntity entity = em.find(key);
em.detach(entity);
em.merge(entity);    //Merge
```

## 2. 간단한 예제 코드&#x20;

```java
@Entity
@Getter
@Setter
public class Member {

    @Id @GeneratedValue
    private long id;
    private String username;
}

@Repository
public class MemberRepository {

    @PersistenceContext
    private EntityManager em;

    public Long save(Member member) {
        em.persist(member);
        return member.getId();
    }

    public Member find(Long id){
        return em.find(Member.class, id);
    }
}

@RunWith(SpringRunner.class)
@SpringBootTest
public class MemberRepositoryTest {

    @Autowired
    MemberRepository memberRepository;

    @Test
    @Transactional //테스트케이스에서 Transactional 어노테이션은 기본적으로 롤백한다.
    @Rollback(false)
    public void testMember() throws Exception{

        //given
        Member member = new Member();
        member.setUsername("memberA");

        //when
        Long saveId = memberRepository.save(member);
        Member findMember = memberRepository.find(saveId);

        //then
        Assertions.assertThat(member.getId()).isEqualTo(findMember.getId());
        Assertions.assertThat(member.getUsername()).isEqualTo(findMember.getUsername());
        Assertions.assertThat(findMember).isEqualTo(member);    //현재 member 엔티티는 equals, hashcode 메서드를 오버로딩 하지 않았기 때문에, 같지 않아야 한다.
                                                                //하지만 JPA 에서는 같은 데이터로 인식한다.(1차캐시)
    }
}
```
