# 데이터 접근 기술 - MyBatis

## 1. MyBatis 소개&#x20;

MyBatis는 앞서 설명한 JdbcTemplate보다 더 많은 기능을 제공하는 SQL Mapper 이다.

기본적으로 JdbcTemplate이 제공하는 대부분의 기능을 제공한다.\
JdbcTemplate과 비교해서 MyBatis의 가장 매력적인 점은 SQL을 XML에 편리하게 작성할 수 있고 또 동적 쿼리를 \
매우 편리하게 작성할 수 있다는 점이다.

먼저 SQL이 여러줄에 걸쳐 있을 때 둘을 비교해보자.

**JdbcTemplate - SQL 여러줄**

```java
String sql = "update item " +
        "set item_name=:itemName, price=:price, quantity=:quantity " +
        "where id=:id";
```

**MyBatis - SQL 여러줄**

```xml
<update id="update">
    update item
    set item_name=#{itemName},
        price=#{price},
        quantity=#{quantity}
    where id = #{id}
</update>
```

MyBatis는 XML에 작성하기 때문에 라인이 길어져도 문자 더하기에 대한 불편함이 없다.

다음으로 상품을 검색하는 로직을 통해 동적 쿼리를 비교해보자.

**JdbcTemplate - 동적 쿼리**

```java
String sql = "select id, item_name, price, quantity from item";
//동적 쿼리
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
```

**MyBatis - 동적 쿼리**

```xml
<select id="findAll" resultType="Item">
    select id, item_name, price, quantity
    from item
    <where>
        <if test="itemName != null and itemName != ''">
            and item_name like concat('%',#{itemName},'%')
        </if>
        <if test="maxPrice != null">
            and price &lt;= #{maxPrice}
        </if>
    </where>
</select>
```

JdbcTemplate은 자바 코드로 직접 동적 쿼리를 작성해야 한다. 반면에 MyBatis는 동적 쿼리를 매우 편리하게 작성할 수 있는 다양한 기능들을 제공해준다.

#### 설정의 장단점

JdbcTemplate은 스프링에 내장된 기능이고, 별도의 설정없이 사용할 수 있다는 장점이 있다. 반면에 MyBatis는 약간의 설정이 필요하다.

#### 정리&#x20;

프로젝트에서 동적 쿼리와 복잡한 쿼리가 많다면 MyBatis를 사용하고, 단순한 쿼리들이 많으면 JdbcTemplate을 선택해서 사용하면 된다. 물론 둘을 함께 사용해도 된다. 하지만 MyBatis를 선택했다면 그것으로 충분할 것이다.

