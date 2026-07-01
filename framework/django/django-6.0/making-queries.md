# Making queries(쿼리 생성)

## Django 쿼리 작성 가이드

원본: https://docs.djangoproject.com/en/6.0/topics/db/queries/

이 문서는 Django ORM으로 데이터를 생성, 조회, 수정, 삭제하는 방법을 다룬다. 아래 예시에서 사용하는 모델 구조:

```python
class Blog(models.Model):
    name = models.CharField(max_length=100)
    tagline = models.TextField()

class Author(models.Model):
    name = models.CharField(max_length=200)
    email = models.EmailField()

class Entry(models.Model):
    blog = models.ForeignKey(Blog, on_delete=models.CASCADE)
    headline = models.CharField(max_length=255)
    body_text = models.TextField()
    pub_date = models.DateField()
    mod_date = models.DateField(default=date.today)
    authors = models.ManyToManyField(Author)
    number_of_comments = models.IntegerField(default=0)
    number_of_pingbacks = models.IntegerField(default=0)
    rating = models.IntegerField(default=5)
```

***

### 1. 객체 생성 (CREATE)

모델 인스턴스를 만들고 `save()`를 호출하면 DB에 INSERT가 실행된다.

```python
b = Blog(name="Beatles Blog", tagline="All the latest Beatles news.")
b.save()  # 이 시점에 INSERT 실행
```

`save()`를 호출하기 전까지는 DB에 아무 일도 일어나지 않는다.

한 번에 생성하고 저장하려면 `create()` 사용:

```python
b = Blog.objects.create(name="Beatles Blog", tagline="All the latest Beatles news.")
```

#### ForeignKey 저장

관련 객체를 할당하고 `save()` 호출:

```python
entry = Entry.objects.get(pk=1)
cheese_blog = Blog.objects.get(name="Cheddar Talk")
entry.blog = cheese_blog
entry.save()
```

#### ManyToManyField 저장

`add()` 메서드 사용. `save()` 필요 없이 즉시 DB에 반영된다:

```python
joe = Author.objects.create(name="Joe")
entry.authors.add(joe)

# 여러 개를 한 번에 추가
entry.authors.add(john, paul, george, ringo)
```

***

### 2. 객체 수정 (UPDATE)

이미 DB에 있는 객체의 속성을 변경하고 `save()` 호출:

```python
b5.name = "New name"
b5.save()  # UPDATE 실행
```

여러 객체를 한 번에 수정하려면 `update()` 사용:

```python
Entry.objects.filter(pub_date__year=2007).update(headline="Everything is the same")
```

`update()`는 SQL UPDATE를 직접 실행한다. 개별 인스턴스의 `save()` 메서드를 호출하지 않으므로, `pre_save`/`post_save` 시그널이나 `auto_now` 같은 옵션이 동작하지 않는다.

F 표현식으로 현재 값 기반 업데이트도 가능:

```python
from django.db.models import F
Entry.objects.update(number_of_pingbacks=F('number_of_pingbacks') + 1)
```

***

### 3. 객체 조회 (READ)

#### 3-1. QuerySet 기본 개념

`QuerySet`은 DB에서 가져올 객체의 컬렉션을 표현한다. SQL의 SELECT 문에 해당하며, filter는 WHERE 절에 해당한다.

```python
# 전체 조회
all_entries = Entry.objects.all()

# 조건 조회
Entry.objects.filter(pub_date__year=2006)
```

#### 3-2. QuerySet은 게으르다 (Lazy Evaluation)

QuerySet을 정의하는 것만으로는 DB 쿼리가 실행되지 않는다. 실제로 데이터를 사용할 때 비로소 실행된다.

```python
q = Entry.objects.filter(headline__startswith="What")
q = q.filter(pub_date__lte=datetime.date.today())
q = q.exclude(body_text__icontains="food")
# 여기까지 DB 쿼리 0개

print(q)  # 이 시점에 비로소 쿼리 실행
```

필터를 아무리 많이 체이닝해도, 실제로 결과를 "요청"하기 전까지 DB를 건드리지 않는다.

#### 3-3. 필터 체이닝

