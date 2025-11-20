# 데이터 접근 기술 - 시작

## 1. 데이터 접근 기술&#x20;

### SQL Mapper&#x20;

* JdbcTemplate&#x20;
* MyBatis&#x20;

#### SQL Mapper 주요 기능&#x20;

* 개발자는 SQL 만 작성하면 해당 SQL 의 결과를 객체로 편리하게 매핑해준다.&#x20;
* JDBC 를 직접 사용할 때는 여러가지 중복을 제거해주고 기타 개발자에게 여러가지 편리한 기능을 제공해준다.&#x20;

### ORM 관련 기술&#x20;

* JPA, Hibernate&#x20;
* 스프링 데이터 JPA&#x20;
* Querydsl&#x20;

#### ORM 주요 기능&#x20;

* JdbcTemplate 이나 MyBatis 같은 SQL 매퍼 기술은 SQL 을 개발자가 직접 작성해야 하지만, JPA 를 사용하면 기본적인 SQL 은 JPA 가 대신 작성하고 처리해준다. 개발자는 저장하고 싶은 객체를 마치 자바 컬렉션에 저장하고 조회하듯 사용하면 ORM 기술이 데이터베이스에 해당 객체를 저장하고 조회해준다.&#x20;
* JPA 는 자바 진영의 ORM 표준이고, Hibernate 는 JPA 에서 가장 많이 사용하는 구현체이다. 자바에서 ORM 을 사용할 때는 JPA 인터페이스를 사용하고, 그 구현체로 하이버네이트를 사용한다고 생각하면 된다.&#x20;
* 스프링 데이터 JPA, Querydsl 은 JPA 를 더 편리하게 사용할 수 있게 도와주는 프로젝트이다. 실무에서는 JPA 를 사용하면 이 프로젝트를 꼭! 함께 사용하는 것이 좋다.&#x20;

## 2. 프로젝트 구조 설명1 - 기본&#x20;

### 도메인 분석&#x20;

#### Item&#x20;

```java
package hello.itemservice.domain;

import lombok.Data;

@Data
public class Item {

    private Long id;

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

* `Item` 은 상품 자체를 나타내는 객체이다. 이름, 가격, 수량을 속성으로 가지고 있다.

### 리포지토리 분석&#x20;

#### ItemRepository 인터페이스&#x20;

```java
package hello.itemservice.repository;

import hello.itemservice.domain.Item;

import java.util.List;
import java.util.Optional;

public interface ItemRepository {

    Item save(Item item);

    void update(Long itemId, ItemUpdateDto updateParam);

    Optional<Item> findById(Long id);

    List<Item> findAll(ItemSearchCond cond);

}
```

* 메모리 구현체에서 향후 다양한 데이터 접근 기술 구현체로 손쉽게 변경하기 위해 리포지토리에 인터페이스를 도입했다.
* 각각의 기능은 메서드 이름으로 충분히 이해가 될 것이다.

#### ItemSearchCond&#x20;

```java
package hello.itemservice.repository;

import lombok.Data;

@Data
public class ItemSearchCond {

    private String itemName;
    private Integer maxPrice;

    public ItemSearchCond() {
    }

    public ItemSearchCond(String itemName, Integer maxPrice) {
        this.itemName = itemName;
        this.maxPrice = maxPrice;
    }
}
```

*   검색 조건으로 사용된다. 상품명, 최대 가격이 있다. 참고로 상품명의 일부만 포함되어도 검색이 가능해야 한다.

    (`like` 검색)
* `cond` -> `condition` 을 줄여서 사용했다.
  * 이 프로젝트에서 검색 조건은 뒤에 `Cond` 를 붙이도록 규칙을 정했다.

#### ItemUpdateDto&#x20;

```java
package hello.itemservice.repository;

import lombok.Data;

@Data
public class ItemUpdateDto {
    private String itemName;
    private Integer price;
    private Integer quantity;

    public ItemUpdateDto() {
    }

