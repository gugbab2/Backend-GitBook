# 데이터 접근 기술 - 스프링 JdbcTemplate

## 1. JdbcTemplate 소개와 설정&#x20;

SQL 을 직접 사용하는 경우에 스프링이 제공하는 JdbcTemplate 은 아주 좋은 선택지이다. JdbcTemplate 은 JDBC 를 매우 편리하게 사용할 수 있게 도와준다.&#x20;

#### 장점

* 설정이 편리함&#x20;
  * JdbcTemplate 은 spring-jdbc 라이브러리에 포함되어 있는데, 이 라이브러리는 스프링으로 JDBC 를 사용할 때 기본으로 사용되는 라이브러리이다. 그리고 별도의 복잡한 설정 없이 바로 사용할 수 있다.
* 반복 문제 해결&#x20;
  * JdbcTemplate 은 템플릿 콜백 패턴을 사용해서 JDBC 를 직접 사용할 때 발생하는 대부분의 반복 작업을 대신 처리해준다.&#x20;
  * 개발자는 SQL 을 작성하고, 전달할 파라미터를 정의하고, 응답 값을 매핑하기만 하면 된다.&#x20;
  * **우리가 생각할 수 있는 대부분의 반복 작업을 대신 처리해준다.**&#x20;
    * **커넥션 획득**
    * **`statement` 를 준비하고 실행**
    * **결과를 반복하도록 루프를 실행**
    * **커넥션 종료, `statement`, `resultset` 종료**
    * **트랜잭션 다루기 위한 커넥션 동기화**
      * **`PlatformTransactionManager` 사용**&#x20;
    * **예외 발생시 스프링 예외 변환기 실행**

#### 단점&#x20;

* 동적 SQL 을 해결하기 어렵다.&#x20;

### JdbcTemplate 설정&#x20;

`build.gradle`

```gradle
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
    implementation 'org.springframework.boot:spring-boot-starter-web'

    //JdbcTemplate 추가
    implementation 'org.springframework.boot:spring-boot-starter-jdbc'
    //H2 데이터베이스 추가
    runtimeOnly 'com.h2database:h2'

    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'

    //테스트에서 lombok 사용
    testCompileOnly 'org.projectlombok:lombok'
    testAnnotationProcessor 'org.projectlombok:lombok'
}
```

## 2. JdbcTemplate 적용1 - 기본&#x20;

이제부터 본격적으로 `JdbcTemplate` 을 사용해서 메모리에 저장하던 데이터를 데이터베이스에 저장해보자. \
`ItemRepository` 인터페이스가 있으니 이 인터페이스를 기반으로 `JdbcTemplate` 을 사용하는 새로운 구현체를 개발하자.&#x20;

#### JdbcTemplateItemRepositoryV1&#x20;

```java
package hello.itemservice.repository.jdbctemplate;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;
import java.sql.PreparedStatement;
import java.util.ArrayList;
import java.util.List;
import java.util.Optional;

/**
 * JdbcTemplate 기반의 ItemRepository 구현체
 */
@Slf4j
public class JdbcTemplateItemRepositoryV1 implements ItemRepository {

    private final JdbcTemplate template;

    public JdbcTemplateItemRepositoryV1(DataSource dataSource) {
        this.template = new JdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "INSERT INTO item (item_name, price, quantity) VALUES (?, ?, ?)";
        KeyHolder keyHolder = new GeneratedKeyHolder();
        template.update(connection -> {
            // 자동 증가 키
            PreparedStatement ps = connection.prepareStatement(sql, new String[]{"id"});
            ps.setString(1, item.getItemName());
            ps.setInt(2, item.getPrice());
            ps.setInt(3, item.getQuantity());
            return ps;
        }, keyHolder);

        long key = keyHolder.getKey().longValue();
        item.setId(key);

        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "UPDATE item SET item_name = ?, price = ?, quantity = ? WHERE id = ?";
        template.update(sql,
                updateParam.getItemName(),
                updateParam.getPrice(),
                updateParam.getQuantity(),
                itemId);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "SELECT id, item_name, price, quantity FROM item WHERE id = ?";
        try {
            Item item = template.queryForObject(sql, itemRowMapper(), id);
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        String sql = "SELECT id, item_name, price, quantity FROM item";
        // 동적 쿼리 생성
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        List<Object> param = new ArrayList<>();
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',?,'%')";
            param.add(itemName);
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= ?";
            param.add(maxPrice);
        }
        log.info("sql={}", sql);
        return template.query(sql, itemRowMapper(), param.toArray());
    }

    private RowMapper<Item> itemRowMapper() {
        return ((rs, rowNum) -> {
            Item item = new Item();
            item.setId(rs.getLong("id"));
            item.setItemName(rs.getString("item_name"));
            item.setPrice(rs.getInt("price"));
            item.setQuantity(rs.getInt("quantity"));
            return item;
        });
    }
}
```

