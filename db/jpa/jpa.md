# JPA 연관 관계 매핑

## 연관 관계 정의 규칙&#x20;

* 연관 관계를 매핑할 때, 생각해야 할 것은 크게 3가지가 있다.&#x20;
  * 방향 : 단방향, 양방향&#x20;
  * 연관 관계의 주인 : 양방향일 때, 연관 관계에서 관리주체&#x20;
  * 다중성 : N:1, 1:N, 1:1, N:M

## 방향 : 단방향, 양방향&#x20;

* 데이터베이스 테이블은 외래 키 하나로 양쪽 테이블 조인이 가능하다. \
  \-> 따라서 데이터베이스는 단방향, 양방향을 나눌 필요가 없다.&#x20;
* 그러나 **객체는 참조용 필드(ID)가 있는 객체만 다른 객체를 참조하는 것이 가능하다.** \
  **-> 때문에, 두 객체 사이에 하나의 객체만 참조용 필드를 갖고 참조하면 단뱡향** \
  **-> 두 객체 모두가 각각 참조용 필드를 갖고 참조하면 양방향 관계라고 한다.** \
  \-> 엄밀하게 양방향은 두 객체 모두 단방향 참조를 가지고 있는 상황을 말한다.&#x20;
* 해당 비지니스 로직에 맞게 선택했는데, 두 객체가 서로 단방향 참조를 했다면 양방향 연관관계라고 생각하면 된다.&#x20;

### 무조건 양방향 관계를 하면 쉽지 않을까?

* 객체 입장에서 양방향 매핑을 했을 때 오히려 복잡해 질 수 있다.&#x20;
* 예를 들어보자&#x20;
  * **일반적인 비지니스 어플리케이션에서 User 엔티티는 상당히 많은 연관관계를 맺는다.**&#x20;
  * **이런 경우 모든 엔티티를 양방향으로 설정하면 User 는 무쟈게 많은 테이블 연관관계를 맺게되어 복잡도가 올라간다!**
* 위에 예와 같은 상황 때문에, 연관 관계 설정 시 고민해야 한다.&#x20;
* 구분하기 쉬운 기준은 **기본적으로 단방향 매핑으로 하고 나중에 역방향 객체 탐색이 꼭 필요하다고 느낄 때 추가하는 것으로 잡으면 된다.**&#x20;

## 연관 관계의 주인

* 두 객체가 양방향 관계, 다시 말해 **단방향 관계 2개를 맺을 때 연관 관계의 주인을 정해야 한다.**&#x20;
* 연관 관계 주인을 지정하는 것은, 두 방향의 관계 중 제어의 권한(외래키를 비롯한 테이블 레코드 CRUD) 을 갖는 실질적인 관계가 어떤 것인지 JPA 에게 알려준다고 생각하면 된다. \
  \-> 연관 관계의 주인은 CRUD 를 할 수 있지만, 주인이 아니라면 조회만 가능하다.&#x20;
* **유용한 팁!**
  * **외래키가 있는 곳을 연관 관계의 주인으로 정하면 된다! 무조건!**&#x20;

### 왜 연관 관계의 주인을 지정해야 하는가?

* 두 객체(Board, Post)가 양방향 연관관계를 맺는다고 생각해보자.
* 해당 상황에서 Post 의 Board 를 다른 Board 로 수정하려 할 때 Post 에서 setBoard() 호출하는 것이 맞는지 Board 에서 setPost() 호출하는 것이 맞는지 헷갈린다. \
  \-> 두 객체 입장에서 각각의 방법 다 맞는 방법이기는 하다.&#x20;
* 그러나, 이렇게 객체에서 양방향 연관관계의 관리 포인트가 2개일 때는 테이블과 매핑을 담당하는 JPA 입장에서 혼란을 주게 된다. \
  \-> 즉 setPost() 를 할 때 FK 를 수정할지, setBoard() 를 할 때 FK 를 수정할 지 결정하기 어려운 것이다.&#x20;