> #### 공식 사이트
>
> [https://mybatis.org/mybatis-3/ko/index.html](https://mybatis.org/mybatis-3/ko/index.html)

## 2. MyBatis 설정&#x20;

`build.gradle` 에 다음 의존 관계를 추가한다.

```gradle
//MyBatis 추가
implementation 'org.mybatis.spring.boot:mybatis-spring-boot-starter:2.2.0'
```

* 참고로 뒤에 버전 정보가 붙는 이유는 스프링 부트가 버전을 관리해주는 공식 라이브러리가 아니기 때문이다. 스프링 부트가 버전을 관리해주는 경우 버전 정보를 붙이지 않아도 최적의 버전을 자동으로 찾아준다.

#### 설정&#x20;

`main - application.properties`

```properties
spring.profiles.active=local
spring.datasource.url=jdbc:h2:tcp://localhost/~/test
spring.datasource.username=sa

logging.level.org.springframework.jdbc=debug

#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

`test - application.properties`

```properties
spring.profiles.active=test
#spring.datasource.url=jdbc:h2:tcp://localhost/~/testcase
#spring.datasource.username=sa

logging.level.org.springframework.jdbc=debug

#MyBatis
mybatis.type-aliases-package=hello.itemservice.domain
mybatis.configuration.map-underscore-to-camel-case=true
logging.level.hello.itemservice.repository.mybatis=trace
```

* `mybatis.type-aliases-package`
  * 마이바티스에서 타입 정보를 사용할 때는 패키지 이름을 적어주어야 하는데, 여기에 명시하면 패키지 이름을 생략할 수 있다.
  * 지정한 패키지와 그 하위 패키지가 자동으로 인식된다.
  * 여러 위치를 지정하려면 `,` , `;` 로 구분하면 된다.
* `mybatis.configuration.map-underscore-to-camel-case`&#x20;
  * JdbcTemplate의 `BeanPropertyRowMapper` 에서 처럼 언더바를 카멜로 자동 변경해주는 기능을 활성화 한다. 바로 다음에 설명하는 관례의 불일치 내용을 참고하자.
* `logging.level.hello.itemservice.repository.mybatis=trace`&#x20;
  * MyBatis에서 실행되는 쿼리 로그를 확인할 수 있다.

#### 관례의 불일치&#x20;

자바 객체에는 주로 카멜케이스 표기법을 사용한다. \
반면에 관계형 데이터베이스에서는 주로 언더스코어를 사용하는 `snake_case` 표기법을 사용한다.

이렇게 관례로 많이 사용하다 보니 `map-underscore-to-camel-case` 기능을 활성화 하면 언더스코어 표기법을 카멜로 자동 변환해준다. 따라서 DB에서 `select item_name`으로 조회해도 객체의 `itemName` (`setItemName()`) 속성에 값이 정상 입력된다.

정리하면 해당 옵션을 켜면 `snake_case` 는 자동으로 해결되니 그냥 두면 되고, 컬럼 이름과 객체 이름이 완전히 다른 경우에는 조회 SQL에서 별칭을 사용하면 된다.

## 3. Mybatis 적용1 - 기본&#x20;

XML에 작성한다는 점을 제외하고는 JDBC 반복을 줄여준다는 점에서 기존 JdbcTemplate과 거의 유사하다.

#### ItemMapper&#x20;

```java
package hello.itemservice.repository.mybatis;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;
import java.util.Optional;

@Mapper
public interface ItemMapper {

    void save(Item item);

    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);

    Optional<Item> findById(Long id);

    List<Item> findAll(ItemSearchCond itemSearchCond);
}
```

* 마이바티스 매핑 XML을 호출해주는 매퍼 인터페이스이다.
* 이 인터페이스에는 `@Mapper` 애노테이션을 붙여주어야 한다. 그래야 MyBatis에서 인식할 수 있다.
* 이 인터페이스의 메서드를 호출하면 다음에 보이는 `xml` 의 해당 SQL을 실행하고 결과를 돌려준다.
* `ItemMapper` 인터페이스의 구현체에 대한 부분은 뒤에 별도로 설명한다.

이제 같은 위치에 실행할 SQL이 있는 XML 매핑 파일을 만들어주면 된다.\
참고로 자바 코드가 아니기 때문에 `src/main/resources` 하위에 만들되, 패키지 위치는 맞추어 주어야 한다.

`src/main/resources/hello/itemservice/repository/mybatis/ItemMapper.xml`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="hello.itemservice.repository.mybatis.ItemMapper">
    <insert id="save" useGeneratedKeys="true" keyProperty="id">
        insert into item (item_name, price, quantity)
        values (#{itemName}, #{price}, #{quantity})
    </insert>
    <update id="update">
        update item
        set item_name=#{updateParam.itemName},
        price=#{updateParam.price},
        quantity=#{updateParam.quantity}
        where id = #{id}
    </update>
    <select id="findById" resultType="Item">
        select id, item_name, price, quantity
        from item
        where id = #{id}
    </select>
    <select id="findAll" resultType="Item">
        select id, item_name, price, quantity
        from item
        <where>
            <if test="itemName != null and itemName != ''">
                and item_name like concat('%',#{itemName},'%')
            </if>
            <if test="maxPrice != null">
                and price &lt;= #{maxPrice}
            </if>
        </where>
    </select>
</mapper>
```

* `namespace` : 앞서 만든 매퍼 인터페이스를 지정하면 된다.
* **주의!** 경로와 파일 이름에 주의하자.

#### **insert - save**