    public ItemUpdateDto(String itemName, Integer price, Integer quantity) {
        this.itemName = itemName;
        this.price = price;
        this.quantity = quantity;
    }
}
```

* 상품을 수정할 때 사용하는 객체이다.
* 단순히 데이터를 전달하는 용도로 사용되므로 DTO를 뒤에 붙였다.

> #### DTO(data transfer object)&#x20;
>
> * 데이터 전송 객체&#x20;
> * DTO 는 기능은 없고 데이터를 전달만 하는 용도로 사용되는 객체를 뜻한다.
>   * 참고로 DTO 에 기능이 있으면 안되는가? 그것은 아니다. 객체의 주 목적이 데이터를 전송하는 것이라면 DTO 라고 할 수 있다.&#x20;
> * 객체 이름에 DTO 를 꼭 붙여야 하는 것은 아니다. 대신 붙여두면 용도를 알 수 있다는 장점이 있다.&#x20;
> * 이전에 설명한 `ItemSearchCond` 도 DTO 역할을 하지만, 이 프로젝트에선 `Cond` 는 검색 조건으로 사용한다는 규칙을 정했다. 따라서 DTO 를 붙이지 않아도 된다. `ItemSearchCondDto` 이렇게 하면 너무 복잡해진다. (그리고 `Cond` 는 딱 봐도 용도를 알 수 있다)&#x20;
> * 참고로 이런 규칙은 정해진 것이 없기 때문에, 해당 프로젝트 안에서 일관성 있게 규칙을 정하면 된다.&#x20;

#### MemberItemRepository&#x20;

```java
package hello.itemservice.repository.memory;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import org.springframework.stereotype.Repository;
import org.springframework.util.ObjectUtils;

import java.util.*;
import java.util.stream.Collectors;

@Repository
public class MemoryItemRepository implements ItemRepository {

    private static final Map<Long, Item> store = new HashMap<>(); //static
    private static long sequence = 0L; //static

    @Override
    public Item save(Item item) {
        item.setId(++sequence);
        store.put(item.getId(), item);
        return item;
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        Item findItem = findById(itemId).orElseThrow();
        findItem.setItemName(updateParam.getItemName());
        findItem.setPrice(updateParam.getPrice());
        findItem.setQuantity(updateParam.getQuantity());
    }

    @Override
    public Optional<Item> findById(Long id) {
        return Optional.ofNullable(store.get(id));
    }

    @Override
    public List<Item> findAll(ItemSearchCond cond) {
        String itemName = cond.getItemName();
        Integer maxPrice = cond.getMaxPrice();
        return store.values().stream()
                .filter(item -> {
                    if (ObjectUtils.isEmpty(itemName)) {
                        return true;
                    }
                    return item.getItemName().contains(itemName);
                }).filter(item -> {
                    if (maxPrice == null) {
                        return true;
                    }
                    return item.getPrice() <= maxPrice;
                })
                .collect(Collectors.toList());
    }

    public void clearStore() {
        store.clear();
    }
}
```

* `ItemRepository` 인터페이스를 구현한 메모리 저장소이다.&#x20;
* 메모리이기 때문에, 자바를 다시 실행하면 기존에 저장된 데이터가 모두 사라진다.&#x20;
* `save`, `update`, `findById` 는 쉽게 이해할 수 있을 것이다. 참고로 `findById` 는 `Optional` 을 반환해야 하기 때문에 `Optional.ofNullable` 을 사용했다.&#x20;
* `findAll` 은 `ItemSearchCond` 이라는 검색 조건을 받아서 내부에서 데이터를 검색하는 기능을 한다. 데이터 베이스로 보면 `where` 구문을 사용해서 필요한 데이터를 필터링 하는 과정을 거치는 것이다.
  * 여기서 자바 스트림을 사용한다.
  * `itemName` 이나, `maxPrice` 가 `null` 이거나 비었으면 해당 조건을 무시한다.
  * `itemName` 이나, `maxPrice` 에 값이 있을 때만 해당 조건으로 필터링 기능을 수행한다.
* `clearStore()` 메모리에 저장된 `Item` 을 모두 삭제해서 초기화한다. 테스트 용도로만 사용한다.

### 서비스 분석&#x20;

#### ItemService 인터페이스&#x20;

```java
package hello.itemservice.service;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;

import java.util.List;
import java.util.Optional;

public interface ItemService {