#### 기본&#x20;

* `JdbcTemplateRepositoryV1` 은 `ItemRepository` 인터페이스를 구현했다.&#x20;
* `this.template = new JdbcTemplate(dataSource)`&#x20;
  * `JdbcTemplate` 은 데이터소스(`dataSource`)가 필요하다.
  * `JdbcTemplateItemRepositoryV1()` 생성자를 보면 `dataSource` 를 의존 관계 주입 받고 생성자 내부에서 `JdbcTemplate` 을 생성한다. 스프링에서는 `JdbcTemplate` 을 사용할 때 관례상 이 방법을 많이 사용한다.
  * 물론 `JdbcTemplate` 을 스프링 빈으로 직접 등록하고 주입받아도 된다.

#### save()&#x20;

데이터를 저장한다.&#x20;

* `template.update()` : 데이터를 변경할 때는 `update()` 를 사용하면 된다.
  * `INSERT`, `UPDATE`, `DELETE` SQL에 사용한다.
  * `template.update()` 의 반환 값은 `int` 인데, 영향 받은 로우 수를 반환한다.
*   데이터를 저장할 때 PK 생성에 `identity` (auto increment) 방식을 사용하기 때문에, PK인 ID 값을 개발자가

    직접 지정하는 것이 아니라 비워두고 저장해야 한다. 그러면 데이터베이스가 PK인 ID를 대신 생성해준다.
*   문제는 이렇게 데이터베이스가 대신 생성해주는 PK ID 값은 데이터베이스가 생성하기 때문에, 데이터베이스에

    INSERT가 완료 되어야 생성된 PK ID 값을 확인할 수 있다.
*   `KeyHolder` 와 `connection.prepareStatement(sql, new String[]{"id"})` 를 사용해서 `id` 를

    지정해주면 `INSERT` 쿼리 실행 이후에 데이터베이스에서 생성된 ID 값을 조회할 수 있다.
* 물론 데이터베이스에서 생성된 ID 값을 조회하는 것은 순수 JDBC로도 가능하지만, 코드가 훨씬 더 복잡하다.
* 참고로 뒤에서 설명하겠지만 JdbcTemplate이 제공하는 `SimpleJdbcInsert` 라는 훨씬 편리한 기능이 있으므로 대략 이렇게 사용한다 정도로만 알아두면 된다.

#### Update()&#x20;

데이터를 업데이트 한다.&#x20;

* `template.update()` : 데이터를 변경할 때는 `update()` 를 사용하면 된다.&#x20;
  * `?` 에 바인딩할 파라미터를 순서대로 전달하면 된다.
  * 반환 값은 해당 쿼리의 영향을 받은 로우 수 이다. 여기서는 `where id=?` 를 지정했기 때문에 영향 받은 로우수는 최대 1개이다.

#### findById()

데이터를 하나 조회한다.&#x20;

* `template.queryForObject()`&#x20;
  * 결과 로우가 하나일 때 사용한다.
  * `RowMapper` 는 데이터베이스의 반환 결과인 `ResultSet` 을 객체로 변환한다.
  * 결과가 없으면 `EmptyResultDataAccessException` 예외가 발생한다.
  * 결과가 둘 이상이면 `IncorrectResultSizeDataAccessException` 예외가 발생한다.
