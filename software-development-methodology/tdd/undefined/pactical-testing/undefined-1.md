# 단위테스트

#### 요구 사항&#x20;

* 주문 목록에 음료 추가 / 삭제 기능
* 주문 목록 전체 지우기
* 주문 목록 총 금액 계산하기
* 주문 생성하기

***

## 1. 수동 테스트 vs 자동화된 테스트&#x20;

### 단위 테스트&#x20;

* **작은 코드 단위를 독립적으로 검증하는 테스트**
  * 작은 : 클래스 or 메서드&#x20;
  * 독립적 : 외부 상황에 의존하는 테스트가 아닌, 클래스 or 메서드만 검증
* **검증 속도가 빠르고, 안정적이다.**

### JUnit 5

* 단위 테스트를 위한 테스트 **프레임워크**
* XUnit - 켄트 백
  * XUnit 으로 시작해서 자바에서는 JUnit 으로 발전되었다.

### AssertJ

* 테스트 코드 작성을 원활하게 돕는 테스트 **라이브러리**&#x20;
* 풍부한 API, 메서드 체이닝 지원&#x20;

### 결론&#x20;

* 일관성이 부족한 개인이 테스트하는 것은 불확실성을 유발한다.&#x20;
* 테스트 도구들을 사용해서 일관적인 자동화된 테스트를 하자!&#x20;

```java
package sample.cafekiosk;

import org.junit.jupiter.api.Test;
import sample.cafekiosk.unit.CafeKiosk;
import sample.cafekiosk.unit.beverage.Americano;
import sample.cafekiosk.unit.beverage.Latte;

import static org.assertj.core.api.Assertions.*;

class CafekioskApplicationTests {

    @Test
    void add_manual_test() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       cafeKiosk.add(new Americano());

       System.out.println(">> 음료 수 : " + cafeKiosk.getBeverages().size());
       System.out.println(">> 음료 이름 : " + cafeKiosk.getBeverages().get(0).getName());
    }

    @Test
    void add() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       cafeKiosk.add(new Americano());

//     assertThat(cafeKiosk.getBeverages().size()).isEqualTo(1);
       assertThat(cafeKiosk.getBeverages()).hasSize(1);
       assertThat(cafeKiosk.getBeverages().get(0).getName()).isEqualTo("아메리카노");
    }

    @Test
    void remove() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();

       cafeKiosk.add(americano);
       assertThat(cafeKiosk.getBeverages()).hasSize(1);

       cafeKiosk.remove(americano);
       assertThat(cafeKiosk.getBeverages()).isEmpty();
    }

    @Test
    void clear() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       cafeKiosk.add(new Americano());
       cafeKiosk.add(new Latte());
       assertThat(cafeKiosk.getBeverages()).hasSize(2);

       cafeKiosk.clear();
       assertThat(cafeKiosk.getBeverages()).isEmpty();
    }

}
```

***

## 2. 테스트 케이스 세분화하기&#x20;

#### 요구사항&#x20;

* 한 종류의 음료 여러 잔을 한 번에 담는 기능&#x20;

#### 질문하기 : 암묵적이거나 아직 드러나지 않은 요구사항이 있는가?

### 테스트 케이스 세분화하기&#x20;

* 해피 케이스&#x20;
* 예외 케이스
* **경계값 테스트가 매우 중요하다.**&#x20;
  * 범위(이상, 이하, 초과, 미만), 구간, 날짜 등
  * 경계값이나 경계값 주변값을 테스트 검증값으로 설정해서 테스트하는 것이 중요하다.&#x20;
  * 예를 들어, 3이상 인지 검증해라.
    * 해피 케이스 : 3
    * 예외 케이스 : 2