    Item save(Item item);

    void update(Long itemId, ItemUpdateDto updateParam);

    Optional<Item> findById(Long id);

    List<Item> findItems(ItemSearchCond itemSearch);
}
```

* 서비스의 구현체를 쉽게 변경하기 위해 인터페이스를 사용했다.
* 참고로 서비스는 구현체를 변경할 일이 많지는 않기 때문에 사실 서비스에 인터페이스를 잘 도입하지는 않는다.
  * 여기서는 예제 설명 과정에서 구현체를 변경할 예정이어서 인터페이스를 도입했다.

#### ItemServiceV1&#x20;

```java
package hello.itemservice.service;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
@RequiredArgsConstructor
public class ItemServiceV1 implements ItemService {

    private final ItemRepository itemRepository;

    @Override
    public Item save(Item item) {
        return itemRepository.save(item);
    }

    @Override
    public void update(Long itemId, ItemUpdateDto updateParam) {
        itemRepository.update(itemId, updateParam);
    }

    @Override
    public Optional<Item> findById(Long id) {
        return itemRepository.findById(id);
    }

    @Override
    public List<Item> findItems(ItemSearchCond cond) {
        return itemRepository.findAll(cond);
    }
}
```

* `ItemServiceV1` 서비스 구현체는 대부분의 기능을 단순히 리포지토리에 위임한다.

### 컨트롤러 분석&#x20;

#### HomeController&#x20;

```javascript
package hello.itemservice.web;

import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

@Controller
@RequiredArgsConstructor
public class HomeController {

    @RequestMapping("/")
    public String home() {
        return "redirect:/items";
    }
}
```

#### ItemController&#x20;

```java
package hello.itemservice.web;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import hello.itemservice.service.ItemService;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.servlet.mvc.support.RedirectAttributes;

import java.util.List;

@Controller
@RequestMapping("/items")
@RequiredArgsConstructor
public class ItemController {

    private final ItemService itemService;

    @GetMapping
    public String items(@ModelAttribute("itemSearch") ItemSearchCond itemSearch, Model model) {
        List<Item> items = itemService.findItems(itemSearch);
        model.addAttribute("items", items);
        return "items";
    }

    @GetMapping("/{itemId}")
    public String item(@PathVariable long itemId, Model model) {
        Item item = itemService.findById(itemId).get();
        model.addAttribute("item", item);
        return "item";
    }

    @GetMapping("/add")
    public String addForm() {
        return "addForm";
    }

    @PostMapping("/add")
    public String addItem(@ModelAttribute Item item, RedirectAttributes redirectAttributes) {
        Item savedItem = itemService.save(item);
        redirectAttributes.addAttribute("itemId", savedItem.getId());
        redirectAttributes.addAttribute("status", true);
        return "redirect:/items/{itemId}";
    }

    @GetMapping("/{itemId}/edit")
    public String editForm(@PathVariable Long itemId, Model model) {
        Item item = itemService.findById(itemId).get();
        model.addAttribute("item", item);
        return "editForm";
    }

    @PostMapping("/{itemId}/edit")
    public String edit(@PathVariable Long itemId, @ModelAttribute ItemUpdateDto updateParam) {
        itemService.update(itemId, updateParam);
        return "redirect:/items/{itemId}";
    }
}
```

## 3. 프로젝트 구조 설명2 - 설정&#x20;

### 스프링 부트 설정 분석&#x20;

#### MemoryConfig&#x20;

```java
package hello.itemservice.config;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.memory.MemoryItemRepository;
import hello.itemservice.service.ItemService;
import hello.itemservice.service.ItemServiceV1;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class MemoryConfig {

    @Bean
    public ItemService itemService() {
        return new ItemServiceV1(itemRepository());
    }

    @Bean
    public ItemRepository itemRepository() {
        return new MemoryItemRepository();
    }

}
```

* `ItemServiceV1`, `MemoryItemRepository` 를 스프링 빈으로 등록하고 생성자를 통해 의존관계를 주입한다.
* 참고로 여기서는 서비스와 리포지토리는 구현체를 편리하게 변경하기 위해, 이렇게 수동으로 빈을 등록했다.
* 컨트롤러는 컴포넌트 스캔을 사용한다.

#### TestDataInit

```java
package hello.itemservice;