*   `ItemRepository.findById()` 인터페이스는 결과가 없을 때 `Optional` 을 반환해야 한다. 따라서 결과가

    없으면 예외를 잡아서 `Optional.empty` 를 대신 반환하면 된다.

#### findAll()&#x20;

데이터를 리스트로 조회한다. 그리고 검색 조건으로 적절한 데이터를 찾는다.&#x20;

* `template.query()`&#x20;
  * 결과가 하나 이상일 때 사용한다.
  * `RowMapper` 는 데이터베이스의 반환 결과인 `ResultSet` 을 객체로 변환한다.
  * 결과가 없으면 빈 컬렉션을 반환한다.
  * 동적 쿼리에 대한 부분은 바로 다음에 다룬다.

#### **itemRowMapper()**

데이터베이스의 조회 결과를 객체로 변환할 때 사용한다.

JDBC를 직접 사용할 때 `ResultSet` 를 사용했던 부분을 떠올리면 된다.\
차이가 있다면 다음과 같이 `JdbcTemplate`이 다음과 같은 루프를 돌려주고, 개발자는 `RowMapper` 를 구현해서 \
그 내부 코드만 채운다고 이해하면 된다.

## 3. JdbcTemplate 적용2 - 동적 쿼리 문제&#x20;

결과를 검색하는 `findAll()` 에서 어려운 부분은 사용자가 검색하는 값에 따라서 실행하는 SQL이 동적으로 달려져야 \
한다는 점이다.

예를 들어서 다음과 같다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-23 15.48.14.png" alt=""><figcaption></figcaption></figure>

결과적으로 4가지 상황에 따른 SQL을 동적으로 생성해야 한다. 동적 쿼리가 언듯 보면 쉬워 보이지만, 막상 개발해보면 생각보다 다양한 상황을 고민해야 한다. 예를 들어서 어떤 경우에는 `where` 를 앞에 넣고 어떤 경우에는 `and` 를 넣어야 하는지 등을 모두 계산해야 한다.&#x20;

그리고 각 상황에 맞추어 파라미터도 생성해야 한다. \
(물론 실무에서는 이보다 더 복잡한 동적 쿼리들이 사용된다)&#x20;

참고로 이후에 설명할 MyBatis 의 가장 큰 장점은 SQL 을 직접 사용할 때 동적 쿼리를 쉽게 작성할 수 있다는 것이다.&#x20;

## 4. JdbcTemplate 적용3 - 구성과 실행&#x20;

#### JdbcTemplateV1Config

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.jdbctemplate.JdbcTemplateItemRepositoryV1;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@RequiredArgsConstructor
public class JdbcTemplateConfigV1 {

    private final DataSource dataSource;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JdbcTemplateItemRepositoryV1(dataSource);
    }

}
```

#### **ItemServiceApplication - 변경**

```java
//@Import(MemoryConfig.class)
@Import(JdbcTemplateV1Config.class)
@SpringBootApplication(scanBasePackages = "hello.itemservice.web")
public class ItemServiceApplication {}
```

#### 데이터베이스 접근 설정&#x20;

`src/main/resources/application.properties`

```properties
spring.profiles.active=local
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa
```

* 이렇게 설정만 하면 스프링 부트가 해당 설정을 사용해서 커넥션 풀과 `DataSource`, 트랜잭션 매니저를 스프링 빈으로 자동 등록한다.

#### 로그 추가&#x20;

JdbcTemplate이 실행하는 SQL 로그를 확인하려면 `application.properties` 에 다음을 추가하면 된다.\
`main`, `test` 설정이 분리되어 있기 때문에 둘다 확인하려면 두 곳에 모두 추가해야 한다.

```properties
#jdbcTemplate sql log
logging.level.org.springframework.jdbc=debug
```

## 5. JdbcTemplate - 이름 지정 파라미터1&#x20;

### 순서대로 바인딩&#x20;

`JdbcTemplate` 을 기본으로 사용하면 파라미터를 순서대로 바인딩한다.&#x20;

```java
String sql = "update item set item_name=?, price=?, quantity=? where id=?";
template.update(sql,
    itemName,
    price,
    quantity,
    itemId);
