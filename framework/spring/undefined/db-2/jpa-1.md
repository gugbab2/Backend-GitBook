# 데이터 접근 기술 - 스프링 데이터 JPA

## 1. 스프링 데이터 JPA 주요 기능&#x20;

스프링 데이터 JPA 는 JPA 를 편리하게 사용할 수 있도록 도와주는 라이브러리이다.&#x20;

수많은 편리한 기능을 제공하지만 가장 대표적인 기능은 다음과 같다.&#x20;

* 공통 인터페이스 기능&#x20;
* 쿼리 메서드 기능&#x20;

### 공통 인터페이스 기능&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-25 15.41.29.png" alt="" width="563"><figcaption></figcaption></figure>

* `JpaRepository` 인터페이스를 통해서 기본적인 CRUD 기능 제공한다.
* 공통화 가능한 기능이 거의 모두 포함되어 있다.

#### JpaRepository 사용법

```java
public interface ItemRepository extends JpaRepository<Item, Long> {
}
```

* `JpaRepository` 인터페이스를 인터페이스 상속 받고, 제네릭에 관리할 `<엔티티, 엔티티ID>` 를 주면 된다.
* 그러면 `JpaRepository` 가 제공하는 기본 CRUD 기능을 모두 사용할 수 있다.

#### 스프링 데이터 JPA가 구현 클래스를 대신 생성

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-25 15.42.48.png" alt="" width="563"><figcaption></figcaption></figure>

* `JpaRepository` 인터페이스만 상속받으면 스프링 데이터 JPA가 프록시 기술을 사용해서 구현 클래스를 만들어준다. 그리고 만든 구현 클래스의 인스턴스를 만들어서 스프링 빈으로 등록한다.
* 따라서 개발자는 구현 클래스 없이 인터페이스만 만들면 기본 CRUD 기능을 사용할 수 있다.

### 쿼리 메서드 기능&#x20;

스프링 데이터 JPA는 인터페이스에 메서드만 적어두면, 메서드 이름을 분석해서 쿼리를 자동으로 만들고 실행해주는 기능을 제공한다.

#### 순수 JPA 리포지토리&#x20;

<pre class="language-java"><code class="lang-java">public List&#x3C;Member> findByUsernameAndAgeGreaterThan(String username, int age) {
<strong>    return em.createQuery("select m from Member m where m.username = :username 
</strong><strong>and m.age > :age")
</strong>    .setParameter("username", username)
    .setParameter("age", age)
    .getResultList();
}
</code></pre>

* 순수 JPA를 사용하면 직접 JPQL을 작성하고, 파라미터도 직접 바인딩 해야 한다.

#### 스프링 데이터 JPA&#x20;

```java
public interface MemberRepository extends JpaRepository<Member, Long> {
    List<Member> findByUsernameAndAgeGreaterThan(String username, int age);
}
```

* 스프링 데이터 JPA는 메서드 이름을 분석해서 필요한 JPQL을 만들고 실행해준다. 물론 JPQL은 JPA가 SQL로 번역해서 실행한다.
* 물론 그냥 아무 이름이나 사용하는 것은 아니고 다음과 같은 규칙을 따라야 한다.

> #### 쿼리 메소드 필터 조건
>
> 스프링 데이터 JPA 공식 문서 참고
>
> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)
>
> [https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-\
> result](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repositories.limit-query-result)

#### **JPQL 직접 사용하기**

```java
public interface SpringDataJpaItemRepository extends JpaRepository<Item, Long> {
    
    //쿼리 메서드 기능
    List<Item> findByItemNameLike(String itemName);
    
    //쿼리 직접 실행
    @Query("select i from Item i where i.itemName like :itemName and i.price <= :price")
    List<Item> findItems(@Param("itemName") String itemName, 
                         @Param("price") Integer price);
}
```

* 쿼리 메서드 기능 대신에 직접 JPQL을 사용하고 싶을 때는 `@Query` 와 함께 JPQL을 작성하면 된다. 이때는 메서드 이름으로 실행하는 규칙은 무시된다.
* 참고로 스프링 데이터 JPA는 JPQL 뿐만 아니라 JPA의 네이티브 쿼리 기능도 지원하는데, JPQL 대신에 SQL을 직접 작성할 수 있다.&#x20;

## 2. 스프링 데이터 JPA 적용1,2&#x20;

패스..&#x20;
