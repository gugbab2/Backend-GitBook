# Collection Pattern

## Collection Pattern

\-> 통상적으로 **RESTFUL 이라는 기준이 된다.**\
**-> Collection : 서버가 관리하는 리소스 디렉토리**

대부분의 경우, 여러 리소스를 하나의 그룹으로 묶을 수 있다. 예를 들어 게시물의 경우, 100개의 게시물을 하나의 게시물 그룹으로 묶어서 표현하는 게 훨씬 유용하다.

Collection Pattern을 쓰지 않는 경우:

* `/the-first-post`
* `/test-post`

Collection Pattern을 쓰는 경우:

* `/posts` ⇒ 모든 게시물을 하나의 URI로 표현(게시물 목록으로도 활용 가능). 일반적으로 복수형을 사용한다.
* `/posts/the-first`
* `/posts/test`
* `/posts/{id}` 또는 `/posts/:id` 등으로 일반적인 형태로 쓸 수 있다. 여기서 {id}를 {post\_id}나 {postId} 등으로 표기할 수도 있다. 여기서는 그냥 Post ID라고 부를 예정.

**리소스의 ID 는 URI 전체가 되는 것이다.**\
**->** Post ID`{id}`**가 리소스의 ID 가 될 수는 없다..!**

* Resource ID = URI = URL
* Post ID = Resource ID를 구성하는 요소 중 하나. posts 그룹 내 식별자(ID).

그룹명은 복수형으로 사용!!

* `/posts/1` ⇒ 복수형으로 그룹명을 지정하고, 그 아래에서 개별 아이템을 지정.
* `/post/1` ⇒ 흔히 하는 실수.

특정 게시물에 댓글이 달리는 상황이라면, 디렉터리처럼 구성할 수 있다.

* `[/posts/{post_id}][/comments]` ⇒ Collection
  * `[/comments][?post_id={post_id}]` 형태로 써줘도 된다. 이렇게 쓰면 Post ID를 이용해 필터링한다는 의미가 됨. URI에서 드러나는 의미는 미묘하게 다르지만, 구현하는 코드나 결과는 사실상 동일하다.
* `[/posts/{post_id}][/comments/{id}]` ⇒ Element. 여기서 {id}는 {comment\_id}를 의미함.
  * `[/comments/{id}]` 형태로 간단히 구성해도 된다. 이렇게 할 경우, URI만 봐서는 어떤 게시물에 대한 댓글인지 알 수 없다. => 댓글 삭제 등 간단한 작업 요청 상황이라면 오히려 좋을 수도 있다.

특정 게시물을 고치는 페이지를 표현하고 싶다면, 마찬가지로 구성할 수 있다.

* `[/posts/{id}][/edit]` ⇒ “Edit Page”라는 리소스.
  * 마찬가지로 `/edit?post_id={post_id}` 형태로 써줄 수도 있다. 다만, `/comments/{id}/edit` 같은 게 도입될 경우 `/edit?comment_id={comment_id}` 같이 구분해서 처리하는 것이 더 복잡할 수 있다.
  * REST API(B/E)의 경우, 페이지만 표현할 일은 거의 없다. 이런 표현은 F/E에 맡기고, 그냥 `/items`, `/items/{id}` 같은 URL만 쓰도록 제약해도 무방하다.\
    \-> REST API 서버를 만드는데, 컬렉션/엘리먼트 이상의 구조를 만들게 된다면 뭔가 RESTFUL 하지 못하다는 반증일 수 있다!!

그룹이 아닌 경우라면 그냥 단수형을 써도 된다.

* `/session` ⇒ 세션은 하나만 유지된다. 다른 세션을 참고할 일도 없다. `/sessions/1` 같은 접속이 있으면 안 된다.
* `/edit?blahblah` ⇒ 위에서 언급한 케이스.
  * 만약 편집 페이지의 유형을 Edit ID 따위로 표현하고 싶다면, `/edits/post`, `/edits/comment` 같이 컬렉션 패턴과 복수형을 사용할 수 있다.
  * 하지만 이렇게 하고 싶다면 edit보다는 그냥 page란 리소스를 사용하는 걸 추천. Page란 리소스, 그리고 Page ID란 모양이 훨씬 명확하다. 이렇게 하면 `/pages/edit-post`, `/pages/edit-comment` 같이 훨씬 자연스럽게 표현할 수 있게 된다.
