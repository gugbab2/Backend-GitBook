# Optimize database access(데이터베이스 접근 최적화)

## Django 데이터베이스 접근 최적화 가이드

원본: https://docs.djangoproject.com/en/6.0/topics/db/optimization/

이 문서는 Django 공식 문서의 핵심 개념을 한글로 재구성한 학습 자료입니다.

***

### 1. 먼저 측정하라 (Profile First)

최적화의 첫 번째 원칙: **추측하지 말고 측정하라.**

어떤 쿼리가 실행되고 있는지, 얼마나 비용이 드는지를 먼저 확인해야 한다. Django에서 이를 확인하는 방법은 세 가지다.

**`QuerySet.explain()`**: 특정 쿼리셋이 DB에서 어떻게 실행되는지 보여준다. 인덱스를 타는지, 풀 스캔을 하는지, 조인이 어떻게 되는지 확인할 수 있다.

```python
# 이 쿼리가 인덱스를 타는지 확인
print(Entry.objects.filter(blog_id=1).explain())
```

**django-debug-toolbar**: 개발 환경에서 각 요청마다 실행된 SQL 수, 실행 시간, 중복 쿼리를 시각적으로 보여주는 도구다.

**`django.db.connection.queries`**: Django가 생성한 SQL을 직접 확인할 수 있다.

최적화할 때 기억할 것: 속도와 메모리 사용량은 트레이드오프 관계일 수 있다. 그리고 **모든 변경 후에 반드시 다시 측정해서** 실제로 개선되었는지 확인해야 한다.

***

### 2. 기본적인 DB 최적화 기법

Django 코드를 건드리기 전에 DB 자체를 먼저 점검해야 한다.

**인덱스**: 가장 중요한 최적화. `filter()`, `exclude()`, `order_by()` 등에서 자주 사용하는 필드에 인덱스를 추가한다. Django에서는 `Field.db_index=True` 또는 `Meta.indexes`로 설정한다. 단, 인덱스 유지 비용(쓰기 성능 저하)도 있으므로 무조건 추가하는 건 아니다.

**적절한 필드 타입 사용**: 데이터 특성에 맞는 필드 타입을 선택한다.

***

### 3. QuerySet의 동작 방식을 이해하라

QuerySet을 제대로 이해하면 간결한 코드로 좋은 성능을 낼 수 있다. 핵심 개념 세 가지:

#### 3-1. QuerySet은 게으르다 (Lazy)

QuerySet을 정의하는 것만으로는 DB 쿼리가 실행되지 않는다. 실제로 데이터를 사용할 때(반복, 슬라이싱, `len()`, `list()`, `bool()` 등) 비로소 쿼리가 실행된다.

```python
# 이 시점에서는 쿼리가 실행되지 않는다
qs = Entry.objects.filter(blog_id=1)

# 이 시점에서 쿼리가 실행된다
for entry in qs:
    print(entry.headline)
```

#### 3-2. 캐시된 속성 vs 매번 호출되는 속성

호출 가능하지 않은(callable이 아닌) 속성은 캐시된다. 호출 가능한 속성은 매번 DB를 조회한다.

```python
entry = Entry.objects.get(id=1)

# blog는 callable이 아님 → 첫 접근 시 DB 조회, 이후 캐시 사용
entry.blog   # DB 조회 발생
entry.blog   # 캐시된 값 사용, DB 조회 없음

# authors.all()은 callable → 호출할 때마다 DB 조회
entry.authors.all()  # DB 조회 발생
entry.authors.all()  # 또 DB 조회 발생
```

이 차이를 모르면 템플릿이나 반복문 안에서 불필요한 쿼리가 계속 발생할 수 있다.

#### 3-3. iterator()로 메모리 절약

대량의 객체를 처리할 때 QuerySet의 캐시가 메모리를 많이 차지할 수 있다. `iterator()`를 사용하면 결과를 캐시하지 않고 하나씩 가져온다.

```python
# 메모리를 많이 쓸 수 있음
for entry in Entry.objects.all():
    process(entry)

# 메모리를 절약
for entry in Entry.objects.all().iterator():
    process(entry)
```

***

### 4. DB에서 처리할 일은 DB에서 하라

Python에서 반복문으로 처리하는 것보다 DB가 처리하는 게 거의 항상 빠르다.

```python
# 나쁜 예: Python에서 필터링
all_entries = Entry.objects.all()
filtered = [e for e in all_entries if e.blog_id == 1]

# 좋은 예: DB에서 필터링
filtered = Entry.objects.filter(blog_id=1)
```