```

여기서는 `itemName`, `price`, `quantity` 가 SQL에 있는 `?` 에 순서대로 바인딩 된다.\
따라서 순서만 잘 지키면 문제가 될 것은 없다. 그런데 문제는 변경시점에 발생한다.

누군가 다음과 같이 SQL 코드의 순서를 변경했다고 가정해보자. (`price` 와 `quantity` 의 순서를 변경했다.)

```java
String sql = "update item set item_name=?, quantity=?, price=? where id=?";
template.update(sql,
    itemName,
    price,
    quantity,
    itemId);
```

이렇게 되면 다음과 같은 순서로 데이터가 바인딩 된다.\
`item_name=itemName, quantity=price, price=quantity`&#x20;

#### 순서대로 바인딩 문제점 정리&#x20;

결과적으로 `price` 와 `quantity` 가 바뀌는 매우 심각한 문제가 발생한다. 이럴일이 없을 것 같지만, 실무에서는 파라미터가 10\~20개가 넘어가는 일도 아주 많다. 그래서 미래에 필드를 추가하거나, 수정하면서 이런 문제가 충분히 발생할 수 있다.\
버그 중에서 가장 고치기 힘든 버그는 데이터베이스에 데이터가 잘못 들어가는 버그다. 이것은 코드만 고치는 수준이 아니라 데이터베이스의 데이터를 복구해야 하기 때문에 버그를 해결하는데 들어가는 리소스가 어마어마하다.\
실제로 수많은 개발자들이 이 문제로 장애를 내고 퇴근하지 못하는 일이 발생한다.

**개발을 할 때는 코드를 몇줄 줄이는 편리함도 중요하지만, 모호함을 제거해서 코드를 명확하게 만드는 것이 유지보수 관점에서 매우 중요하다.**

이처럼 파라미터를 순서대로 바인딩 하는 것은 편리하기는 하지만, 순서가 맞지 않아서 버그가 발생할 수도 있으므로 주의해서 사용해야 한다.

### 이름 지정 바인딩&#x20;

`JdbcTemplate`은 이런 문제를 보완하기 위해 `NamedParameterJdbcTemplate` 라는 이름을 지정해서 파라미터를 바인딩 하는 기능을 제공한다.

#### JdbcTemplateItemRepositoryV2&#x20;

```java
package hello.itemservice.repository.jdbctemplate;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.jdbc.support.GeneratedKeyHolder;
import org.springframework.jdbc.support.KeyHolder;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;
import java.sql.PreparedStatement;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Optional;

/**
 * NamedParameterJdbcTemplate 기반의 ItemRepository 구현체
 */
@Slf4j
public class JdbcTemplateItemRepositoryV2 implements ItemRepository {

//    private final JdbcTemplate template;
    private final NamedParameterJdbcTemplate template;

    public JdbcTemplateItemRepositoryV2(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
    }

    @Override
    public Item save(Item item) {
        String sql = "INSERT INTO item (item_name, price, quantity) " +
                "VALUES (:itemName, :price, :quantity)";

        SqlParameterSource param = new BeanPropertySqlParameterSource(item);

        KeyHolder keyHolder = new GeneratedKeyHolder();
        template.update(sql, param, keyHolder);

        long key = keyHolder.getKey().longValue();
        item.setId(key);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "UPDATE item " +
                "SET item_name = :itemName, price = :price, quantity = :quantity " +
                "WHERE id = :itemId";

        MapSqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("itemId", itemId);
        template.update(sql, param);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "SELECT id, item_name, price, quantity FROM item WHERE id = :itemId";
        try {
            Map<String, Object> param = Map.of("itemId", id);
            Item item = template.queryForObject(sql, param, itemRowMapper());
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

        String sql = "SELECT id, item_name, price, quantity FROM item";
        // 동적 쿼리 생성
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= :maxPrice";
        }
        log.info("sql={}", sql);

        return template.query(sql, param, itemRowMapper());
    }

