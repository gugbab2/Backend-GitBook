# Collection Pattern 적용

### [CQS](https://en.wikipedia.org/wiki/Command%E2%80%93query\_separation)

CRUD를 중요한 특징에 따라 구분하면 Command와 Query 둘로 나눌 수 있다.

1. **Command** → Create, Update, Delete ⇒ **상태가 변함. 안전하지 않음**.
2. **Query** → Read ⇒ **상태가 변하지 않음. 안전함**. 분산, 캐시 등이 수월함.

### HTTP Method

Collection Pattern과 HTTP Method를 이용해 CRUD를 표현할 수 있다.

1. GET → Read
2. POST → Create&#x20;
3. PUT, PATCH → Update, PUT Override 를 하기 때문에, 생성 할 수도 있지만 우리는 그냥 수정이라 생각하자
4. DELETE → Delete

#### 게시물

1. `GET /posts` → 게시물 목록 (Read, Collection) → List (습관적인 표현 중 하나)
2. `GET /posts/{id}` → 게시물 상세 (Read, Element) → Detail (습관적인 표현 중 하나)
3. `POST /posts` → 게시물 생성 (Create) → Post ID는 서버에서 생성함.
4. `PUT 또는 PATCH /posts/{id}` → 게시물 수정 (Update, Element)
5. `DELETE /posts/{id}` → 게시물 삭제 (Delement, Element)

종종 Bulk(여러개) update, Bulk delete 등을 하기도 함. \
\-> 이럴 때는 Collection을 활용하고, API 스펙 문서에 정확히 기록.(문서화는 정말 중요하다!!)\
\-> 예를들어, JSON 형식으로 아래와 같이 여러개의 ID 를 전달할 수 있다.\
"Ids" : \[1,2,3,4]

#### 댓글

그냥 바로 comments로 시작하는 경우:

1. `GET /comments` → 전체 댓글 목록
2. `GET /comments?post_id={post_id}` → 특정 게시물에 대한 댓글 목록
3. `GET /comments/{id}` → 댓글 상세
4. `POST /comments` → 댓글 생성 ⇒ 어떤 게시물에 대해? → Body에 Post ID 정보를 담아줘야 함.
5. `POST /comments?post_id={post_id}` → 특정 게시물에 대한 댓글 생성
6. `PUT 또는 PATCH /comments/{id}` → 댓글 수정
7. `DELETE /comments/{id}` → 댓글 삭제

특정 게시물 아래로 표현하는 경우:

1. `GET /posts/{post_id}/comments` → 특정 게시물에 대한 댓글 목록
2. `GET /posts/{post_id}/comments/{id}` → 특정 게시물에 달린 특정 댓글 상세
3. `POST /posts/{post_id}/comments` → 특정 게시물에 대한 댓글 작성
4. `PUT 또는 PATCH /posts/{post_id}/comments/{id}` → 특정 게시물에 달린 특정 댓글 수정
5. `DELETE /posts/{post_id}/comments/{id}` → 특정 게시물에 달린 특정 댓글 삭제

#### 로그인/로그아웃

로그인과 로그아웃을 어떻게 리소스로 표현할 수 있을까? 이럴 때는 추상적인 개념인 “세션”을 도입하곤 한다.

1. `POST /session` → 세션 생성 = 로그인
2. `DELETE /session` → 세션 파괴 = 로그아웃

덤으로…

1. `GET /session` → 세션 확인 → 내 정보 확인?
2. `GET /users/me` → User ID를 me라고 쓰면, 현재 사용자의 User ID로 처리하게 정하고, API 스펙 문서에 기록.