```xml
<insert id="save" useGeneratedKeys="true" keyProperty="id">
    insert into item (item_name, price, quantity)
    values (#{itemName}, #{price}, #{quantity})
</insert>
```

* Insert SQL은 `<insert>` 를 사용하면 된다.
*   `id` 에는 매퍼 인터페이스에 설정한 메서드 이름을 지정하면 된다. 여기서는 메서드 이름이 `save()` 이므로

    `save` 로 지정하면 된다.
* 파라미터는 `#{}` 문법을 사용하면 된다. 그리고 매퍼에서 넘긴 객체의 프로퍼티 이름을 적어주면 된다.
* `#{}` 문법을 사용하면 `PreparedStatement` 를 사용한다. JDBC의 `?` 를 치환한다 생각하면 된다.
* `useGeneratedKeys` 는 데이터베이스가 키를 생성해 주는 `IDENTITY` 전략일 때 사용한다. `keyProperty` 는 생성되는 키의 속성 이름을 지정한다. Insert가 끝나면 `item` 객체의 `id` 속성에 생성된 값이 입력된다.

#### update - update

```xml
<update id="update">
    update item
    set item_name=#{updateParam.itemName},
        price=#{updateParam.price},
        quantity=#{updateParam.quantity}
    where id = #{id}
</update>
```

* Update SQL은 `<update>` 를 사용하면 된다.
*   여기서는 파라미터가 `Long id`, `ItemUpdateDto updateParam`으로 2개이다. 파라미터가 1개만 있으면

    `@Param` 을 지정하지 않아도 되지만, 파라미터가 2개 이상이면 `@Param`으로 이름을 지정해서 파라미터를 구분해야 한다.

**select - findById**

```xml
<select id="findById" resultType="Item">
    select id, item_name, price, quantity
    from item
    where id = #{id}
</select>
```

* Select SQL은 `<select>` 를 사용하면 된다.
* `resultType` 은 반환 타입을 명시하면 된다. 여기서는 결과를 `Item` 객체에 매핑한다.
  * 앞서 `application.properties` 에 `mybatis.type-aliases-package=hello.itemservice.domain` 속성을 지정한 덕분에 모든 패키지 명을 다 적지는 않아도 된다. \
    그렇지 않으면 모든 패키지 명을 다 적어야 한다.
  * JdbcTemplate의 `BeanPropertyRowMapper` 처럼 SELECT SQL의 결과를 편리하게 객체로 바로 변환해준다.
  * `mybatis.configuration.map-underscore-to-camel-case=true` 속성을 지정한 덕분에 언더스코어를 카멜 표기법으로 자동으로 처리해준다. (`item_name` -> `itemName`)
* 자바 코드에서 반환 객체가 하나이면 `Item`, `Optional<Item>` 과 같이 사용하면 되고, 반환 객체가 하나 이상이면 컬렉션을 사용하면 된다. 주로 `List` 를 사용한다. 다음을 참고하자.

#### **select - findAll**

```xml
<select id="findAll" resultType="Item">
    select id, item_name, price, quantity
    from item
    <where>
        <if test="itemName != null and itemName != ''">
            and item_name like concat('%',#{itemName},'%')
        </if>
        <if test="maxPrice != null">
            and price &lt;= #{maxPrice}
        </if>
    </where>
</select>
```

* Mybatis는 `<where>`, `<if>` 같은 동적 쿼리 문법을 통해 편리한 동적 쿼리를 지원한다.
* `<if>` 는 해당 조건이 만족하면 구문을 추가한다.
* `<where>` 은 적절하게 `where` 문장을 만들어준다.
  * 예제에서 `<if>` 가 모두 실패하게 되면 SQL `where` 를 만들지 않는다.
  * 예제에서 `<if>` 가 하나라도 성공하면 처음 나타나는 `and` 를 `where`로 변환해준다.

## 4. MyBatis 적용2 - 설정과 실행

#### MyBatisItemRepository

