# EXPLAIN 명령어

> 참고 링크&#x20;
>
> [https://www.postgresql.org/docs/current/using-explain.html](https://www.postgresql.org/docs/current/using-explain.html)
>
> [https://datarian.io/blog/postgresql-using-explain](https://datarian.io/blog/postgresql-using-explain)

#### 해당 글은 PostgreSQL 을 기준으로 설명한다.&#x20;

## 1. EXPLAIN 기본 사항

*
*
* 기본적으로 PostgreSQL 플래너는 특정 시스템에서 좋은 플랜을 선택하는 기능을 가지고 있다. EXPLAIN 명령어를 사용하여, 플래너가 모든 쿼리에 대해 어떤 쿼리 플랜을 만드는지 확인할 수 있다.&#x20;
* 이 실행 계획을 통해서 원하는 자료를 추출하기 위해 어떤 테이블 자료를 FULL SCAN 하는지, 인덱스 검색을 하는지를 보여준다.&#x20;
* 또한 여러 테이블이 조인이 될 경우 그 각 테이블들의 조인 알고리즘은 어떤 것을 사용할 것인지를 보여준다.
* 아래 결과는 100만개의 행을 조건없이 조회했을 때 플랜이다.&#x20;

```sql
EXPLAIN ANALYZE SELECT * from TEST_EXPLAIN;
```

<figure><img src="../../.gitbook/assets/image (2) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

## 2. EXPLAIN 항목

### 2-1. COST&#x20;

* 전체 COST 계산을 위해서 테이블의 ROWS 수와 BLOCK 수를 확인한다.&#x20;
*

## 3. EXPLAIN ANALYZE

## 4. EXPLAIN 추가 옵션

## 5. 주의 사항&#x20;