QuerySet을 필터링하면 새로운 QuerySet이 생성된다. 원본은 변하지 않는다.

```python
q1 = Entry.objects.filter(headline__startswith="What")
q2 = q1.exclude(pub_date__gte=datetime.date.today())   # q1과 별개
q3 = q1.filter(pub_date__gte=datetime.date.today())     # q1과 별개
# q1, q2, q3는 각각 독립적인 QuerySet
```

#### 3-4. get() vs filter()

`get()`은 정확히 1개의 객체를 반환. 0개면 `DoesNotExist`, 2개 이상이면 `MultipleObjectsReturned` 예외 발생.

```python
one_entry = Entry.objects.get(pk=1)
```

`filter()`는 항상 QuerySet을 반환. 결과가 0개여도 예외 없이 빈 QuerySet을 반환.

#### 3-5. 결과 수 제한 (LIMIT/OFFSET)

Python 슬라이싱 문법 사용:

```python
Entry.objects.all()[:5]      # LIMIT 5
Entry.objects.all()[5:10]    # OFFSET 5 LIMIT 5
```

음수 인덱싱은 지원하지 않는다. 슬라이싱은 새 QuerySet을 반환하며 쿼리를 실행하지 않는다. 단, step을 사용하면(`[:10:2]`) 즉시 실행된다.

단일 객체를 인덱스로 가져오기:

```python
Entry.objects.order_by('headline')[0]  # 결과 없으면 IndexError
```

***

### 4. 필드 룩업 (Field Lookups)

SQL WHERE 절의 조건을 지정하는 방법. `필드명__룩업타입=값` 형태의 키워드 인자로 사용한다.

```python
Entry.objects.filter(pub_date__lte='2006-01-01')
# SQL: SELECT * FROM entry WHERE pub_date <= '2006-01-01';
```

룩업 타입을 생략하면 `exact`가 기본값:

```python
Blog.objects.get(id__exact=14)  # 명시적
Blog.objects.get(id=14)          # 동일 (exact 생략)
```

#### 주요 룩업 타입

```python
# exact: 정확히 일치
Entry.objects.get(headline__exact="Cat bites dog")

# iexact: 대소문자 무시 일치
Blog.objects.get(name__iexact="beatles blog")

# contains: 포함 (대소문자 구분)
Entry.objects.get(headline__contains="Lennon")
# SQL: WHERE headline LIKE '%Lennon%'

# icontains: 포함 (대소문자 무시)
Entry.objects.get(headline__icontains="lennon")

# startswith / endswith: 시작/끝 일치
# istartswith / iendswith: 대소문자 무시 버전

# gt, gte, lt, lte: 크기 비교
Entry.objects.filter(pub_date__gt='2006-01-01')

# in: 목록 안에 포함
Entry.objects.filter(id__in=[1, 3, 4])

# isnull: NULL 여부
Entry.objects.filter(pub_date__isnull=True)

# year, month, day: 날짜 필드의 연/월/일
Entry.objects.filter(pub_date__year=2006)
```

ForeignKey 필드는 `_id` 접미사로 직접 값 비교 가능:

```python
Entry.objects.filter(blog_id=4)  # blog 객체 대신 id 직접 사용
```

#### pk 단축키

`pk`는 모델의 기본키 필드를 가리키는 단축키:

```python
Blog.objects.get(id__exact=14)  # 명시적
Blog.objects.get(id=14)          # exact 생략
Blog.objects.get(pk=14)          # pk 단축키
```

조인에서도 사용 가능:

```python
Entry.objects.filter(blog__pk=3)  # blog__id__exact=3 과 동일
```

***

### 5. 관계를 넘나드는 조회 (JOIN)

더블 언더스코어(`__`)로 관계를 따라가면 Django가 자동으로 SQL JOIN을 처리한다.

```python
# Entry → Blog 관계를 따라가서 Blog의 name으로 필터링
Entry.objects.filter(blog__name="Beatles Blog")
# SQL: SELECT ... FROM entry INNER JOIN blog ON ... WHERE blog.name = 'Beatles Blog'
```

역방향도 가능:

```python
# Blog에서 Entry의 headline으로 필터링
Blog.objects.filter(entry__headline__contains="Lennon")
```

