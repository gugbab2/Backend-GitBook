# 연관 관계 매핑 part.1

## 1. 연관 관계 정의 규칙

* 연관 관계를 매핑할 때, 생각해야 할 것은 크게 3가지가 있다.
  * 방향 : 단방향, 양방향
  * 연관 관계의 주인 : 양방향일 때, 연관 관계에서 관리주체
  * 다중성 : N:1, 1:N, 1:1, N:M

## 2. 연관관계가 필요한 이유

* '객체지향 설계의 목표는 자율적인 객체들의 협력 공동체를 만드는 것이다.'
* **객체를 테이블에 맞추어서 데이터 중심으로 모델링하면, 협력 관계를 만들 수 없다...**\
  **-> 테이블은 외래 키로 조인을 사용해 연관된 테이블을 찾는다.**\
  **-> 객체는 참조를 통해서 연관된 객체를 찾는다.**

<figure><img src="../../../.gitbook/assets/스크린샷 2023-07-02 14.29.29.png" alt="" width="375"><figcaption><p>table focosing modeling</p></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2023-07-02 14.30.12.png" alt="" width="375"><figcaption><p>object focosing modeling</p></figcaption></figure>

## 3. 단방향 연관관계

* 기존 코드들은 디비 쿼리에 종속적인 엔티티 설계를 하여, 객체 지향적인 코드를 작성하지 못했다..
* 하지만 JPA 는 어노테이션을 통해 연관관계를 맺을 수 있다.
* 아래 코드는 단방향 연관관계 매핑 코드이다.

```java
//    @Column(name = "TEAM_ID")
//    private Long teamId;
@ManyToOne
@JoinColumn(name = "TEAM_ID")
private Team team;

//=======================================
// 저장 
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);

em.persist(member);

// 조회
Member findMember = em.find(Member.class, member.getId());
Team findTeam = findMember.getTeam();
System.out.println("findTeam.getName() = " + findTeam.getName());
```

## 4. 양방향 연관관계와 연관관계의 주인 - 기본

<figure><img src="../../../.gitbook/assets/스크린샷 2023-07-02 14.48.16.png" alt="" width="375"><figcaption></figcaption></figure>

* 테이블간의 연관관계는 한 테이블에 FK 만 가지게 되면 JOIN 을 통해 어느 테이블에서든 조회가 가능하다.\
  \-> 방향의 개념 자체가 없다.
* 하지만 객체의 연관관계는 레퍼런스를 통해서 조회를 할 수 있기 때문에, 해당 클래스 매개변수로 상대 레퍼런스를 가지고 있어야 한다.\
  \-> 방향의 개념이 있다.
* mappedBy 는 처음에는 이해하기 어렵다..
  * 객체 연관관계 = 2개\
    \-> **방향의 개념이 있다!!!(레퍼런스O)**
    * 회원 -> 팀 연관관계 1개(단방향)
    * 팀 -> 회원 연관관계 1개(단방향)
    * **객체의 양방향 관계는 사실 양방향 관계가 아닌 단뱡향이 2개인 것이다!**
  * 테이블 연관관계 = 1개\
    \-> **방향의 개념이 없다...(레퍼런스X)**
    * 외래키(TEAM\_ID)

```java
@OneToMany(mappedBy = "team")
private List<Member> members = new ArrayList<>();
```

### 4-1. 연관관계의 주인

**-> 둘 중 하나로 외래 키를 관리해야 한다.**

<figure><img src="../../../.gitbook/assets/스크린샷 2023-07-02 15.03.22.png" alt="" width="375"><figcaption></figcaption></figure>

* 양방향 매핑 규칙
  * **연관관계의 주인만이 외래 키를 관리(등록, 수정)**\
    **-> \_외래키가 있는 곳을 주인으로 정해라!!!**\_
  * **주인이 아닌 쪽은 읽기만 가능**
  * 주인은 mappedBy 속성 사용 X

## 5. 양방향 연관관계와 연관관계의 주인 - 주의점, 정리

### 5-1. 양방향 매핑 시 가장 많이 하는 실수 (연관관계의 주인에 값을 입력하지 않음)

* **항상 연관관계의 주인이 값을 입력해야 한다!!**

```java
// 비정상 케이스
Member member = new Member();        // 연관관계 주인인 member 에서 값을 입력하지 않음 ;;
member.setUsername("member1");
em.persist(member);

Team team = new Team();
team.setName("TeamA");
team.getMembers().add(member);
em.persist(team);

// 정상 케이스
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);                // 연관관계 주인인 member 에서 값을 입력한다 !!
em.persist(member);
```

* **하지만, 객체의 입장에서 생각을 해보면, 양쪽 모두에서 값을 입력해주어야 한다!**\
  **-> em.flush() / em.clear() 를 입력하지 않으면, 1차 캐시에서 가져오는 것이기 때문에, findTeam.getMembers() 의 값을 찾아올 수 없다!**
* 또한, 순수한 자바 클래스에서 테스트 코드를 작성할 때를 생각하면 양쪽 객체에 값을 세팅하는 것이 맞다!
* _**순수 객체 상태를 고려해서 항상 양쪽에 값을 설정하자!**_

```java
 Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);

// 1차 캐시에서 값을 가져오게 된다. 
//em.flush();    
//em.clear();

// 객체의 입장에서 생각을 하면 team.setMembers 를 하지 않았기 때문에, 문제가 된다. 
Team findTeam = em.find(Team.class, team.getId());
List<Member> members = findTeam.getMembers();

for (Member m : members) {
    System.out.println("m.getUsername() = " + m.getUsername());
}
```

* 하지만 사람인지라 까먹을 수 있기 때문에, 다음과 같은 관례를 고려하자!\
  **(연관관계 편의 메서드)**\
  **-> \_연관관계 편의 메서드는 둘 중 하나의 메서드에서만 만들어야 한다!**\_

```java
public void changeTeam(Team team) {
    this.team = team;
    team.getMembers().add(this);
}
```

### 5-2. 양방향 매핑시에 무한 루프를 조심하자!!

* toSring(), lombok, JSON 생성 라이브러리 를 조심하자!

### 5-3. 양방향 매핑 정리

* **프로젝트 초기에 테이블 설계 시, 단방향 매핑만으로 이미 연관관계 매핑은 완료 되어야 한다.**\
  _**-> 나중에 개발하다가, 역방향으로 탐색할 일이 많아질 때(JPQL) 그 때 연관관계를 추가해라!**_
* 단방향 매핑을 잘 하고, 양방향은 필요할 때 추가해도 된다!
* 연관관계의 주인은 비지니스 로직을 기준으로 정하는 것이 아닌, _**외래 키의 위치를 기준으로 정해야 한다!**_
