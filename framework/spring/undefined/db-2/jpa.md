# 데이터 접근 기술 - JPA

## 1. JPA 시작&#x20;

스프링과 JPA 는 자바 엔터프라이즈(기업) 시장의 주력 기술이다.

스프링이 DI 컨테이너를 포함한 애플리케이션 전바의 다양한 기능을 제공한다면, JPA 는 ORM 데이터 접근 기술을 제공한다.&#x20;

스프링 + 데이터 접근기술의 조합을 구글 트랜드로 비교했을 때

* 글로벌에서는 스프링+JPA 조합을 80%이상 사용한다.
* 국내에서도 스프링 + JPA 조합을 50%정도 사용하고, 2015년 부터 점점 그 추세가 증가하고 있다.

JPA는 스프링 만큼이나 방대하고, 학습해야할 분량도 많다. 하지만 한번 배워두면 데이터 접근 기술에서 매우 큰 생산성 향상을 얻을 수 있다. 대표적으로 JdbcTemplate이나 MyBatis 같은 SQL 매퍼 기술은 SQL을 개발자가 직접 작성해야 하지만, JPA를 사용하면 SQL도 JPA가 대신 작성하고 처리해준다.

## 2. JPA 설정

`spring-boot-starter-data-jpa` 라이브러리를 사용하면 JPA와 스프링 데이터 JPA를 스프링 부트와 통합하고, 설정도 아주 간단히 할 수 있다.

`build.gradle`&#x20;

```gradle
//JPA, 스프링 데이터 JPA 추가
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
```

`main - application.properties`

```properties
#JPA log
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

`test - application.properties`

```properties
#JPA log
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

* `org.hibernate.SQL=DEBUG` : 하이버네이트가 생성하고 실행하는 SQL을 확인할 수 있다.
* `org.hibernate.type.descriptor.sql.BasicBinder=TRACE` : SQL에 바인딩 되는 파라미터를 확인할 수 있다.

## 3. JPA 적용1 - 개발&#x20;

#### **Item - ORM 매핑**

```java
package hello.itemservice.domain;

import lombok.Data;

import javax.persistence.*;

@Data
@Entity
public class Item {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "item_name", length = 10)
    private String itemName;
    private Integer price;
    private Integer quantity;

    public Item() {
    }

    public Item(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

* `@Entity` : JPA가 사용하는 객체라는 뜻이다. 이 에노테이션이 있어야 JPA가 인식할 수 있다. 이렇게 `@Entity` 가 붙은 객체를 JPA에서는 엔티티라 한다.
* `@Id` : 테이블의 PK와 해당 필드를 매핑한다.
* `@GeneratedValue(strategy = GenerationType.IDENTITY)` : PK 생성 값을 데이터베이스에서 생성하는 `IDENTITY` 방식을 사용한다. 예) MySQL auto increment
* `@Column` : 객체의 필드를 테이블의 컬럼과 매핑한다.
  * `name = "item_name"` : 객체는 `itemName` 이지만 테이블의 컬럼은 `item_name` 이므로 이렇게 매핑했다.
  * `length = 10` : JPA의 매핑 정보로 DDL(`create table` )도 생성할 수 있는데, 그때 컬럼의 길이 값으로 활용된다. (`varchar 10`)
  * `@Column` 을 생략할 경우 필드의 이름을 테이블 컬럼 이름으로 사용한다.&#x20;
    * **참고로 지금처럼 스프링 부트와 통합해서 사용하면 필드 이름을 테이블 컬럼 명으로 변경할 때 객체 필드의 카멜 케이스를 테이블 컬럼의 언더스코어로 자동으로 변환해준다.**
    * `itemName` -> `item_name`, 따라서 위 예제의 `@Column(name = "item_name")` 를 생략해도 된다.

JPA는 `public` 또는 `protected` 의 기본 생성자가 필수이다. 기본 생성자를 꼭 넣어주자.

```java
public Item() {
}
```

#### **JpaItemRepositoryV1**

```java
package hello.itemservice.repository.jpa;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.util.StringUtils;

import javax.persistence.EntityManager;
import javax.persistence.TypedQuery;
import java.util.List;
import java.util.Optional;

@Slf4j
@Repository
@Transactional
public class JpaItemRepositoryV1 implements ItemRepository {

    private final EntityManager em;

    public JpaItemRepositoryV1(EntityManager em) {
        this.em = em;
    }