```python
# 나쁜 예: Python에서 집계
entries = Entry.objects.all()
total = sum(e.rating for e in entries)

# 좋은 예: DB에서 집계
from django.db.models import Sum
total = Entry.objects.aggregate(Sum('rating'))
```

**F 표현식**을 사용하면 같은 모델 내 다른 필드 값을 기준으로 필터링할 수 있다.

```python
from django.db.models import F

# 댓글 수가 조회수보다 많은 글 찾기 (DB에서 처리)
Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks'))
```

ORM으로 원하는 SQL을 만들 수 없을 때는 `RawSQL` 표현식이나 Raw SQL을 사용할 수 있다. 단, 이때는 SQL Injection에 주의해야 한다.

***

### 5. 개별 객체 조회 시 유니크하고 인덱스된 컬럼을 사용하라

`get()`으로 단일 객체를 조회할 때는 `unique`이거나 `db_index`가 설정된 필드를 사용한다.

```python
# 빠름: id는 인덱스가 있고 유니크함
entry = Entry.objects.get(id=10)

# 느림: headline에 인덱스가 없음
entry = Entry.objects.get(headline="News Item Title")

# 더 느림: startswith는 인덱스 활용이 제한적이고, 여러 결과가 나올 수 있음
entry = Entry.objects.get(headline__startswith="News")
```

인덱스가 없는 필드로 조회하면 DB가 전체 테이블을 스캔한다. 또한 유니크하지 않은 필드로 `get()`을 호출하면, 조건에 맞는 모든 레코드를 가져온 뒤에야 하나인지 확인하므로 매우 비효율적이다.

***

### 6. 필요한 데이터는 한 번에 가져와라

#### select\_related()와 prefetch\_related()

Django에서 가장 중요한 최적화 도구다. 관련 객체를 사전에 로드해서 N+1 문제를 방지한다.

**`select_related()`**: ForeignKey, OneToOneField 관계에 사용. SQL JOIN으로 한 번에 가져온다.

```python
# 나쁜 예: N+1 문제 발생
entries = Entry.objects.all()
for entry in entries:
    print(entry.blog.name)  # 매번 blog 테이블을 조회

# 좋은 예: JOIN으로 한 번에 가져옴
entries = Entry.objects.select_related('blog')
for entry in entries:
    print(entry.blog.name)  # 추가 쿼리 없음
```

**`prefetch_related()`**: ManyToManyField, 역참조 관계에 사용. 별도 쿼리를 실행한 뒤 Python에서 합친다.

```python
# 나쁜 예: 매번 authors 테이블 조회
entries = Entry.objects.all()
for entry in entries:
    print(entry.authors.all())  # 매번 쿼리 발생

# 좋은 예: 별도 쿼리 1개로 모든 authors를 미리 가져옴
entries = Entry.objects.prefetch_related('authors')
for entry in entries:
    print(entry.authors.all())  # 캐시된 결과 사용
```

***

### 7. 필요 없는 데이터는 가져오지 마라

#### values()와 values\_list()

ORM 모델 객체 전체가 필요 없고, 특정 필드의 값만 필요할 때 사용한다.

```python
# 모든 필드를 가져옴 (불필요한 데이터 포함)
entries = Entry.objects.all()

# 필요한 필드만 딕셔너리로 가져옴
entries = Entry.objects.values('id', 'headline')

# 필요한 필드만 튜플로 가져옴
entries = Entry.objects.values_list('id', 'headline')
```

#### defer()와 only()

모델 객체는 필요하지만 특정 큰 필드(텍스트, BLOB 등)를 제외하고 싶을 때 사용한다.

```python
# body 필드를 제외하고 가져옴 (나중에 접근하면 그때 별도 쿼리 발생)
entries = Entry.objects.defer('body')

# headline과 blog만 가져옴
entries = Entry.objects.only('headline', 'blog')
```

주의: `defer()`/`only()`로 제외한 필드에 나중에 접근하면 별도 쿼리가 발생한다. 프로파일링 없이 과도하게 사용하면 오히려 성능이 나빠질 수 있다.

#### count(), exists(), contains()는 적절히 사용하라

```python
# 개수만 필요할 때
Entry.objects.filter(blog_id=1).count()  # SELECT COUNT(*)
# 이렇게 하면 모든 객체를 메모리에 올림 → 느림
len(Entry.objects.filter(blog_id=1))

# 존재 여부만 확인할 때
Entry.objects.filter(blog_id=1).exists()  # LIMIT 1
# 이렇게 하면 모든 객체를 가져온 뒤 bool로 변환 → 느림
if Entry.objects.filter(blog_id=1):
    pass
```

**하지만 과도하게 사용하지 마라.** 어차피 데이터를 전부 쓸 거라면 `count()`, `exists()`, `contains()`를 별도로 호출하면 쿼리가 추가된다.

