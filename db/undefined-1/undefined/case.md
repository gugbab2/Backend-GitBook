# CASE 문

## CASE 문 기본1&#x20;

오늘의 문제 상황을 보자&#x20;

**"상품 목록을 조회하는데, 그냥 가격만 보여주지 말고, 가격대에 따라 '고가', '중가', '저가'와 같은 알아보기 쉬운 등급을 옆에 함께 표시하고 싶다."**

* 10만원 이상은 고가&#x20;
* 3만원 이상, 10만원 미만은 중가&#x20;
* 3만원 미만은 저가&#x20;

### CASE 문의 두가지 종류&#x20;

#### ☑️ 단순 `CASE` 문 (Simple CASE Expression)

단순 `CASE` 문은 특정 하나의 컬럼이나 표현식이 값에 따라서 결과를 다르게 하고 싶을 때 사용한다.&#x20;

```sql
CASE 비교대상_컬럼_또는_표현식
    WHEN 값1 THEN 결과1
    WHEN 값2 THEN 결과2
    ...
    ELSE 그_외의_경우_결과
END
```

실행 순서 : 단순 `CASE` 문도 위에서 아래 순서대로 조건을 평가하며, **가장 먼저 일치하는 `WHEN` 절을 만난 순간 그 `THEN` 의 결과를 반환하고 즉시 평가를 종료한다.**&#x20;

단순 CASE 문 예제: 주문 상태(status)를 한글로 표시하기

```sql
select 
    order_id, 
    user_id, 
    product_id, 
    quantity, 
    status, 
    case status
	when 'PENDIN' then '주문 대기'
        WHEN 'COMPLETED' THEN '결제 완료'
	WHEN 'SHIPPED' THEN '배송'
	WHEN 'CANCELLED' THEN '주문 취소'
        else '알 수 없음'
    end as status_korean
from 
    orders;
```

#### ☑️ 검색 `CASE` 문(Searched CASE Expression)&#x20;

검색 `CASE` 문은 **단순 `CASE` 문처럼 하나의 특정 값을 비교하는 대신, 각 `WHEN` 절에 독립적인 조건식을 사용하여 복잡한 논리를 구현할 때 사용한다.** 앞서 만난 문제 상황처럼 '가격이 얼마 이상', '날짜가 언제 이전'과 같은 범위 조건이나 여러 컬럼을 조합한 복합적인 조건이 필요할 때 주로 사용된다.

```sql
CASE 
    WHEN 값1 THEN 결과1
    WHEN 값2 THEN 결과2
    ...
    ELSE 그_외의_경우_결과
END
```

실행 순서 : 검색 `CASE` 문도 위에서 아래 순서대로 조건을 평가하며, **가장 먼저 참이 되는 `WHEN` 절을 만나는 순간 그 `THEN` 의 결과를 반환하고 즉시 평가를 종료한다.**&#x20;

검색 CASE 문 예제: 상품 가격에 따라 등급 표시하기

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 100000 THEN '고가'
        WHEN price >= 30000 THEN '중가'
        ELSE '저가'
    END AS price_label
FROM
    products;
```

## CASE 문 기본2&#x20;

### 검색 CASE 문 사용 시 주의사항: WHEN 절의 순서

앞서 `CASE` 문은 위에서 아래로 순차적으로 평가하며, **가장 먼저 참이 되는 조건을 만나는 순간 실행을 멈춘다**고 강조했다. 이 점이 검색 `CASE` 문에서는 특히 중요하다. 만약 조건을 잘못 배치하면 예상과 다른 결과가 나올 수 있다.

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 30000 THEN '중가'    -- 이 조건이 먼저 평가된다
        WHEN price >= 100000 THEN '고가'
        ELSE '저가'
    END AS price_label
FROM
    products;
```

이처럼 `WHEN price >= 100000` 인 상품도 `price >= 30000` 조건을 먼저 만족시켜 '중가'로 잘못 분류될 것이다. \
따라서 검색 `CASE` 문을 사용할 때는 조건의 순서를 신중하게 고려해야 한다. **더 포괄적인(범위가 넓은) 조건보다는 더 구체적인(범위가 좁은) 조건을 먼저 배치하는 것이 일반적이다.**

### **CASE 문과 사용 위치**&#x20;

`CASE` 문은 `SELECT` 절 외에도 `ORDER BY`, `GROUP BY`, `WHERE` 절 등 다양한 SQL 구문과 함께 사용될 수 있다.

예를 들어, 상품을 '고가', '중가', '저가' 순서로 정렬하고 싶다면 `ORDER BY` 절에 `CASE` 문을 사용할 수 있다.

```sql
SELECT
    name,
    price,
    CASE
        WHEN price >= 100000 THEN '고가'
        WHEN price >= 30000 THEN '중가'
        ELSE '저가'
    END AS price_label
FROM
    products;
ORDER BY
    CASE
        WHEN price >= 100000 THEN 1 -- 고가: 1
        WHEN price >= 30000 THEN 2 -- 중가: 2
        ELSE 3 -- 저가: 3
    END ASC, -- 숫자가 작은 순서대로 정렬
    price DESC; -- 같은 등급 내에서는 가격 내림차순
```