    @Override
    public Item save(Item item) {
        em.persist(item);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = em.find(Item.class, itemId);
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        Item item = em.find(Item.class, id);
        return Optional.ofNullable(item);
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String jpql = "select i from Item i";
        Integer maxPrice = cond.getMaxPrice();
        String itemName = cond.getItemName();
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            jpql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            jpql += " i.itemName like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                jpql += " and";
            }
            jpql += " i.price <= :maxPrice";
        }
        log.info("jpql={}", jpql);
        TypedQuery<Item> query = em.createQuery(jpql, Item.class);
        if (StringUtils.hasText(itemName)) {
            query.setParameter("itemName", itemName);
        }
        if (maxPrice != null) {
            query.setParameter("maxPrice", maxPrice);
        }
        return query.getResultList();
    }
}
```

* `private final EntityManager em` : 생성자를 보면 스프링을 통해 엔티티 매니저(`EntityManager`) 라는 \
  것을 주입받은 것을 확인할 수 있다. JPA의 모든 동작은 엔티티 매니저를 통해서 이루어진다. 엔티티 매니저는 내부에 데이터소스를 가지고 있고, 데이터베이스에 접근할 수 있다.
* `@Transactional` : JPA의 모든 데이터 변경(등록, 수정, 삭제)은 트랜잭션 안에서 이루어져야 한다. 조회는 트랜잭션이 없어도 가능하다. 변경의 경우 일반적으로 서비스 계층에서 트랜잭션을 시작하기 때문에 문제가 없다.&#x20;
  * 하지만 이번 예제에서는 복잡한 비즈니스 로직이 없어서 서비스 계층에서 트랜잭션을 걸지 않았다. JPA에서는 데이터 변경시 트랜잭션이 필수다. 따라서 리포지토리에 트랜잭션을 걸어주었다.&#x20;
  * **다시 한번 강조하지만 일반적으로는 비즈니스 로직을 시작하는 서비스 계층에 트랜잭션을 걸어주는 것이 맞다.**

#### JpaConfig

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.jpa.JpaItemRepositoryV1;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.persistence.EntityManager;

@Configuration
@RequiredArgsConstructor
public class JpaConfig {

    private final EntityManager em;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JpaItemRepositoryV1(em);
    }

}
```

#### **ItemServiceApplication - 변경**

```java
package hello.itemservice;

import hello.itemservice.config.*;
import hello.itemservice.repository.ItemRepository;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Profile;

@Slf4j
//@Import(MemoryConfig.class)
//@Import(JdbcTemplateConfigV1.class)
//@Import(JdbcTemplateConfigV2.class)
//@Import(JdbcTemplateConfigV3.class)
//@Import(MybatisConfig.class)
@Import(JpaConfig.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
public class ItemServiceApplication {

    public static void main(String[] args) {
       SpringApplication.run(ItemServiceApplication.class, args);
    }

    @Bean
    @Profile("local")
    public TestDataInit testDataInit(ItemRepository itemRepository) {
       return new TestDataInit(itemRepository);
    }
}
```

## 4. JPA 적용2 - 리포지토리 분석

## 5. JPA 적용3 - 예외 변환&#x20;

JPA의 경우 예외가 발생하면 JPA 예외가 발생하게 된다.

```java
@Slf4j
@Repository
@Transactional
public class JpaItemRepositoryV1 implements ItemRepository {

    private final EntityManager em;

    public JpaItemRepositoryV1(EntityManager em) {
        this.em = em;
    }

    @Override
    public Item save(Item item) {
        em.persist(item);
        return item;
    }
}
```

*   `EntityManager` 는 순수한 JPA 기술이고, 스프링과는 관계가 없다. 따라서 엔티티 매니저는 예외가 발생하면

    JPA 관련 예외를 발생시킨다.
* JPA는 `PersistenceException` 과 그 하위 예외를 발생시킨다.
  * 추가로 JPA는 `IllegalStateException`, `IllegalArgumentException` 을 발생시킬 수 있다.
* 그렇다면 JPA 예외를 스프링 예외 추상화(`DataAccessException`)로 어떻게 변환할 수 있을까?
* 비밀은 바로 `@Repository` 에 있다.

#### 예외 변환 전&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-25 14.37.16.png" alt=""><figcaption></figcaption></figure>

**@Repository의 기능**

* `@Repository` 가 붙은 클래스는 컴포넌트 스캔의 대상이 된다.
* `@Repository` 가 붙은 클래스는 예외 변환 AOP의 적용 대상이 된다.
  * 스프링과 JPA를 함께 사용하는 경우 스프링은 JPA 예외 변환기(`PersistenceExceptionTranslator`)를 \
    등록한다.
  * 예외 변환 AOP 프록시는 JPA 관련 예외가 발생하면 JPA 예외 변환기를 통해 발생한 예외를 스프링 데이터 접근 예외로 변환한다.

#### 예외 변환 후&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-25 14.38.24.png" alt=""><figcaption></figcaption></figure>

결과적으로 리포지토리에 `@Repository` 애노테이션만 있으면 스프링이 예외 변환을 처리하는 AOP를 만들어준다.