    private RowMapper<Item> itemRowMapper() {
        return BeanPropertyRowMapper.newInstance(Item.class);
    }
}
```

#### 기본&#x20;

* `JdbcTemplateItemRepositoryV2` 는 `ItemRepository` 인터페이스를 구현했다.
* `this.template = new NamedParameterJdbcTemplate(dataSource)`&#x20;
  * `NamedParameterJdbcTemplate` 도 내부에 `dataSource` 가 필요하다.
  *   `JdbcTemplateItemRepositoryV2` 생성자를 보면 의존관계 주입은 `dataSource` 를 받고 내부에서

      `NamedParameterJdbcTemplate` 을 생성해서 가지고 있다. 스프링에서는 `JdbcTemplate` 관련 기능을 사용할 때 관례상 이 방법을 많이 사용한다.
  * 물론 `NamedParameterJdbcTemplate` 을 스프링 빈으로 직접 등록하고 주입받아도 된다.

#### save()&#x20;

SQL 에서 다음과 같이 `?` 대신에 `:파라미터이름` 을 받는 것을 확인할 수 있다.&#x20;

```java
insert into item (item_name, price, quantity) " +
                "values (:itemName, :price, :quantity)"
```

추가로 `NamedParameterJdbcTemplate` 은 데이터베이스가 생성해주는 키를 매우 쉽게 조회하는 기능도 제공해준다.

## 6. JdbcTemplate - 이름 지정 파라미터2&#x20;

### 이름 지정 파라미터

파라미터를 전달하려면 `Map` 처럼 `key`, `value` 데이터 구조를 만들어서 전달해야 한다.

여기서 `key` 는 `:파리이터이름` 으로 지정한, 파라미터의 이름이고, `value` 는 해당 파라미터의 값이 된다.

#### 이름 지정 바인딩에서 자주 사용하는 파라미터의 종류는 크게 3가지가 있다.

* `Map`
* `SqlParameterSource`&#x20;
  * `MapSqlParameterSource`&#x20;
  * `BeanPropertySqlParamterSource`&#x20;

#### 1. Map&#x20;

단순히 `Map` 을 사용한다 .

`findById()` 코드에서 확인할 수 있다.

```java
Map<String, Object> param = Map.of("itemId", id);
Item item = template.queryForObject(sql, param, itemRowMapper()
```

#### 2. MapSqlParameterSource

`Map` 과 유사한데, SQL 타입을 지정할 수 있는 등 SQL 에 좀 더 특화된 기능을 제공한다. \
`SqlParameterSource` 인터페이스의 구현체이다.\
`MapSqlParameterSource` 는 메서드 체인을 통해 편리한 사용법도 제공한다.

`update()` 코드에서 확인할 수 있다.

```java
MapSqlParameterSource param = new MapSqlParameterSource()
    .addValue("itemName", updateParam.getItemName())
    .addValue("price", updateParam.getPrice())
    .addValue("quantity", updateParam.getQuantity())
    .addValue("itemId", itemId);
template.update(sql, param);
```

#### 3. BeanPropertySqlParameterSource

자바빈 프로퍼티 규약을 통해서 자동으로 파라미터 객체를 생성한다.\
예) (`getXxx() -> xxx, getItemName() -> itemName` )

예를 들어서 `getItemName()`, `getPrice()` 가 있으면 다음과 같은 데이터를 자동으로 만들어낸다.

* `key=itemName, value=상품명 값`&#x20;
* `key=price, value=가격 값`

`SqlParameterSource` 인터페이스의 구현체이다.

`save()` , `findAll()` 코드에서 확인할 수 있다.

```java
SqlParameterSource param = new BeanPropertySqlParameterSource(item);
KeyHolder keyHolder = new GeneratedKeyHolder();
template.update(sql, param, keyHolder);
```

*   여기서 보면 `BeanPropertySqlParameterSource` 가 많은 것을 자동화 해주기 때문에 가장 좋아보이지만,

    `BeanPropertySqlParameterSource` 를 항상 사용할 수 있는 것은 아니다.
*   예를 들어서 `update()` 에서는 SQL에 `:id` 를 바인딩 해야 하는데, `update()` 에서 사용하는

    `ItemUpdateDto` 에는 `itemId` 가 없다. 따라서 `BeanPropertySqlParameterSource` 를 사용할 수 없고, \
    대신에 `MapSqlParameterSource` 를 사용했다.

### BeanPropertyRowMapper&#x20;

이번 코드에서 `V1` 과 비교해서 변화된 부분이 하나 더 있다. 바로 `BeanPropertyRowMapper` 를 사용한 것이다.

#### JdbcTemplateItemRepositoryV1 - itemRowMapper()

```java
private RowMapper<Item> itemRowMapper() {
    return (rs, rowNum) -> {
        Item item = new Item();
        item.setId(rs.getLong("id"));
        item.setItemName(rs.getString("item_name"));
        item.setPrice(rs.getInt("price"));
        item.setQuantity(rs.getInt("quantity"));
        return item;
    };
}
```

#### JdbcTemplateItemRepositoryV2 - itemRowMapper()

```java
private RowMapper<Item> itemRowMapper() {
    return BeanPropertyRowMapper.newInstance(Item.class); //camel 변환 지원
}
```

`BeanPropertyRowMapper` 는 `ResultSet` 의 결과를 받아서 자바빈 규약에 맞추어 데이터를 변환한다.\
예를 들어서 데이터베이스에서 조회한 결과가 `select id, price` 라고 하면 다음과 같은 코드를 작성해준다. \
(실제로는 리플렉션 같은 기능을 사용한다.)

```java
Item item = new Item();
item.setId(rs.getLong("id"));
item.setPrice(rs.getInt("price"));
```

데이터베이스에서 조회한 결과 이름을 기반으로 `setId()`, `setPrice()` 처럼 자바빈 프로퍼티 규약에 맞춘 메서드를 호출하는 것이다.

#### 별칭&#x20;

그런데 `select item_name` 의 경우 `setItem_name()` 이라는 메서드가 없기 때문에 골치가 아프다.\
이런 경우 개발자가 조회 SQL을 다음과 같이 고치면 된다.\
`select item_name as itemName`&#x20;

별칭 `as` 를 사용해서 SQL 조회 결과의 이름을 변경하는 것이다. 실제로 이 방법은 자주 사용된다. 특히 데이터베이스\
컬럼 이름과 객체 이름이 완전히 다를 때 문제를 해결할 수 있다. 예를 들어서 데이터베이스에는 `member_name` 이라고 되어 있는데 객체에 `username` 이라고 되어 있다면 다음과 같이 해결할 수 있다.\
`select member_name as username`

이렇게 데이터베이스 컬럼 이름과 객체의 이름이 다를 때 별칭(`as`)을 사용해서 문제를 많이 해결한다.\
`JdbcTemplate` 은 물론이고, `MyBatis` 같은 기술에서도 자주 사용된다.

#### 관례의 불일치&#x20;

자바 객체는 카멜(`camelCase`) 표기법을 사용한다. `itemName` 처럼 중간에 낙타 봉이 올라와 있는 표기법이다.\
반면에 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 `snake_case` 표기법을 사용한다. `item_name` 처럼 중간에 언더스코어를 사용하는 표기법이다.

이 부분을 관례로 많이 사용하다 보니 `BeanPropertyRowMapper` 는 언더스코어 표기법을 카멜로 자동 변환해준다.\
따라서 `select item_name` 으로 조회해도 `setItemName()` 에 문제 없이 값이 들어간다.

정리하면 `snake_case` 는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회\
SQL에서 별칭을 사용하면 된다.

## 7. JdbcTemplate - 이름 지정 파라미터3&#x20;

이제 이름 지정 파라미터를 사용하도록 구성하고 실행해보자.

#### JdbcTemplateV2Config

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.jdbctemplate.JdbcTemplateItemRepositoryV2;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@RequiredArgsConstructor
public class JdbcTemplateConfigV2 {

    private final DataSource dataSource;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JdbcTemplateItemRepositoryV2(dataSource);
    }

}
```

#### **ItemServiceApplication - 변경**

```java
package hello.itemservice;

import hello.itemservice.config.*;
import hello.itemservice.repository.ItemRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Profile;


//@Import(MemoryConfig.class)
//@Import(JdbcTemplateConfigV1.class)
@Import(JdbcTemplateConfigV2.class)
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

## 8. JdbcTemplate - SimpleJdbcInsert

`JdbcTemplate`은 INSERT SQL를 직접 작성하지 않아도 되도록 `SimpleJdbcInsert` 라는 편리한 기능을 제공한다.

#### JdbcTemplateItemRepositoryV3&#x20;

```java
package hello.itemservice.repository.jdbctemplate;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.extern.slf4j.Slf4j;
import org.springframework.dao.EmptyResultDataAccessException;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.MapSqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;
import org.springframework.jdbc.core.simple.SimpleJdbcInsert;
import org.springframework.util.StringUtils;

import javax.sql.DataSource;
import java.util.List;
import java.util.Map;
import java.util.Optional;

/**
 * SimpleJdbcTemplate 기반의 ItemRepository 구현체
 */
@Slf4j
public class JdbcTemplateItemRepositoryV3 implements ItemRepository {

//    private final JdbcTemplate template;
    private final NamedParameterJdbcTemplate template;
    private final SimpleJdbcInsert jdbcInsert;

    public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
        this.jdbcInsert = new SimpleJdbcInsert(dataSource)
                .withTableName("item")
                .usingGeneratedKeyColumns("id");
//                .usingColumns("item_name", "price", "quantity");  // 생략 가능
    }

    @Override
    public Item save(Item item) {
        SqlParameterSource param = new BeanPropertySqlParameterSource(item);
        Number key = jdbcInsert.executeAndReturnKey(param);
        item.setId(key.longValue());
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        String sql = "UPDATE item " +
                "SET item_name = :itemName, price = :price, quantity = :quantity " +
                "WHERE id = :itemId";

        MapSqlParameterSource param = new MapSqlParameterSource()
                .addValue("itemName", updateParam.getItemName())
                .addValue("price", updateParam.getPrice())
                .addValue("quantity", updateParam.getQuantity())
                .addValue("itemId", itemId);
        template.update(sql, param);
    }

    @Override
    public Optional<Item> findById(Long id) {
        String sql = "SELECT id, item_name, price, quantity FROM item WHERE id = :itemId";
        try {
            Map<String, Object> param = Map.of("itemId", id);
            Item item = template.queryForObject(sql, param, itemRowMapper());
            return Optional.of(item);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();

        SqlParameterSource param = new BeanPropertySqlParameterSource(cond);

        String sql = "SELECT id, item_name, price, quantity FROM item";
        // 동적 쿼리 생성
        if (StringUtils.hasText(itemName) || maxPrice != null) {
            sql += " where";
        }
        boolean andFlag = false;
        if (StringUtils.hasText(itemName)) {
            sql += " item_name like concat('%',:itemName,'%')";
            andFlag = true;
        }
        if (maxPrice != null) {
            if (andFlag) {
                sql += " and";
            }
            sql += " price <= :maxPrice";
        }
        log.info("sql={}", sql);

        return template.query(sql, param, itemRowMapper());
    }

    private RowMapper<Item> itemRowMapper() {
        return BeanPropertyRowMapper.newInstance(Item.class);
    }
}
```

#### 기본&#x20;

* `JdbcTemplateItemRepositoryV3` 은 `ItemRepository` 인터페이스를 구현했다.
*   `this.jdbcInsert = new SimpleJdbcInsert(dataSource)` : 생성자를 보면 의존관계 주입은

    `dataSource` 를 받고 내부에서 `SimpleJdbcInsert` 을 생성해서 가지고 있다. 스프링에서는 `JdbcTemplate` 관련 기능을 사용할 때 관례상 이 방법을 많이 사용한다.

    * 물론 `SimpleJdbcInsert` 을 스프링 빈으로 직접 등록하고 주입받아도 된다.

#### SimpleJdbcInsert

```java
public JdbcTemplateItemRepositoryV3(DataSource dataSource) {
        this.template = new NamedParameterJdbcTemplate(dataSource);
        this.jdbcInsert = new SimpleJdbcInsert(dataSource)
                .withTableName("item")
                .usingGeneratedKeyColumns("id");
//                .usingColumns("item_name", "price", "quantity");  // 생략 가능
}
```

* `withTableName` : 데이터를 저장할 테이블 명을 지정한다.
* `usingGeneratedKeyColumns` : `key` 를 생성하는 PK 컬럼 명을 지정한다.
* `usingColumns` : INSERT SQL에 사용할 컬럼을 지정한다. 특정 값만 저장하고 싶을 때 사용한다. 생략할 수 있다.

`SimpleJdbcInsert` 는 생성 시점에 데이터베이스 테이블의 메타 데이터를 조회한다. 따라서 어떤 컬럼이 있는지 확인 할 수 있으므로 `usingColumns` 을 생략할 수 있다. 만약 특정 컬럼만 지정해서 저장하고 싶다면 `usingColumns`를 사용하면 된다.

애플리케이션을 실행해보면 `SimpleJdbcInsert` 이 어떤 INSERT SQL을 만들어서 사용하는지 로그로 확인할 수 있다.

```
DEBUG 39424 --- [ main] o.s.jdbc.core.simple.SimpleJdbcInsert :
Compiled insert object: insert string is [INSERT INTO item (ITEM_NAME, PRICE,
QUANTITY) VALUES(?, ?, ?)]
```

#### save()&#x20;

`jdbcInsert.executeAndReturnKey(param)` 을 사용해서 INSERT SQL을 실행하고, 생성된 키 값도 매우 편리하게 조회할 수 있다.&#x20;

```java
public Item save(Item item) {
    SqlParameterSource param = new BeanPropertySqlParameterSource(item);
    Number key = jdbcInsert.executeAndReturnKey(param);
    item.setId(key.longValue());
    return item;
}
```

#### JdbcTemplateV3Config

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.jdbctemplate.JdbcTemplateItemRepositoryV3;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@RequiredArgsConstructor
public class JdbcTemplateConfigV3 {

    private final DataSource dataSource;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new JdbcTemplateItemRepositoryV3(dataSource);
    }

}
```

#### ItemServiceApplication - 변경

```java
package hello.itemservice;

import hello.itemservice.config.*;
import hello.itemservice.repository.ItemRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Profile;


//@Import(MemoryConfig.class)
//@Import(JdbcTemplateConfigV1.class)
//@Import(JdbcTemplateConfigV2.class)
@Import(JdbcTemplateConfigV3.class)
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

## 9. JdbcTemplate 기능 정리

### 주요 기능&#x20;

* `JdbcTemplate`&#x20;
  * 순서 기반 파라미터 바인딩을 지원한다.
* `NamedParameterJdbcTemplate`
  * 이름 기반 파라미터 바인딩을 지원한다. (권장)
* `SimpleJdbcInsert`&#x20;
  * INSERT SQL을 편리하게 사용할 수 있다.
* `SimpleJdbcCall`&#x20;
  * 스토어드 프로시저를 편리하게 호출할 수 있다.

> #### 참고
>
> 스토어드 프로시저를 사용하기 위한 `SimpleJdbcCall` 에 대한 자세한 내용은 다음 스프링 공식 메뉴얼을 참고하자.
>
> [https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-simple-jdbc-call-1](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-simple-jdbc-call-1)

### JdbcTemplate 사용법 정리

`JdbcTemplate`에 대한 사용법은 스프링 공식 메뉴얼에 자세히 소개되어 있다.&#x20;

> #### 참고
>
> 스프링 `JdbcTemplate` 사용 방법 공식 메뉴얼
>
> [https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-JdbcTemplate](https://docs.spring.io/spring-framework/docs/current/reference/html/data-access.html#jdbc-JdbcTemplate)
