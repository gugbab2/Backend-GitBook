---
description: 트
---

# EntityManager 와 영속성 컨텍스트

* JPA 는 EntityManager 와 영속성 컨텍스트를 통해서 데이터의 상태 변화를 감지하고 필요한 쿼리를 자동적으로 수행한다.&#x20;
* Application 이 시작될 때 EntityManager 를 자동으로 Bean 에 등록하고, 우리가 알지 못하는 사이에 가져다 사용하고 있다.&#x20;

## 영속성 컨텍스트(Persistence Context)

* ORM 에 사용되는 개념으로 **객체와 데이터베이스간 관리와 상호작용을 담당하는 메모리 영역**이다.\
  영속성 컨텍스트는 객체 지향 프로그래밍의 개념으로, **객체를 데이터베이스에 저장하고, 조회하는 작업을 효율적으로 처리하기 위해 사용**된다. \
  \-> 일반적으로 애플리케이션 실행 중에 유지되며, 객체의 수명 주기를 관리한다. \
  \-> 객체의 변경사항을 추적하고(dirty check), 캐시에 저장함(1차캐시)으로써 데이터베이스와의 효율적인 상호작용을 지원한다.&#x20;

### 영속성 컨텍스트 특징

* Identity Map(식별자 맵) : 영속성 컨텍스트는 객체를 식별자(주로 데이터베이스 기본키) 와 연결하여 식별자를 기반으로 객체를 고유하게 관리한다. \
  \-> 객체를 중복적으로 생성하지 않는다!&#x20;
* 1차 캐시 : 영속성 컨텍스트는 객체를 메모리에 캐싱하여 재사용한다. 객체를 데이터베이스로부터 조회하거나 변경할 때, 영속성 컨텍스트는 먼저 자신의 1차 캐시에서 해당 객체를 검색한다. \
  이미 캐시에 존재하는 경우, 추가적인 데이터베이스 접근 없이 객체를 반환하거나, 변경사항을 처리한다. 이를 통해서 데이터베이스 접근을 최소화하여 성능을 향상시킨다.&#x20;
* Dirty Checking(변경 감지) : 영속성 컨텍스트는 객체의 변경사항을 감지하여 자동으로 데이터베이스에 반영한다. 객체의 상태를 주기적으로 확인하여 변경된 객체만을 데이터베이스에 업데이트하므로, 개발자는 직접 SQL 쿼리를 작성하지 않아도 된다.&#x20;
* 트랜잭션 지원 : 영속성 컨텍스트는 트랙잭션을 지원한다. 트랜잭션 내에서 객체의 변경사항을 추적하고, 트랜잭션이 커밋되는 시점에 변경사항을 데이터베이스에 일괄적으로 반영한다. 이를 통해서 데이터 일관성을 유지하고 롤백 시 변경사항을 취소할 수 있다.&#x20;

### 생명주기

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
* 영속 상태가 되면 몇 가지의 장점을 가지게 된다.&#x20;
  * 1차 캐시&#x20;
    * em.find(key) 를 호출하면 영속성 컨텍스트에 캐시된 데이터를 먼저 찾는다.&#x20;
    * 캐시된 데이터가 없다면 DB 에 접근하여 데이터를 로드하고 1차 캐시 데이터에 저장한다.&#x20;
  * 동일성 보장&#x20;
    * JPA 를 통해서 불러운 데이터는 모두 캐시 데이터에 저장되기 때문에, 같은 id 를 가진 데이터는 같은 데이터이다. \
      \-> 일반적으로 Java 에서 '같다'라는 기준을 Identity(hashcode) / Equals(equals) 이다.&#x20;
  * 트랜잭션 지원하는 쓰기 지연&#x20;
    * 트랜잭션이 시작된 후 JPA 가 생성한 쿼리는 모두 쓰기 지연 저장소(Write-Behind Store)에 저장된다.&#x20;
    * commit 이 수행되면 저장된 모든 쿼리를 실행한다.&#x20;
  * 변경 감지(Dirty Checking)
    * SQL 을 직접 활용하여 개발하면 update 문을 수행할 때 매우 귀찮은 점이 있다!
    * 컬럼 1개, 2개, 3개, ... 개를 모두 수정해야 할 때는 모두 쿼리로 작성해야 한다는 점이다...\
      \-> 이렇게 되면 비지니스 로직이 쿼리에 의존적일 수 밖에 없다.
    * JPA 는 데이터를 저장하기 전에 영속성 컨텍스트에 저장된 데이터가 있는지 확인한다.&#x20;
    * 동일 데이터가 존재하면 update, 없으면 insert 를 수행한다.&#x20;
    * JPA가 실제로 수행하는 쿼리는 모든 컬럼을 변경한다.&#x20;
      * 컬럼이 굉장히 많은(30개이상) 테이블이 아니면 성능에 크게 영향을 끼치지 않는다.&#x20;
      * 엔티티 클래스에 @DynamicUpdate 를 붙여주면, SET 절에 변경된 데이터만 삽입된다.&#x20;
  * 지연로딩
    * 지연로딩을 통해서 엔티티의 연관 관계가 있는 필드에 대해 조회를 처음에는 수행하지 않고!\
      해당 필드에 접근하는 시점(필요한 시점)에서 실제 데이터베이스에서 로딩한다. \
      \-> 필요한 시점에 데이터를 로딩함으로 성능을 향상시킬 수 있다. &#x20;

```java
SomeEntity a = em.find(SomeEntity.class, "1");
SomeEntity b = em.find(SomeEntity.class, "1");
// a == b : true (Identity)
// a.equals(b) : true (Equality)
```

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

### 플러시&#x20;

* 영속성 컨텍스트의 변경 내용을 DB 에 반영하는 절차이다.&#x20;
* 플러시를 수행하면 아래 순서대로 동작한다.&#x20;
  * 데이터의 변경을 감지한다.&#x20;
  * 생성된 쿼리를 쓰기 지연 저장소에 등록한다.&#x20;
  * commit 되면 저장되어 있던 쿼리를 모두 수행한다. \
    \-> em.flush() 를 통해서 직접 플러시 할 수 있다.&#x20;

### 종료

* 영속성 컨텍스트를 종료하려면, EntityManager 의 close 메소드를 호출한다.&#x20;

```java
em.close();
```

### 병합(Merge)

* 준영속 상태의 데이터는 병합 기능을 사용해서 다시 영속 상태로 돌릴 수 있다.&#x20;

```java
SomeEntity entity = em.find(key);
em.detach(entity);
em.merge(entity);    //Merge
```

## 간단한 예제 코드&#x20;

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