```python
members = group.members.all()

# 이 코드는 총 0~1개의 쿼리만 실행한다
if members:                        # 여기서 쿼리 실행 + 결과 캐시
    if current_user in members:    # 캐시된 결과에서 확인
        print(len(members))        # 캐시된 결과에서 계산
    for member in members:         # 캐시된 결과를 순회
        print(member.username)

# 만약 exists(), contains(), count()를 각각 호출하면 쿼리가 3개 추가된다
```

핵심: QuerySet을 변수에 저장하면 결과가 캐시되어 재사용된다. 이미 전체 데이터를 쓸 계획이라면 `exists()`나 `count()` 대신 캐시를 활용하라.

#### FK 값은 직접 접근하라

관련 객체의 PK만 필요할 때 전체 객체를 가져올 필요 없다.

```python
# 나쁜 예: blog 객체 전체를 가져온 뒤 id를 꺼냄
blog_id = entry.blog.id

# 좋은 예: 이미 entry에 저장된 FK 값을 바로 사용
blog_id = entry.blog_id
```

#### 필요 없는 정렬을 제거하라

정렬은 공짜가 아니다. 모델에 기본 정렬(`Meta.ordering`)이 설정되어 있는데 정렬이 필요 없다면 명시적으로 제거한다.

```python
# Meta.ordering이 설정된 모델에서 정렬을 제거
Entry.objects.order_by()
```

#### update()와 delete()로 벌크 처리하라

객체를 하나씩 가져와서 수정하고 저장하는 대신, 벌크 SQL 문을 사용한다.

```python
# 나쁜 예: 각 객체마다 UPDATE 쿼리 발생
for entry in Entry.objects.filter(blog_id=1):
    entry.headline = "Updated"
    entry.save()

# 좋은 예: 하나의 UPDATE 쿼리로 처리
Entry.objects.filter(blog_id=1).update(headline="Updated")
```

주의: 벌크 `update()`/`delete()`는 개별 인스턴스의 `save()`/`delete()` 메서드를 호출하지 않는다. 시그널이나 커스텀 로직이 있다면 동작하지 않을 수 있다.

***

### 8. 벌크 메서드를 사용하라

SQL 문 수를 줄여서 성능을 개선한다.

#### bulk\_create()

```python
# 나쁜 예: INSERT 쿼리가 2개
Entry.objects.create(headline="Test 1")
Entry.objects.create(headline="Test 2")

# 좋은 예: INSERT 쿼리 1개
Entry.objects.bulk_create([
    Entry(headline="Test 1"),
    Entry(headline="Test 2"),
])
```

#### bulk\_update()

```python
# 나쁜 예: UPDATE 쿼리가 2개
entries[0].headline = "Updated 1"
entries[0].save()
entries[1].headline = "Updated 2"
entries[1].save()

# 좋은 예: UPDATE 쿼리 1개
entries[0].headline = "Updated 1"
entries[1].headline = "Updated 2"
Entry.objects.bulk_update(entries, ["headline"])
```

#### ManyToMany 관계의 벌크 추가/삭제

```python
# 나쁜 예: INSERT 쿼리가 2개
my_band.members.add(me)
my_band.members.add(my_friend)

# 좋은 예: INSERT 쿼리 1개
my_band.members.add(me, my_friend)
```

삭제도 마찬가지로 `remove()`에 여러 객체를 한 번에 전달하는 게 효율적이다.

***

### 핵심 체크리스트

| 상황                       | 해야 할 것                                                      |
| ------------------------ | ----------------------------------------------------------- |
| 어떤 쿼리가 실행되는지 모를 때        | `explain()`, django-debug-toolbar로 확인                       |
| 관련 객체를 반복문 안에서 접근할 때     | `select_related()` 또는 `prefetch_related()` 사용               |
| 특정 필드 값만 필요할 때           | `values()` 또는 `values_list()` 사용                            |
| FK의 PK만 필요할 때            | `entry.blog_id` 사용 (`entry.blog.id` 아님)                     |
| Python에서 반복문으로 필터링/집계할 때 | `filter()`, `annotate()`, `F()` 등으로 DB에서 처리                 |
| 개수만 필요할 때                | `count()` 사용 (`len()` 아님)                                   |
| 존재 여부만 확인할 때             | `exists()` 사용                                               |
| 여러 객체를 생성/수정/삭제할 때       | `bulk_create()`, `bulk_update()`, `update()`, `delete()` 사용 |
| 대량 데이터를 순회할 때            | `iterator()` 사용                                             |
| 정렬이 필요 없을 때              | `order_by()`로 기본 정렬 제거                                      |