import hello.itemservice.domain.Item;
import hello.itemservice.repository.ItemRepository;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.boot.context.event.ApplicationReadyEvent;
import org.springframework.context.event.EventListener;

@Slf4j
@RequiredArgsConstructor
public class TestDataInit {

    private final ItemRepository itemRepository;

    /**
     * 확인용 초기 데이터 추가
     */
    @EventListener(ApplicationReadyEvent.class)
    public void initData() {
        log.info("test data init");
        itemRepository.save(new Item("itemA", 10000, 10));
        itemRepository.save(new Item("itemB", 20000, 20));
    }

}
```

* 애플리케이션을 실행할 때 초기 데이터를 저장한다.
* 리스트에서 데이터가 잘 나오는지 편리하게 확인할 용도로 사용한다.
  * 이 기능이 없으면 서버를 실행할 때 마다 데이터를 입력해야 리스트에 나타난다. \
    (메모리여서 서버를 내리면 데이터가 제거된다.)
*   `@EventListener(ApplicationReadyEvent.class)` : 스프링 컨테이너가 완전히 초기화를 다 끝내고,

    실행 준비가 되었을 때 발생하는 이벤트이다. 스프링이 이 시점에 해당 애노테이션이 붙은 `initData()` 메서드를 \
    호출해준다.

    * 참고로 이 기능 대신 `@PostConstruct` 를 사용할 경우 AOP 같은 부분이 아직 다 처리되지 않은 시점에 호출될 수 있기 때문에, 간혹 문제가 발생할 수 있다. 예를 들어서 `@Transactional` 과 관련된 AOP가 적용되지 않은 상태로 호출될 수 있다.
    * `@EventListener(ApplicationReadyEvent.class)` 는 AOP를 포함한 스프링 컨테이너가 완전히 초기화 된 이후에 호출되기 때문에 이런 문제가 발생하지 않는다.

#### ItemServiceApplication&#x20;

```java
package hello.itemservice;

import hello.itemservice.config.*;
import hello.itemservice.repository.ItemRepository;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.context.annotation.Profile;


@Import(MemoryConfig.class)
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

* `@Import(MemoryConfig.class)` : 앞서 설정한 `MemoryConfig` 를 설정 파일로 사용한다.
*   `scanBasePackages = "hello.itemservice.web"` : 여기서는 컨트롤러만 컴포넌트 스캔을 사용하고,

    나머지는 직접 수동 등록한다. 그래서 컴포넌트 스캔 경로를 `hello.itemservice.web` 하위로 지정했다.
*   `@Profile("local")` : 특정 프로필의 경우에만 해당 스프링 빈을 등록한다. 여기서는 `local` 이라는 이름의

    프로필이 사용되는 경우에만 `testDataInit` 이라는 스프링 빈을 등록한다. 이 빈은 앞서 본 것인데, 편의상 초기 \
    데이터를 만들어서 저장하는 빈이다.

### 프로필&#x20;

스프링은 로딩 시점에 `application.properties`의`spring.profiles.active` 속성을 읽어서 프로필로 사용한다.&#x20;

이  프로필은 로컬, 운영환경, 테스트 실행 등등 다양한 환경에 따라서 다른 설정을 할 때 사용하는 정보이다.&#x20;

예를 들어, 로컬에서는 로컬에 설치된 데이터베이스에 접근하고, 운영 환경에서는 운영 데이터베이스에 접근해야 한다면 서로 설정 정보가 달라야 한다. 심지어 환경에 따라서 다른 스프링 빈을 등록해야 할 수 있다. 프로필을 사용하면 이런 문제를 깔끔하게 사용할 수 있다.&#x20;

#### main 프로필&#x20;

`/src/main/resources` 하위의 `application.properties`

```properties
spring.profiles.active=local
```

*   이 위치의 `application.properties` 는 `/src/main` 하위의 자바 객체를 실행할 때 (주로 `main()` ) 동작

    하는 스프링 설정이다. `spring.profiles.active=local` 이라고 하면 스프링은 `local` 이라는 프로필로

    동작한다. 따라서 직전에 설명한 `@Profile("local")` 가 동작하고, `testDataInit` 가 스프링 빈으로 등록된다.

