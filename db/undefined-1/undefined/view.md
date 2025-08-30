# View

## 뷰(View) 소개&#x20;

<pre class="language-sql"><code class="lang-sql">-- 지난 시간에 만든 복잡하고 유용한 쿼리
SELECT
    p.category,
    COUNT(*) AS total_orders,
    SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
    SUM(CASE WHEN o.status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
    SUM(CASE WHEN o.status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
    orders o
JOIN
    products p ON o.product_id = p.product_id
GROUP BY
<strong>    p.category;
</strong></code></pre>

만약 위 쿼리가 매우 유용해서, 우리 쇼핑몰의 재무팀, 마케팅팀, 운영팀에서 매일 아침 확인해야 하는 핵심 지표라고 가정해보자. 여기서 오늘의 문제 상황이 발생한다.&#x20;

* **복잡성** : 이 쿼리가 너무 길고 복잡하다. SQL 에 익숙하지 않은 직원이 이 쿼리를 매일 실수 없이 정확하게 입력하기란 거의 불가능에 가깝다.&#x20;
* **재사용성** : 여러 팀의 여러 사람이 이 쿼리를 사용하면, 각자 이 긴 쿼리문을 어딘가에 저장해두고 복사-붙여넣기를 해야할 것이다. 만약 보고서 요구사항이 변경된다면, 모든 사람들이 각자 저장해 둔 쿼리를 수정해야 하는 문제가 발생한다.&#x20;
* **보안** : 이 쿼리를 실행하려면, 사용자는 원본 테이블인 orders, products 에 직접 접근할 수 있는 권한이 있어야 한다. 하지만, 운영팀 직원에게는 고객의 개인정보나 상품의 원가 같은 민감한 정보를 알려주고 싶지 않다. 딱 이 요약된 보고서 내용만 보게 하고 싶다.&#x20;

이 모든 문제를 해결할 도구가 바로 뷰(View) 이다.&#x20;

### 뷰(View) 의 개념&#x20;

뷰는 실제 데이터를 가지고 있지 않은 '가상의 테이블' 이다. 그 실체는 데이터베이스에 이름과 함께 저장된 하나의 SELECT 쿼리문이다.&#x20;

우리는 복잡한 쿼리를 실행할 필요 없이, `SELECT * FROM 뷰_테이블;` 이라는 간단한 명령만으로 그 복잡한 쿼리의 결과를 얻을 수 있다.&#x20;

### 뷰(View) 의 동작 원리&#x20;

사용자가 뷰\_테이블 이라는 뷰를 조회하면, 데이터베이스 내부에서는 다음과 같은 일이 일어난다.&#x20;

1. 사용자가 `SELECT * FROM 뷰_테이블;` 라는 쿼리를 실행한다.&#x20;
2. 데이터베이스는 뷰\_테이블 라는 이름의 실제 테이블을 찾지 않는다. 대신, 이 뷰에 정의된 원래의 긴 SELECT 쿼리문을 찾아낸다.&#x20;
3. 데이터베이스는 그 저장된 SELECT 쿼리문을 그 순간에 실행한다.&#x20;
4. 실행 결과가 사용자에게 반환된다. 사용자는 마치 그냥 테이블을 조회한 것처럼 느낀다.&#x20;

여기서 중요한 점은 뷰 테이블은 데이터를 저장하지 않기 때문에, **뷰의 데이터는 항상 최신 상태를 유지한다.**&#x20;

### 뷰(View) 의 장점&#x20;

1. **편리성** : 복잡한 쿼리를 단순화하여 쉽게 재사용할 수 있다.&#x20;
2. **보안성** : 사용자에게 원본 테이블에 대한 접근 권한을 주지 않고, 뷰를 통해서만 제한된 데이터에 접근하도록 허용할 수 있다.&#x20;
3. **논리적 독립성** : 만약 나중에 원본 테이블의 구조가 일부 변경되더라도, 뷰의 정의만 수정하면 뷰를 사용하는 응용 프로그램은 코드를 바꿀 필요가 없을 수도 있다. 뷰가 중간에서 변화를 흡수하는 '추상화 계층' 역할을 하기 때문이다.&#x20;

## 뷰 생성, 조회, 수정, 삭제&#x20;

```sql
// 뷰 생성 
CREATE VIEW v_category_order_status AS
SELECT
    p.category,
    COUNT(*) AS total_orders,
    SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
    SUM(CASE WHEN o.status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
    SUM(CASE WHEN o.status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
    orders o
JOIN
    products p ON o.product_id = p.product_id
GROUP BY
    p.category;

// 뷰 조회 
SELECT * FROM v_category_order_status;

// 뷰 수정 
ALTER VIEW v_category_order_status AS
SELECT
    p.category,
    SUM(p.price * o.quantity) AS total_sales, -- 매출액 컬럼 추가!
    COUNT(*) AS total_orders,
    SUM(CASE WHEN o.status = 'COMPLETED' THEN 1 ELSE 0 END) AS completed_count,
    SUM(CASE WHEN o.status = 'SHIPPED' THEN 1 ELSE 0 END) AS shipped_count,
    SUM(CASE WHEN o.status = 'PENDING' THEN 1 ELSE 0 END) AS pending_count
FROM
    orders o
JOIN
    products p ON o.product_id = p.product_id
GROUP BY
    p.category;

// 뷰 삭제 
DROP VIEW v_category_order_status;
```

## 뷰의 단점 및 주의사항&#x20;

### 1. 성능 문제&#x20;

`SELECT * FROM my_view;` 라는 간단하 쿼리 뒤에는 어마무시한 무거운 실제 쿼리가 숨어 있을 수 있다.&#x20;

* 착각 : 사용자는 자신이 매우 가벼운 쿼리를 실행한다고 착각하지만, 실제로는 시스템에 큰 부하를 주는 쿼리를 날리고 있을 수 있다.&#x20;
* 최적화의 한계 : 데이터베이스의 쿼리 옵티마이저는 매우 똑똑하지만, 뷰의 구조가 너무 복잡해지면 최적의 실행 계획을 찾아내는 데 어려움을 겪을 수 있다. 특히 **뷰 위에 또 다른 뷰를 쌓는 방식(Nested Views) 은 성능 저하의 주요 원인이 되므로 가급적 피해야만 한다!**

### **2. 업데이트 제약**&#x20;

뷰를 테이블처럼 다룰 수 있다고 했지만, 그것은 주로 조회(읽기) 에 국한된 이야기이다.&#x20;

* 수정 불가(대부분의 경우) : `JOIN`, 집계 함수(`SUM`, `COUNT` 등), `GROUP BY`, `DISTINCT` 등을 사용한 복잡한 뷰는 일반적으로 `INSERT`, `UPDATE`, `DELETE` 가 불가능하다. 왜냐하면 데이터베이스 입장에서, 뷰의 한 행을 수정하는 것이 원본 테이블들의 데이터를 어떻게 바꿔야 하는지에 대한 명확한 규칙을 특정할 수 없기 때문이다.
* 수정 가능 : 뷰가 오직 하나의 기본 테이블만을 참조하고, 뷰의 모든 컬럼이 기본 테이블의 실제 컬럼을 직접 참조하는 경우에는 뷰에 데이터를 추가/수정할 수 있다. 이 경우 원본 테이블의 값이 변경된다.&#x20;
* **실무 규칙 : "뷰는 기본적으로 조회용이다" 라고 생각하는 것이 가장 안전하고 일반적인 원칙이다. 데이터를 수정해야 한다면, 뷰가 아닌 원본 테이블에 직접 수행해야 한다.**&#x20;