## CASE 문 - 그룹핑&#x20;

### 데이터 분류 및 그룹핑&#x20;

`CASE` 문의 진정한 힘은, 이렇게 동적으로 만들어낸 값을 **다른 SQL 구문과 결합**할 때 드러난다. 오늘은 `CASE` 문과 `GROUP BY` 를 함께 사용하여, 데이터를 우리가 원하는 기준으로 분류하고, 분류된 그룹별로 통계를 내는 실용적인 기술을 배워보겠다.

오늘의 문제 상황이다.&#x20;

**"고객들을 출생 연대에 따라 '1990년대생', '1980년대생', '그 이전 출생'으로 분류하고, 각 그룹에 고객이 총 몇 명씩 있는지 알고 싶다."**

```sql
select 
    case 
	when year(birth_date) >= 1990 then '1990년대생'
        when year(birth_date) >= 1980 then '1980년대생'
        else '그 이전 출생'
    end as birth_decade, 
    count(*) as customer_count
from users
group by birth_decade	-- 원칙적으로는 안되지만, MySQL(다른 RDB 도 해당) 지원 
;
```

SQL 표준의 논리적 실행 순서에 따르면 `GROUP BY` 절이 `SELECT` 절보다 먼저 처리된다. 따라서 원칙적으로는 `SELECT` 절에서 정의한 별칭(`birth_decade`)을 `GROUP BY` 절에서 사용할 수 없다. 하지만 MySQL을 포함한 최신 버전의 많은 데이터베이스들은 사용자 편의를 위해 이러한 별칭 사용을 예외적으로 허용한다.

GROUP BY\` 절에 `CASE` 문 전체를 다시 쓸 필요 없이, `SELECT` 절에서 `AS` 로 지정한 별칭(`birth_decade`)을 바로 사용했다.

이는 쿼리를 훨씬 간결하고 읽기 쉽게 만들어준다.

이처럼 `CASE` 문을 이용해 기존에 없던 새로운 분류 기준을 동적으로 생성하고, 이를 `GROUP BY` 와 결합하는 패턴은 실무 데이터 분석과 리포팅에서 정말로 흔하게 사용되는 핵심 기술 중 하나다.

## CASE 문 - 조건부 집계1, 2&#x20;

오늘의 문제 상황은 다음과 같다. 엑셀의 '피벗 테이블' 기능과 유사한 보고서를 SQL로 직접 만들어보는 것이다.

**"하나의 쿼리로, 전체 주문 건수와 함께 '결제 완료(COMPLETED)', '배송(SHIPPED)', '주문 대기(PENDING)' 상태의 주문이 각각 몇 건인지 별도의 컬럼으로 나누어 보고 싶다."**

### **CASE 를 품은 집계 함수**

#### **패턴1 : `COUNT(CASE ...)`**

**`COUNT` 함수는  `NULL` 이 아닌 모든 값을 센다는 특징을 이용한다.**&#x20;

```sql
COUNT(CASE WHEN status = 'COMPLETED' THEN 1 END)
```

#### 패턴2 : `SUM(CASE ...)`

`SUM` 함수는 숫자들의 합계를 구한다.

```sql
SUM(CASE WHEN status = 'COMPLETED' THEN 1 ELSE 0 END)
```

두 패턴 모두 동일한 결과를 내며, 실무에서 널리 쓰인다. 여기서는 각 상태의 합계를 구한다는 의미를 더 명확히 보여주는 `SUM` 패턴을 사용해 보겠다.

### 실습 1: 전체 주문 상태 요약하기

먼저 `orders` 테이블 전체를 대상으로, 각 상태별 주문 건수를 집계해 보자. `GROUP BY` 는 아직 필요 없다.

```sql
elect 
    p.category,
    count(*) as total_orders, 
    sum(case when status = 'COMPLETED' then 1 else 0 end) as completed_count, 
    sum(case when status = 'SHIPPED' then 1 else 0 end) as shipped_count, 
    sum(case when status = 'PENDING' then 1 else 0 end) as pending_count
from 
    orders
;
```

### 실습 2: `GROUP BY` 와 함께 사용하기 (피벗 테이블)

조건부 집계는 `GROUP BY` 와 함께 사용할 때 그 진정한 힘을 발휘한다.

이번에는 한 단계 더 나아가 **"상품 카테고리별로, 상태별 주문 건수를 집계하라"**&#xB294; 보고서를 만들어 보자.

이를 위해서는 `orders` 테이블과 `products` 테이블을 `JOIN` 하여 `category` 정보를 가져온 뒤, `p.category` 를 기준으로 `GROUP BY` 해주면 된다.

```sql
select 
    p.category,
    count(*) as total_orders, 
    sum(case when status = 'COMPLETED' then 1 else 0 end) as completed_count, 
    sum(case when status = 'SHIPPED' then 1 else 0 end) as shipped_count, 
    sum(case when status = 'PENDING' then 1 else 0 end) as pending_count
from 
    orders o
join products p on p.product_id = o.product_id
group by p.category;
```