#### test 프로필&#x20;

`/src/test/resources` 하위의 `application.properties`

```properties
spring.profiles.active=test
```

* 이 위치의 `application.properties` 는 `/src/test` 하위의 자바 객체를 실행할 때 동작하는 스프링 설정이다.
* 주로 테스트 케이스를 실행할 때 동작한다.
* `spring.profiles.active=test`로 설정하면 스프링은 `test` 라는 프로필로 동작한다. 이 경우 직전에 설명한 `@Profile("local")` 는 프로필 정보가 맞지 않아서 동작하지 않는다. 따라서 `testDataInit` 이라는 스프링 빈도 등록되지 않고, 초기 데이터도 추가하지 않는다.

#### 프로필 정리&#x20;

* 프로필 기능을 사용해서 스프링으로 웹 애플리케이션을 로컬(`local` )에서 직접 실행할 때는 `testDataInit` 이 스프링 빈으로 등록된다. 따라서 등록한 초기화 데이터를 편리하게 확인할 수 있다.
* 초기화 데이터 덕분에 편리한 점도 있지만, 테스트 케이스를 실행할 때는 문제가 될 수 있다. 테스트에서 이런 데이터가 들어있다면 오류가 발생할 수 있다. 예를 들어서 데이터를 하나 저장하고 전체 카운트를 확인하는데 1이 아니라 `testDataInit` 때문에 데이터가 2건 추가되어서 3이 되는 것이다.
*   프로필 기능 덕분에 테스트 케이스에서는 `test` 프로필이 실행된다. 따라서 `TestDataInit` 는 스프링 빈으로

    추가되지 않고, 따라서 초기 데이터도 추가되지 않는다.

## 4. 프로젝트 구조 설명3 - 테스트&#x20;

```java
package hello.itemservice.domain;

import hello.itemservice.repository.ItemRepository;
import hello.itemservice.repository.ItemSearchCond;
import hello.itemservice.repository.ItemUpdateDto;
import hello.itemservice.repository.memory.MemoryItemRepository;
import org.junit.jupiter.api.AfterEach;
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;

import java.util.List;

import static org.assertj.core.api.Assertions.assertThat;

@SpringBootTest
class ItemRepositoryTest {

    @Autowired
    ItemRepository itemRepository;

    @AfterEach
    void afterEach() {
        //MemoryItemRepository 의 경우 제한적으로 사용
        if (itemRepository instanceof MemoryItemRepository) {
            ((MemoryItemRepository) itemRepository).clearStore();
        }
    }

    @Test
    void save() {
        //given
        Item item = new Item("itemA", 10000, 10);

        //when
        Item savedItem = itemRepository.save(item);

        //then
        Item findItem = itemRepository.findById(item.getId()).get();
        assertThat(findItem).isEqualTo(savedItem);
    }

    @Test
    void updateItem() {
        //given
        Item item = new Item("item1", 10000, 10);
        Item savedItem = itemRepository.save(item);
        Long itemId = savedItem.getId();

        //when
        ItemUpdateDto updateParam = new ItemUpdateDto("item2", 20000, 30);
        itemRepository.update(itemId, updateParam);

        //then
        Item findItem = itemRepository.findById(itemId).get();
        assertThat(findItem.getItemName()).isEqualTo(updateParam.getItemName());
        assertThat(findItem.getPrice()).isEqualTo(updateParam.getPrice());
        assertThat(findItem.getQuantity()).isEqualTo(updateParam.getQuantity());
    }

    @Test
    void findItems() {
        //given
        Item item1 = new Item("itemA-1", 10000, 10);
        Item item2 = new Item("itemA-2", 20000, 20);
        Item item3 = new Item("itemB-1", 30000, 30);

        itemRepository.save(item1);
        itemRepository.save(item2);
        itemRepository.save(item3);

        //둘 다 없음 검증
        test(null, null, item1, item2, item3);
        test("", null, item1, item2, item3);

        //itemName 검증
        test("itemA", null, item1, item2);
        test("temA", null, item1, item2);
        test("itemB", null, item3);

        //maxPrice 검증
        test(null, 10000, item1);

        //둘 다 있음 검증
        test("itemA", 10000, item1);
    }

    void test(String itemName, Integer maxPrice, Item... items) {
        List<Item> result = itemRepository.findAll(new ItemSearchCond(itemName, maxPrice));
        assertThat(result).containsExactly(items);
    }
}
```