* **때문에, 두 객체 사이의 연관 관계의 주인을 정해서 명확하게 Post 에서 Board 를 수정할 때만 FK 를 수정하겠다! 라고 정하는 것이다 .**

### **연관 관계의 주인만 제어하면 되는가?**

* 맞다!&#x20;
* 맞긴한데 데이터베이스만 생각했을 때 그렇고, 객체를 생각해보면 사실  둘 다 변경해주는 것이 좋다.\
  \-> 연관 관계의 주인이 아닌 곳에서도 변경!
* 왜냐면 두 참조를 사용하는 순수한 두 객체는 데이터 동기화를 해주어야 하기 때문이다.&#x20;

## 다중성&#x20;

### N:1

* 게시판과 게시글의 관계를 예로 들어보자
* 요구사항&#x20;
  * 하나의 게시판(1) 에는 여러개의 게시글(N) 을 작성할 수 있다.&#x20;
  * 하나의 게시글은 하나의 게시판에만 작성할 수 있다.&#x20;
  * 게시글과 게시판은 N:1 관계를 가진다.
* 데이터베이스를 기준으로 다중성을 결정했다. \
  \-> 즉, 외래키를 게시글(N)이 관리하는 일반적인 형태이다. \
  \-> 데이터베이스는 무조건 N 쪽에서 외래 키를 갖는다.&#x20;

```java
// 단방향 
@Entity
public class Post {
    @Id @GeneratedValue
    @Column(name = "POST_ID")
    private Long id;

    @Column(name = "TITLE")
    private String title;

    @ManyToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
    //... getter, setter
}

@Entity
public class Board {
    @Id @GeneratedValue
    private Long id;
    private String title;
    //... getter, setter
    
}

// 양방향 
@Entity
public class Post {
    @Id @GeneratedValue
    @Column(name = "POST_ID")
    private Long id;

    @Column(name = "TITLE")
    private String title;

    @ManyToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
    //... getter, setter
}

@Entity
public class Board {
    @Id @GeneratedValue
    private Long id;
    private String title;

    @OneToMany(mappedBy = "board")
    List<Post> posts = new ArrayList<>();
    //... getter, setter
}
```

### 1:N

* 위 와 달리, 연관 관계의 주인을 1 쪽에 둔 사례를 살펴보자.\
  \-> 실무에서는 1:N 단방향을 거의 쓰지 않도록 하자 ..

```java
@Entity
public class Post {
    @Id @GeneratedValue
    @Column(name = "POST_ID")
    private Long id;

    @Column(name = "TITLE")
    private String title;
  //... getter, setter
}

@Entity
public class Board {
    @Id @GeneratedValue
    private Long id;
    private String title;

    @OneToMany
    @JoinColumn(name = "POST_ID") //일대다 단방향을 @JoinColumn필수
    List<Post> posts = new ArrayList<>();
    //... getter, setter
}

//...
Post post = new Post();
post.setTitle("가입인사");

entityManager.persist(post); // post 저장

Board board = new Board();
board.setTitle("자유게시판");
board.getPosts().add(post);

entityManager.persist(board); // board 저장
//...
```

* 위와 같은 시나리오 대로 살펴보면, Post 를 저장할 때 멀쩡하게 Insert 쿼리가 동작한다.&#x20;
* 하지만, Board 를 저장할 때는, Board 를 Insert 하는 쿼리가 나간 후, Post 를 update 하는 쿼리가 나간다. \
  \-> 1차 캐시를 사용하기 때문이다..

#### 치명적인 단점&#x20;

* 1 만 수정한 것 같은데, 다른 수정이 생기는 쿼리가 발생하는 것&#x20;
  * Board 를 저장했는데, 왜 Post 가 수정이 되지?
  * update 쿼리 때문에, 성능상 이슈는 그렇게 크지는 않다...

### 1:1

* 주 테이블에 왜리키를 넣을 수도 있고, 대상 테이블에 외래키를 넣을 수도 있다.&#x20;
*
