# 문자열 vs DATE

> **\[참고자료]**\
> **Indexes** : [https://www.postgresql.org/docs/current/indexes.html](https://www.postgresql.org/docs/current/indexes.html)\
> **Index Types** : [https://www.postgresql.org/docs/current/indexes-types.html](https://www.postgresql.org/docs/current/indexes-types.html)\
> **Query Optimization** : [https://www.postgresql.org/docs/current/runtime-config-query.html](https://www.postgresql.org/docs/current/runtime-config-query.html)\
> **Using EXPLAIN** : [https://www.postgresql.org/docs/current/using-explain.html](https://www.postgresql.org/docs/current/using-explain.html)

## 1. 문자 타입(Character Types)&#x20;

### 문자 타입 종류&#x20;

* varchar(n) : 길이 제한이 있는 가변길이 문자 타입
* char(n), bpchar(n) : 고정 길이 문자 타입, 남은 공간은 공백으로 채움&#x20;
* text : 길이 제한이 없는 가변 길이 문자 타입&#x20;

### SQL 표준과 문자 타입&#x20;

* SQL 표준에는 두 가지 주요 문자 타입이 정의되어 있습니다.&#x20;
  * 'character verying(n)' : 길이 제한이 있는 가변 길이 문자열 타입
  * 'character(n)' : 고정 길이 문자열 타입&#x20;
* 둘 다 최대 n개의 문자를 저장할 수 있다. 만약 n 보다 긴 문자열을 저장하려고 하면 에러가 발생하며, 모든 초과 문자들이 공백인 경우, 문자열은 최대 길이로 잘린다. 이는 SQL 표준에 따른 동작이다.&#x20;

### PostgreSQL 에서의 추가 문자 타입

* text : SQL 표준에는 없지만 PostgreSQL 에서 많이 사용되는 타입이다. 이 타입은 길이 제한이 없으며 대부분의 내장 문자열 함수가 이 타입을 사용한다.&#x20;

### 성능 팁&#x20;

* 성능 차이 : PostgreSQL 에서는 세 가지 타입 간의 성능 차이는 거의 없다. 하지만, 'bpchar(n)' 타입은 공간 소모가 크고, 길이 검증과정이 필요하므로 가장 느릴 수 있다.&#x20;
* 대부분의 상황에서는 'text' 또는 'varchar' 를 사용하는 것이 좋다.&#x20;

## 2. 날짜와 시간 타입&#x20;

### 날짜와 시간 타입 종류&#x20;

* `date` : 날짜 타입을 저장하며, `YYYY-MM-DD` 형식으로 저장된다.&#x20;
* `time` : 시간 정보를 저장하며, 기본적으로 시간대 정보를 포함하지 않는다. `HH:MI:SS` 형식으로 저장된다.&#x20;
* `time with time zone` : 시간 정보를 저장하며, 시간 정보와 함께 시간대 정보를 저장한다.&#x20;
* `timestamp` : 날짜와 시간을 함께 저장하며, 시간대 정보를 포함하지 않는다. &#x20;
* `timestamp with time zone` : 날짜와 시간을 함께 저장하며, 시간대 정보를 포함한다.&#x20;
* `interval` : 기간이나 시간 간격을 나타내는 데 사용된다.&#x20;

## 3. 제대로 된 날짜 비교는?&#x20;

#### 결론부터 말하자면 날짜 컬럼의 타입은 문자열 보다는 날짜(date, timestamp) 타입이 권장된다.

\-> 날짜 타입과, 문자열 타입의 장단점은 아래와 같다.&#x20;

### 날짜 타입을 사용할 경우

#### 장점&#x20;

* 정확한 날짜/시간 처리 : 날짜 간의 차이를 계산하거나 날짜를 더하거나 빼는 연산이 쉽게 가능하다.&#x20;
* 내장 함수 지원 : PostgreSQL 은 날짜 타입에 대해 다양한 내장 함수 및 연산자를 지원한다.&#x20;
* 일관된 데이터 형식 : 날짜 타입은 표준화 된 형식으로 저장되므로 데이터의 일관성을 보장한다. 형식이 고정되어 있기 때문에, 날짜 형식에 대한 유효성 검사가 필요 없다.&#x20;
* 자동 정렬 : 날짜 데이터는 시간 순서대로 정렬하기가 편하다. `order by` 를 사용하면 날짜 타입이 적절한 순서로 저장된다.&#x20;

#### 단점&#x20;

* 유연성 부족 : 특수한 날짜 형식이 필요한 경우, 기본 날짜 타입은 이를 직접적으로 지원하지 않는다. 이런 경우 추가적인 포멧팅 작업이 필요하다.&#x20;
* 시간대 고려 필요 :  `timestamp with time zone` 을 사용할 때는 시간대 관리를 신경 써야 한다. 이는 글로벌 애플리케이션을 개발할 때 복잡성을 증가시킬 수 있다.&#x20;

### 문자열 타입을 사용할 경우&#x20;

#### 장점&#x20;

* 형식의 유연성 : 날짜로 문자열을 저장하면 원하는 형식으로 자유롭게 저장할 수 있다.&#x20;
* 단순한 입력 : 유효성 검사를 하지 않으면 단순히 문자열로 입력할 수 있다. 특수한 날짜나 임의의 문자열도 저장할 수 있다(?)\
  \-> 이건 장점이 아닌 것 같다..&#x20;

#### 단점&#x20;

* 데이터 무결성 문제 : 문자열은 유효한 날짜인지 여부를 검사하지 않으므로 잘못된 날짜 형식이 저장될 위험이 있다.&#x20;
* 복잡한 연산 : 문자열로 저장된 날짜는 계산이 어렵다  .. \
  \-> 이 부분은 JexFramework 에서 함수로 지원을 해준다. \
  \-> 전문 통신이 많은 서비스 특성상 날짜와 시간을 저장할 때 문자열 타입을 사용하는 것으로 판단
* 성능 저하 : 날짜 연산을 위해 문자열을 자주 변환해야 하므로 성능이 떨어질 수 있다. 또한, 날짜를 기준으로 정렬하거나 필터링 시 성능이 저하될 수 있다.&#x20;
* 표준화 부족 : 여러 사람이 데이터를 입력할 때 날짜 형식이 일관되지 않을 수 있다 .. \
  \-> 이렇기 때문에, 문자열을 직접 입력 받는 것이 아닌, API 내에서 사용되는 것이 현명한 사용이라 판단

## 4. BPCHAR, VARCHAR, TEXT, DATE 인덱싱은 어떻게 적용될까?

### 문자열 타입의 인덱싱&#x20;

#### 인덱스 생성&#x20;

* 문자열 컬럼에 인덱스를 생성하면, PostgreSQL 은 사전 순서에 따라서 문자열을 생성한다.&#x20;
* 기본적으로는 B-Tree 인덱스가 사용된다.&#x20;

```sql
CREATE INDEX idx_name ON employees (last_time); 
```

#### 인덱스의 작동 방식

* 문자열 인덱스는 문자열을 알파벳 순으로 정렬하여 인덱스에 저장한다. 이로 인해 `LIKE`, `=`, `>`, `<` 연산에서 빠르게 검색할 수 있다.&#x20;
* 문자열 인덱스는 `LIKE 'abc%'` 같은 패턴 매칭에는 효과적이지만, `LIKE '%abc'` 같은 접두어가없는 검색에는 효과가 떨어진다.&#x20;
* **`bpchar` 타입은 고정 크기 타입으로 , 고정 크기 이하의 문자열은 오른쪽에 공백을 채워 크기를 유지하는데, 이 공백은 인덱스 적용에 영향을 미칠 수 있다.** \
  \-> 예를 들어서 `'abc'` 와 `'abc '` 를 동일하게 취급하거나 비교하기 어렵다. \
  \-> 만약, `bpchar` 타입의 커럼을 공백이 없는 문자열로 채웠다고 하더라도, 동작하지 않는 경우가 많다.. (경험담!)

#### 성능 고려사항&#x20;

* 길이와 중복도 : 긴 문자열이나, 중복된 값이 많은 컬럼에 인덱스를 적용하면 성능이 저하될 수 있다. 특히, 텍스트가 길거나 패턴 매칭이 복잡할 때 인덱스 효율이 떨어진다.&#x20;

### 날짜 / 시간 타입 인덱싱&#x20;

#### 인덱스 생성&#x20;

* 날짜, 시간 컬럼에 인덱스를 생성하면, PostgreSQL 은 시간 순서에 따라 데이터를 인덱싱한다. 이 또한 기본적으로 B-Tree 인덱스가 사용된다.&#x20;

```sql
CREATE INDEX idx_date ON events (event_date); 
```

#### 인덱스의 동작 방식&#x20;

* 날짜, 시간 인덱스는 해당 값들을 시간 순서대로 정렬하여 인덱스에 저장한다. 이를 통해 날짜 범위 검색이 매우 빠르게 이루어진다. (`BETWEEN`, `<`, `>`)\
  \-> 예를 들어, 특정 기간 내의 데이터를 조회할 때 인덱스가 효과적으로 사용된다.&#x20;

#### 성능 고려 사항&#x20;

* 정확한 정렬 및 범위 검색 : 날짜, 시간 인덱스는 정렬된 순서로 저장되기 때문에, 특정 기간 내의 데이터를 검색할 때 매우 빠르다.&#x20;
* 시간대 처리 : `timestamp with time zone` 타입을 사용하는 경우, 시간대가 인덱스에 포함되어 관리된다. 따라서 시간대 변환을 포함한 쿼리에서도 인덱스를 효과적으로 사용할 수 있다.&#x20;

## 5. 주의사항&#x20;

### 문자열 타입을 YYYYMMDD24MISS 형식으로 저장하고 인덱싱 하게 된다면 timestamp 형식이랑 동일한 성능을 보이지 않을까?&#x20;

#### 두 접근 방식의 차이점&#x20;

* 인덱스 페이지 크기 및 메모리 사용&#x20;
  * 문자열은 각 문자마다 바이트를 차지하며, 일반적으로 `timestamp` 보다 더 많은 메모리를 사용한다. 이로 인해 인덱스 페이지 크기가 더 커질 수 있고, 인덱스를 읽는 데 더 많은 I/O 가 발생할 수 있다.&#x20;
  * 하지만, `timestamp` 는 8바이트로 고정된 크기를 가지므로, 메모리와 I/O 측면에서 더욱 효율적이다.&#x20;
* 연산 성능&#x20;
  * `timestamp` 는 **숫자 기반의 비교 연산을 수행하므로, 범위 검색 및 계산에 있어서 더 빠르다.**&#x20;
  * 문자열은 문자열 비교가 이루어지기 때문에, CPU 사용량이 상대적으로 더 높을 수 있다. \
    \-> 특히, 정렬과 같은 작업에서 성능 차이가 나타날 수 있다.&#x20;
* 추가적인 기능 지원&#x20;
  * `timestamp` 는 날짜 및 시간과 관련된 다양한 SQL 함수를 사용할 수 있다.&#x20;
  * 문자열 형식을 사용하면 연산을 수동으로 작업을 해야하며, 이러한 작업은 성능 저하와 코드 복잡성을 초래할 수 있다.&#x20;