```java
package hello.itemservice.repository.mybatis;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Repository;

import java.util.List;
import java.util.Optional;

@Slf4j
@Repository
@RequiredArgsConstructor
public class MyBatisItemRepository implements ItemRepository {

    private final ItemMapper itemMapper;

    @Override
    public Item save(Item item) {
        log.info("itemMapper class= {}", itemMapper.getClass());
        itemMapper.save(item);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        itemMapper.update(itemId, updateParam);
    }

    @Override
    public Optional<Item> findById(Long id) {
        return itemMapper.findById(id);
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        return itemMapper.findAll(cond);
    }
}
```

* `ItemRepository` 를 구현해서 `MyBatisItemRepository` 를 만들자.
* `MyBatisItemRepository` 는 단순히 `ItemMapper` 에 기능을 위임한다.

#### MyBatisConfig

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.jdbctemplate.JdbcTemplateItemRepositoryV3;
import hello.itemservice.repository.mybatis.ItemMapper;
import hello.itemservice.repository.mybatis.MyBatisItemRepository;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import javax.sql.DataSource;

@Configuration
@RequiredArgsConstructor
public class MybatisConfig {

    private final ItemMapper itemMapper;

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new MyBatisItemRepository(itemMapper);
    }

}
```

#### ItemServiceApplication - 변경

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
import org.springframework.jdbc.datasource.DriverManagerDataSource;

import javax.sql.DataSource;

@Slf4j
//@Import(MemoryConfig.class)
//@Import(JdbcTemplateConfigV1.class)
//@Import(JdbcTemplateConfigV2.class)
//@Import(JdbcTemplateConfigV3.class)
@Import(MybatisConfig.class)
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

## 5. MyBatis 적용3 - 분석

생각해보면 지금까지 진행한 내용중에 약간 이상한 부분이 있다.\
`ItemMapper` 매퍼 인터페이스의 구현체가 없는데 어떻게 동작한 것일까?

#### ItemMapper 인터페이스

```java
package hello.itemservice.repository.mybatis;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import org.apache.ibatis.annotations.Mapper;
import org.apache.ibatis.annotations.Param;

import java.util.List;
import java.util.Optional;

@Mapper
public interface ItemMapper {

    void save(Item item);

    void update(@Param("id") Long id, @Param("updateParam") ItemUpdateDto updateParam);

    Optional<Item> findById(Long id);

    List<Item> findAll(ItemSearchCond itemSearchCond);
}
```

이 부분은 MyBatis 스프링 연동 모듈에서 자동으로 처리해주는데 다음과 같다.

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-06-24 20.25.44.png" alt=""><figcaption></figcaption></figure>

1. 애플리케이션 로딩 시점에 MyBatis 스프링 연동 모듈은 `@Mapper` 가 붙어있는 인터페이스를 조사한다.
2. 해당 인터페이스가 발견되면 동적 프록시 기술을 사용해서 `ItemMapper` 인터페이스의 구현체를 만든다.
3. 생성된 구현체를 스프링 빈으로 등록한다.

#### **매퍼 구현체**

* 마이바티스 스프링 연동 모듈이 만들어주는 `ItemMapper` 의 구현체 덕분에 인터페이스 만으로 편리하게 XML의 \
  데이터를 찾아서 호출할 수 있다.
*   원래 마이바티스를 사용하려면 더 번잡한 코드를 거쳐야 하는데, 이런 부분을 인터페이스 하나로 매우 깔끔하고

    편리하게 사용할 수 있다.
*   매퍼 구현체는 예외 변환까지 처리해준다. MyBatis에서 발생한 예외를 스프링 예외 추상화인 `DataAccessException` 에 맞게 변환해서 반환해준다. JdbcTemplate이 제공하는 예외 변환 기능을 여기서

    도 제공한다고 이해하면 된다.

#### 정리

* 매퍼 구현체 덕분에 마이바티스를 스프링에 편리하게 통합해서 사용할 수 있다.
* 매퍼 구현체를 사용하면 스프링 예외 추상화도 함께 적용된다.
* 마이바티스 스프링 연동 모듈이 많은 부분을 자동으로 설정해주는데, 데이터베이스 커넥션, 트랜잭션과 관련된 기능도 마이바티스와 함께 연동하고, 동기화해준다.
