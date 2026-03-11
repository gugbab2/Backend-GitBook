# 공식 문서

## Django 6.0 학습 링크 (우선순위순)

AI가 짠 코드를 검증하는 데 필요한 "데이터 레이어 동작 원리" 중심으로 정리.

***

### 1순위

#### 1. Optimize database access

https://docs.djangoproject.com/en/6.0/topics/db/optimization/

AI가 짠 Django 코드는 대부분 "돌아가는 코드"이지 "효율적인 코드"가 아니다. 이 문서는 `select_related`와 `prefetch_related`의 차이, 쿼리셋이 실제로 평가되는 시점, 불필요한 DB 호출을 줄이는 패턴을 다룬다. AI가 생성한 ORM 코드를 머지하기 전에 "이게 N+1 쿼리를 만드는가?"를 판단하는 기준이 여기서 나온다.

#### 2. Making queries

https://docs.djangoproject.com/en/6.0/topics/db/queries/

ORM 코드가 어떤 SQL로 변환되는지의 기본 규칙. `filter`, `exclude`, `Q` 객체, ForeignKey 접근 시 발생하는 쿼리 패턴을 이해해야 한다. 이걸 모르면 AI가 짠 `object.related_set.all()` 같은 코드가 루프 안에서 매번 쿼리를 날리는지 아닌지 구분할 수 없다.

#### 3. Aggregation

https://docs.djangoproject.com/en/6.0/topics/db/aggregation/

`annotate`, `aggregate`, `F` 객체를 사용한 집계 쿼리. 매출 합산, 사용자별 액션 수 집계 같은 비즈니스 데이터를 다룰 때 필수다. AI가 작성한 집계 로직이 Python 레벨에서 반복문으로 처리하는지, DB 레벨에서 한 번에 처리하는지를 판단하려면 이 개념을 알아야 한다.

#### 4. Query Expressions

https://docs.djangoproject.com/en/6.0/ref/models/expressions/

`Subquery`, `OuterRef`, `Window`, `Value`, `Case` 등 고급 쿼리 표현식. 복잡한 비즈니스 로직을 ORM으로 표현할 수 있는지, 아니면 Raw SQL로 가야 하는지를 판단하는 기준이 된다. 특히 `Window`는 3단계에서 다룰 윈도우 함수 분석을 ORM 레벨에서 구현할 때 직접 쓰인다.

#### 5. Database Functions

https://docs.djangoproject.com/en/6.0/ref/models/database-functions/

`Coalesce`, `Cast`, `TruncMonth`, `ExtractWeek` 등 DB 함수. 날짜별 집계, 코호트 분석, 널 처리 같은 작업을 ORM에서 처리할 때 필요하다. 이걸 모르면 AI가 Python 코드로 날짜를 파싱하고 반복문으로 집계하는 비효율적인 코드를 짜도 잡아낼 수 없다.

***

### 2순위

#### 6. Transactions

https://docs.djangoproject.com/en/6.0/topics/db/transactions/

`atomic()` 블록의 동작 방식, 중첩 트랜잭션, savepoint. 결제 처리, 포인트 차감 같이 "중간에 실패하면 전부 롤백해야 하는" 로직에서 데이터 정합성을 보장하는 방법이다. AI가 짠 코드에서 가장 놓치기 쉬운 영역이 바로 트랜잭션 경계 설정이다.

#### 7. Indexes

https://docs.djangoproject.com/en/6.0/ref/models/indexes/

어떤 필드에 인덱스를 걸어야 하는지, 복합 인덱스의 컬럼 순서가 왜 중요한지. 트래픽이 늘어나면 가장 먼저 터지는 곳이 인덱스 없는 쿼리다. AI는 모델 필드를 정의할 때 `db_index=True`를 빠뜨리는 경우가 많고, 복합 인덱스가 필요한 상황을 거의 제안하지 않는다.

#### 8. Raw SQL

https://docs.djangoproject.com/en/6.0/topics/db/sql/

ORM으로 표현하기 어려운 복잡한 분석 쿼리를 직접 실행하는 방법. 코호트 분석이나 퍼널 분석처럼 여러 테이블을 복잡하게 조인해야 할 때 ORM의 한계를 느끼게 되는데, 그때 Raw SQL로 전환하는 방법과 보안(SQL Injection 방지) 규칙을 다룬다.

#### 9. Conditional Expressions

https://docs.djangoproject.com/en/6.0/ref/models/conditional-expressions/

`Case`, `When`을 사용한 조건부 집계. "결제 완료 유저 수 vs 미완료 유저 수"처럼 조건별로 데이터를 나눠서 한 쿼리로 집계할 때 쓴다. 이걸 모르면 조건별로 쿼리를 여러 번 날리거나 Python에서 후처리하는 비효율적인 패턴이 된다.

#### 10. PostgreSQL specific features

https://docs.djangoproject.com/en/6.0/ref/contrib/postgres/

`ArrayField`, `JSONField`, `SearchVector`(전문 검색), `ArrayAgg`, `StringAgg` 등 PostgreSQL 전용 기능. 현재 회사가 PostgreSQL을 쓴다면, 이 기능들을 알아야 AI가 범용적으로 짠 코드를 PostgreSQL에 최적화된 코드로 개선할 수 있다.

***

### 참고 (필요할 때)

#### QuerySet method reference

https://docs.djangoproject.com/en/6.0/ref/models/querysets/

`values`, `defer`, `only`, `bulk_create`, `update_or_create` 등 전체 메서드 목록. 암기할 필요 없이 사전처럼 찾아보는 용도. AI가 생소한 메서드를 사용했을 때 동작을 확인하는 데 쓴다.

#### Managers

https://docs.djangoproject.com/en/6.0/topics/db/managers/

커스텀 매니저와 커스텀 쿼리셋 정의. 자주 쓰는 쿼리 패턴을 재사용 가능하게 만들 때 필요하다. 코드베이스가 커질수록 중요해지지만 초기에는 우선순위가 낮다.

#### Multiple databases

https://docs.djangoproject.com/en/6.0/topics/db/multi-db/

읽기/쓰기 DB 분리, 분석용 레플리카 연결 등. 트래픽이 커져서 DB를 분리해야 하거나, 분석 쿼리를 프로덕션 DB에 직접 날리지 않으려 할 때 필요하다.

#### Custom lookups

https://docs.djangoproject.com/en/6.0/howto/custom-lookups/

ORM의 기본 lookup(`__exact`, `__contains` 등)으로 표현할 수 없는 DB 연산을 커스텀으로 만드는 방법. 특수한 요구사항이 생겼을 때 찾아보면 된다.