## 5. 데이터베이스 테이블 생성&#x20;

#### 테이블 생성&#x20;

```sql
drop table if exists item CASCADE;
create table item
(
    id bigint generated by default as identity,
    item_name varchar(10),
    price integer,
    quantity integer,
    primary key (id)
);
```

### 참고 - 권장하는 식별자 선택 전략&#x20;

#### 데이터베이스 기본 키는 다음 3가지 조건을 모두 만족해야 한다.&#x20;

1. `null` 값은 허용하지 않는다.&#x20;
2. 유일해야 한다.&#x20;
3. 변해선 안 된다.&#x20;

#### 테이블의 기본 키를 선택하는 전략은 크게 2가지가 있다.&#x20;

* 자연 키 (natural key)&#x20;
  * 비즈니스에 의미가 있는 키&#x20;
  * 예 : 주민번호, 이메일, 전화번호&#x20;
* 대리 키 (surrogate key)&#x20;
  * 비지니스와 관련 없는 임의로 만들어진 키, 대체 키로도 불린다.&#x20;
  * 예 : 오라클 시퀀스, auto-increment, identity, 키생성 테이블 사용&#x20;

#### 자연 키보다 대리 키를 권장한다.&#x20;

**자연 키와 대리 키는 일장 일단이 있지만 될 수 있으면 대리 키의 사용을 권장한다.** 예를 들어 자연 키인 전화번호를 기본 키로 선택한다면 그 번호가 유일할 수는 있지만, 전화번호가 없을 수도 있고 전화번호가 변경될 수도 있다. 따라서 기본 키로 적당하지 않다. 문제는 주민등록번호처럼 그럴듯하게 보이는 값이다. 이 값은 `null` 이 아니고 유일하며 변하지 않는다는 3가지 조건을 모두 만족하는 것 같다. 하지만 현실과 비즈니스 규칙은 생각보다 쉽게 변한다. 주민등록번호 조차도 여러 가지 이유로 변경될 수 있다.

#### 비지니스 환경은 언젠가 변한다.&#x20;

나의 경험을 하나 이야기하겠다. 레거시 시스템을 유지보수할 일이 있었는데, 분석해보니 회원 테이블에 주민등록번호가 기본 키로 잡혀 있었다. 회원과 관련된 수많은 테이블에서 조인을 위해 주민등록번호를 외래 키로 가지고 있었고 심지어 자식 테이블의 자식 테이블까지 주민등록번호가 내려가 있었다. 문제는 정부 정책이 변경되면서 법적으로 주민등록번호를 저장할 수 없게 되면서 발생했다. 결국 데이터베이스 테이블은 물론이고 수많은 애플리케이션 로직을 수정 했다. 만약 데이터베이스를 처음 설계할 때부터 자연 키인 주민등록번호 대신에 비즈니스와 관련 없는 대리 키를 사용했다면 수정할 부분이 많지는 않았을 것이다.

기본 키의 조건을 현재는 물론이고 미래까지 충족하는 자연 키를 찾기는 쉽지 않다. 대리 키는 비즈니스와 무관한 임의의 값이므로 요구사항이 변경되어도 기본 키가 변경되는 일은 드물다. 대리 키를 기본 키로 사용하되 주민등록번호나 이메일처럼 자연 키의 후보가 되는 컬럼들은 필요에 따라 유니크 인덱스를 설정해서 사용하는 것을 권장한다.

참고로 JPA는 모든 엔티티에 일관된 방식으로 대리 키 사용을 권장한다.&#x20;

비즈니스 요구사항은 계속해서 변하는데 테이블은 한 번 정의하면 변경하기 어렵다. 그런면에서 외부 풍파에 쉽게 흔들리지 않는 대리 키가 일반적으로 좋은 선택이라 생각한다.