```java
package sample.cafekiosk;

import org.junit.jupiter.api.Test;
import sample.cafekiosk.unit.CafeKiosk;
import sample.cafekiosk.unit.beverage.Americano;
import sample.cafekiosk.unit.beverage.Latte;

import static org.assertj.core.api.Assertions.*;

class CafekioskApplicationTests {

    ...

    @Test
    void add() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       cafeKiosk.add(new Americano());

//     assertThat(cafeKiosk.getBeverages().size()).isEqualTo(1);
       assertThat(cafeKiosk.getBeverages()).hasSize(1);
       assertThat(cafeKiosk.getBeverages().get(0).getName()).isEqualTo("아메리카노");
    }

    @Test
    void addSeveralBeverages() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();

       cafeKiosk.add(americano, 2);

       assertThat(cafeKiosk.getBeverages()).hasSize(2);
       assertThat(cafeKiosk.getBeverages().get(0)).isEqualTo(americano);
       assertThat(cafeKiosk.getBeverages().get(1)).isEqualTo(americano);
    }

    @Test
    void addZeroBeverages() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();

       assertThatThrownBy(() -> cafeKiosk.add(americano, 0))
             .isInstanceOf(IllegalArgumentException.class)
             .hasMessage("음료는 1잔 이상 주문하실 수 있습니다.");
    }

    ...

}
```

***

## 3. 테스트하기 어려운 영역을 분리하기&#x20;

#### 요구사항&#x20;

* 가게 운영 시간(10:00 \~ 22:00) 외에는 주문을 생성할 수 없다.&#x20;

### 테스트하기 어려운 영역&#x20;

* **관측할 때마다 다른 값에 의존하는 코드**&#x20;
  * 현재 날짜/시간, 랜덤 값, 전역 변수/함수, 사용자 입력 등..&#x20;
* **우리가 작성한 코드가 외부 세계에 영향을 주는 코드**&#x20;
  * _**외부 세계 자체가 우리가 테스트하기 어렵다.**_
  * 표준 출력, 메시지 발송, 데이터베이스 기록하기 등..&#x20;

### 테스트하기 쉬운 영역 (순수 함수)&#x20;

* 같은 입력에 항상 같은 결과&#x20;
* 외부 세상과 단절된 형태
* 테스트하기 쉬운 코드

### 테스트하기 여러운 영역 구분하고 분리하기&#x20;

* 테스트 하기 어려운 부분은 외부(매개변수 등등..) 로부터 받는 구조로 변경하면 된다.
* **테스트 하고자 하는 부분이 무엇인가?**&#x20;
  * **영업 시간**&#x20;
* 내부 로직을 외부로 분리할수록 테스트 가능한 코드는 많아진다.&#x20;

```java
package sample.cafekiosk;

import org.junit.jupiter.api.Test;
import sample.cafekiosk.unit.CafeKiosk;
import sample.cafekiosk.unit.beverage.Americano;
import sample.cafekiosk.unit.beverage.Latte;
import sample.cafekiosk.unit.order.Order;

import java.time.LocalDateTime;

import static org.assertj.core.api.Assertions.*;

class CafekioskApplicationTests {

    ...

    @Test
    void createOrder() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();
       cafeKiosk.add(americano);

       Order order = cafeKiosk.createOrder();

       assertThat(order.getBaverages()).hasSize(1);
       assertThat(order.getBaverages().get(0)).isEqualTo(americano);

    }

    @Test
    void createOrderWithCurrentTime() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();
       cafeKiosk.add(americano);

       Order order = cafeKiosk.createOrder(LocalDateTime.of(2025, 9, 19, 10, 0));

       assertThat(order.getBaverages()).hasSize(1);
       assertThat(order.getBaverages().get(0)).isEqualTo(americano);

    }

    @Test
    void createOrderOutsideOpenTime() {
       CafeKiosk cafeKiosk = new CafeKiosk();
       Americano americano = new Americano();
       cafeKiosk.add(americano);

       assertThatThrownBy(() -> cafeKiosk.createOrder(LocalDateTime.of(2025, 9, 19, 9, 59)))
             .isInstanceOf(IllegalArgumentException.class)
             .hasMessage("주문 시간이 아닙니다. 관리자에게 문의하세요.");

    }

}
```

***

## 4. 키워드 정리&#x20;

* 단위 테스트&#x20;
* 수동 테스트, 자동화 테스트&#x20;
* JUnit 5, AssertJ&#x20;
* 해피 케이스, 예외 케이스&#x20;
* 경계값 테스트&#x20;
* 테스트하기 쉬운/어려운 영역(순수 함수)
* lombok (사용 가이드)&#x20;
  * `@Date`, `@Setter`, `@AllArgsConstructor` 지양&#x20;
  * 양방향 연관관계 시 `@ToString` 순환 참조 문제