깊이 제한 없이 체이닝 가능:

```python
Blog.objects.filter(entry__authors__name="Lennon")
```

#### 다중 값 관계에서 filter의 동작 (매우 중요)

이 부분은 AI가 짠 코드에서 자주 실수가 발생하는 영역이다.

**하나의 filter() 안에 조건을 넣으면**: 같은 관련 객체가 모든 조건을 만족해야 한다 (AND on same object).

```python
# 2008년에 발행되었고 AND headline에 "Lennon"이 포함된 Entry가 있는 Blog
Blog.objects.filter(
    entry__headline__contains="Lennon",
    entry__pub_date__year=2008,
)
```

**filter()를 체이닝하면**: 각 조건이 서로 다른 관련 객체에 적용될 수 있다 (AND on potentially different objects).

```python
# headline에 "Lennon"이 포함된 Entry가 있고,
# 별도로 2008년에 발행된 Entry도 있는 Blog (같은 Entry일 필요 없음)
Blog.objects.filter(
    entry__headline__contains="Lennon",
).filter(
    entry__pub_date__year=2008,
)
```

이 차이를 모르면 의도와 다른 결과가 나올 수 있다. 조건이 같은 관련 객체를 가리켜야 하면 하나의 `filter()` 안에 넣고, 각각 독립적이어도 되면 체이닝한다.

#### exclude()의 다른 동작

`exclude()`는 `filter()`와 다르게 동작한다. 하나의 `exclude()` 안에 여러 조건을 넣어도, 같은 관련 객체를 가리키지 않을 수 있다.

```python
# "Lennon"이 포함된 Entry도 있고 2008년 Entry도 있는 Blog를 제외
# (같은 Entry가 아니어도 됨)
Blog.objects.exclude(
    entry__headline__contains="Lennon",
    entry__pub_date__year=2008,
)

# 같은 Entry에서 두 조건을 모두 만족하는 것을 제외하려면:
Blog.objects.exclude(
    entry__in=Entry.objects.filter(
        headline__contains="Lennon",
        pub_date__year=2008,
    ),
)
```

***

### 6. F 표현식 (같은 모델의 다른 필드 참조)

`F()` 객체를 사용하면 같은 모델 내 다른 필드의 값을 참조할 수 있다. DB 레벨에서 처리되므로 Python으로 값을 가져올 필요가 없다.

```python
from django.db.models import F

# 댓글 수가 핑백 수보다 많은 글
Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks'))

# 댓글 수가 핑백 수의 2배보다 많은 글
Entry.objects.filter(number_of_comments__gt=F('number_of_pingbacks') * 2)

# rating이 (댓글 수 + 핑백 수)보다 작은 글
Entry.objects.filter(rating__lt=F('number_of_comments') + F('number_of_pingbacks'))
```

관계를 넘나드는 F 표현식:

```python
# 저자 이름이 블로그 이름과 같은 글
Entry.objects.filter(authors__name=F('blog__name'))
```

날짜 연산:

```python
from datetime import timedelta
# 발행일로부터 3일 이후에 수정된 글
Entry.objects.filter(mod_date__gt=F('pub_date') + timedelta(days=3))
```

***

### 7. Q 객체 (복잡한 OR/NOT 조건)

`filter()`의 키워드 인자는 기본적으로 AND로 결합된다. OR 조건이 필요하면 `Q` 객체를 사용한다.

```python
from django.db.models import Q

# OR 조건
Entry.objects.filter(
    Q(headline__startswith="Who") | Q(headline__startswith="What")
)
# SQL: WHERE headline LIKE 'Who%' OR headline LIKE 'What%'

# NOT 조건
Q(question__startswith="Who") | ~Q(pub_date__year=2005)
# SQL: ... OR NOT pub_date_year = 2005

# AND + OR 결합
Poll.objects.get(
    Q(question__startswith="Who"),
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
)
# SQL: WHERE question LIKE 'Who%'
#   AND (pub_date = '2005-05-02' OR pub_date = '2005-05-06')
```

Q 객체와 키워드 인자를 함께 쓸 수 있지만, **Q 객체가 키워드 인자보다 앞에** 와야 한다:

```python
# 올바름
Poll.objects.get(
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
    question__startswith="Who",
)

# 에러 발생
Poll.objects.get(
    question__startswith="Who",
    Q(pub_date=date(2005, 5, 2)) | Q(pub_date=date(2005, 5, 6)),
)
```

***

### 8. 관련 객체 (Related Objects)

#### One-to-Many (ForeignKey)

**정방향 접근** (Entry → Blog): 속성으로 직접 접근. 첫 접근 시 DB 조회, 이후 캐시됨.

```python
e = Entry.objects.get(id=2)
e.blog       # DB 조회 (Blog 객체 반환)
e.blog       # 캐시 사용, DB 조회 없음
```

`select_related()`로 미리 가져오면 최초 접근에서도 추가 쿼리 없음:

```python
e = Entry.objects.select_related().get(id=2)
e.blog       # 추가 쿼리 없음
```

**역방향 접근** (Blog → Entry): `모델명_set` Manager 사용.

```python
b = Blog.objects.get(id=1)
b.entry_set.all()                              # 모든 관련 Entry
b.entry_set.filter(headline__contains="Lennon") # 필터링 가능
b.entry_set.count()                            # 개수
```

`related_name`을 설정하면 `_set` 대신 지정한 이름 사용:

```python
# 모델 정의: blog = ForeignKey(Blog, related_name='entries')
b.entries.all()  # entry_set 대신 entries 사용
```

관련 객체 조작 메서드:

```python
b.entry_set.add(e1)           # 추가
b.entry_set.create(headline="New") # 생성 + 추가
b.entry_set.remove(e1)        # 제거
b.entry_set.clear()           # 전체 제거
b.entry_set.set([e1, e2])     # 교체
```

#### Many-to-Many

양쪽 모두 접근 가능. ManyToManyField를 정의한 쪽은 필드명으로, 반대쪽은 `모델명_set`으로 접근:

```python
e = Entry.objects.get(id=3)
e.authors.all()          # Entry → Author (필드명 사용)

a = Author.objects.get(id=5)
a.entry_set.all()        # Author → Entry (역방향, _set 사용)
```

#### One-to-One

ForeignKey와 유사하지만, 역방향 접근이 Manager가 아닌 단일 객체를 반환:

```python
ed = EntryDetail.objects.get(id=2)
ed.entry                 # 정방향: Entry 객체 반환

e = Entry.objects.get(id=2)
e.entrydetail            # 역방향: EntryDetail 단일 객체 반환 (없으면 DoesNotExist)
```

#### M2M에서 filter()의 sticky 동작

M2M 관계에서 `filter()`는 JOIN이 한 번만 수행되어 "sticky"하게 동작한다:

```python
# anna의 Entry 중에서 authors에 "Gloria"가 포함된 것을 찾으려 함
anna.entry_set.filter(authors__name="Gloria")
# 결과: 빈 QuerySet (예상과 다름!)

# 이유: JOIN이 한 번만 수행되어, 하나의 author-entry 관계에서
# anna와 gloria가 동시에 매칭되는 것을 찾기 때문

# 해결: filter()를 체이닝하면 별도 JOIN이 수행됨
anna.entry_set.filter().filter(authors__name="Gloria")
# 결과: 예상대로 Entry 반환
```

***

### 9. QuerySet 캐시

QuerySet은 처음 평가될 때 결과를 캐시에 저장하고, 이후 재사용한다.

```python
# 나쁜 예: 같은 쿼리가 2번 실행됨
print([e.headline for e in Entry.objects.all()])
print([e.pub_date for e in Entry.objects.all()])

# 좋은 예: 쿼리 1번, 캐시 재사용
queryset = Entry.objects.all()
print([p.headline for p in queryset])    # 쿼리 실행 + 캐시
print([p.pub_date for p in queryset])    # 캐시 재사용
```

#### 캐시가 되지 않는 경우

슬라이싱이나 인덱스 접근은 전체 QuerySet을 캐시하지 않는다:

```python
queryset = Entry.objects.all()
print(queryset[5])  # DB 조회
print(queryset[5])  # 또 DB 조회 (캐시 안 됨)

# 전체 평가 후에는 인덱스도 캐시 사용
[entry for entry in queryset]  # 전체 평가 + 캐시
print(queryset[5])              # 캐시 사용
```

전체 캐시를 채우는 동작: `list(qs)`, `bool(qs)`, `for x in qs`, `x in qs`. `print(qs)`는 `__repr__()`이 슬라이스만 반환하므로 캐시를 채우지 않는다.

***

### 10. 객체 삭제 (DELETE)

```python
# 단일 삭제
e.delete()  # (1, {'blog.Entry': 1}) 반환

# 벌크 삭제
Entry.objects.filter(pub_date__year=2005).delete()
```

Django는 기본적으로 `ON DELETE CASCADE`를 따른다. Blog를 삭제하면 관련 Entry도 모두 삭제된다. 이 동작은 ForeignKey의 `on_delete` 인자로 커스터마이즈 가능하다.

안전장치: `Entry.objects.delete()`는 동작하지 않는다. 전체 삭제는 `Entry.objects.all().delete()`로 명시해야 한다.

***

### 11. JSONField 쿼리

JSONField는 키, 인덱스, 경로를 더블 언더스코어로 탐색할 수 있다.

```python
class Dog(models.Model):
    name = models.CharField(max_length=200)
    data = models.JSONField(null=True)

# 키로 필터링
Dog.objects.filter(data__breed="collie")

# 중첩 키 경로
Dog.objects.filter(data__owner__name="Bob")

# 배열 인덱스
Dog.objects.filter(data__owner__other_pets__0__name="Fishy")

# 키 존재 여부
Dog.objects.filter(data__has_key="owner")
Dog.objects.filter(data__has_keys=["breed", "owner"])
Dog.objects.filter(data__has_any_keys=["owner", "breed"])
```

#### None vs JSON null

JSONField에서 `None`은 주의가 필요하다:

```python
Dog.objects.create(name="Max", data=None)                    # SQL NULL
Dog.objects.create(name="Archie", data=Value(None, JSONField()))  # JSON null

Dog.objects.filter(data=None)            # JSON null인 Archie만 반환
Dog.objects.filter(data__isnull=True)    # SQL NULL인 Max만 반환
```

***

### 12. 비동기 쿼리

비동기 뷰에서는 동기 ORM 메서드를 직접 호출할 수 없다. `a` 접두사가 붙은 비동기 버전을 사용한다.

```python
# 동기
user = User.objects.filter(username=my_input).first()

# 비동기
user = await User.objects.filter(username=my_input).afirst()
```

`filter()`, `exclude()` 같은 QuerySet을 반환하는 메서드는 쿼리를 실행하지 않으므로 동기/비동기 모두 사용 가능. `get()`, `first()`, `delete()` 같은 결과를 반환하는 메서드만 비동기 버전(`aget()`, `afirst()`, `adelete()`)을 사용해야 한다.

비동기 반복:

```python
async for entry in Entry.objects.filter(headline__startswith="A"):
    ...
```

주의: 비동기 환경에서는 트랜잭션을 사용할 수 없다.

***

### 핵심 체크리스트

| 상황               | 기억할 것                                    |
| ---------------- | ---------------------------------------- |
| QuerySet 정의      | 쿼리가 실행되지 않는다 (lazy)                      |
| 같은 데이터를 여러 번 쓸 때 | QuerySet을 변수에 저장해서 캐시 재사용                |
| 인덱스/슬라이싱 접근      | 캐시되지 않으므로 주의                             |
| 관계를 따라가는 필터      | 하나의 filter()=같은 객체, 체이닝=다른 객체 가능         |
| OR 조건            | Q 객체 사용                                  |
| 같은 모델 필드 비교      | F 표현식 사용                                 |
| 단일 객체 조회         | get()은 없으면 예외, filter().first()는 None 반환 |
| FK의 PK만 필요할 때    | entry.blog\_id (entry.blog.id 아님)        |
| 벌크 수정            | update()는 save() 시그널 미동작 주의              |
| CASCADE 삭제       | Blog 삭제 시 관련 Entry도 삭제됨                  |
