# 연관관계 매핑 기초

## 1. 예제 시나리오&#x20;

* 회원과 팀이 있다.&#x20;
* 회원은 하나의 팀에만 소속될 수 있다.&#x20;
* 회원과 팀은 다대일 관계이다.&#x20;

### 객체를 테이블에 맞추어 모델링&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-31 20.52.05.png" alt=""><figcaption></figcaption></figure>

```java
package hellojpa;

import javax.persistence.*;

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

    @Column(name = "TEAM_ID")
    private Long teamId;

    ...
}
```

```java
package hellojpa;

import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;

    private String name;

    ...
}
```

**객체간 연간관계가 없기 때문에, 테이블 설계에 의존된 코드를 작성하게 된다.** \
**(객체지향적인 코드가 아니다..)**&#x20;

```java
//조회
Member findMember = em.find(Member.class, member.getId());
//연관관계가 없음
Team findTeam = em.find(Team.class, team.getId());// Some code
```

### 문제점&#x20;

**객체를 테이블에 맞추어 데이터 중심으로 모델링하면, 협력 관계를 맺을 수 없다.**&#x20;

* 테이블은 외래 키로 조인을 사용해서 연관된 테이블을 찾는다.&#x20;
* 객체는 참조를 사용해서 연관된 객체를 찾는다.&#x20;
* 테이블과 객체 사이에는 큰 간격이 존재한다..&#x20;

## 2. 단방향 연관관계

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-31 20.58.50.png" alt=""><figcaption></figcaption></figure>

```java
package hellojpa;

import javax.persistence.*;

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private Long teamId;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
}
```

**참조를 통해서 연관관계 조회가 가능하다. (객체 그래프 탐색)**&#x20;

```java
//팀 저장
Team team = new Team();
team.setName("TeamA");
em.persist(team);

//회원 저장
Member member = new Member();
member.setName("member1");
member.setTeam(team); //단방향 연관관계 설정, 참조 저장
em.persist(member);

//조회
Member findMember = em.find(Member.class, member.getId());

//참조를 사용해서 연관관계 조회
Team findTeam = findMember.getTeam();
```

**연관관계 수정 또한 객체지향적으로 가능하다.**&#x20;

```java
// 새로운 팀B
Team teamB = new Team();
teamB.setName("TeamB");
em.persist(teamB);

// 회원1에 새로운 팀B 설정
member.setTeam(teamB);
```

## 3. 양방향 연관관계와 연관관계의 주인1 - 기본

**결론부터 이야기하자면, 일단 JPA 설계라는 것은 단방향 매핑으로 마무리가 되어야 한다!**

**단방향 매핑을 잘 하고 양방향은 필요할 때 추가하면 된다.**\
(테이블에 영향을 주지 않는다)

### 양방향 매핑&#x20;

외래키를 통해서 두개의 테이블이 연관관계를 맺는다면, 이 연관관계는 양방향으로 관계를 맺는다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-01 00.48.14.png" alt=""><figcaption></figcaption></figure>

```java
package hellojpa;

import javax.persistence.*;

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private Long teamId;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
}
```

```java
package hellojpa;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();

    ...
}
```

```java
//조회
Team findTeam = em.find(Team.class, team.getId());
int memberSize = findTeam.getMembers().size(); //역방향 조회
```

### 객체와 테이블이 관계를 맺는 차이&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-01 00.48.14.png" alt=""><figcaption></figcaption></figure>

#### 객체 연관관계 = 단방향 연관관계 2개&#x20;

* 회원 -> 팀 연관관계 1개(단방향)
* 팀 -> 회원 연관관계 1개(단방향)&#x20;
* **객체의 양방향 관계는 사실 양방향 관계가 아니라 서로 다른 단방향 관계 2개이다.**&#x20;
* 객체를 양방향으로 참조하려면 **단뱡향 연관관계 2개**를 만들어야 한다.&#x20;

#### 테이블 연관관계 1개&#x20;

* 회원 <-> 팀의 연관관계 1개(양방향)&#x20;
* 테이블은 외래 키 하나로 두 테이블의 연관관계를 관리한다.
* `MEMBER.TEAM_ID` 외래 키 하나로 양방향 연관관계를 가진다. (양쪽으로 조인이 가능하다)&#x20;

```sql
SELECT *
FROM MEMBER M
JOIN TEAM T ON M.TEAM_ID = T.TEAM_ID

SELECT *
FROM TEAM T
JOIN MEMBER M ON T.TEAM_ID = M.TEAM_ID
```

**결론적으로 둘 중 하나로 외래 키를 관리해야 한다.**&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-01 00.54.01.png" alt=""><figcaption></figcaption></figure>

### 연관관계 주인(Owner)&#x20;

#### 양방향 매핑 규칙

* 위 상황에서 "연관관계 주인" 이라는 개념이 등장한다.&#x20;
* 객체의 두 관계 중 하나를 연관관계 주인으로 지정해야 한다.&#x20;
* **연관관계의 주인만이 외래 키를 관리(등록, 수정) 한다.**&#x20;
* **주인이 아닌 쪽은 읽기만 가능하다.**&#x20;
* 주인이 아닌 쪽이 `mappedBy` 속성을 사용한다.&#x20;

#### 누구를 주인으로 해야 하는가?&#x20;

* 외래키가 있는 곳을 주인으로 정하자!&#x20;
* 여기서는 `Member.team` 이 연관관계의 주인이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-01 00.56.43.png" alt=""><figcaption></figcaption></figure>

## 4. 양방향 연관관계와 연관관계의 주인2 - 주의점&#x20;

### 양방향 매핑시 가장 많이 하는 실수&#x20;

연관관계의 주인의 값을 입력하지 않는다.&#x20;

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

//역방향(주인이 아닌 방향)만 연관관계 설정
team.getMembers().add(member);

em.persist(member);
```

### 양방향 매핑시 연관관계의 주인에 값을 입력해야 한다

순수한 객체 관계를 고려하면 항상 양쪽 다 값을 입력해야 한다.&#x20;

```javascript
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setName("member1");

team.getMembers().add(member);
//연관관계의 주인에 값 설정
member.setTeam(team); //**

em.persist(member);
```

<figure><img src="../../../.gitbook/assets/스크린샷 2025-09-01 01.21.55.png" alt=""><figcaption></figcaption></figure>

#### 만약 순수한 객체 관계를 고려하지 않고 연관관계의 주인에만 값을 입력하면 어떻게 될까?&#x20;

* 아래 코드에서는 디비에는 값이 들어가지만,&#x20;
* 메모리 내에서는 `team` 에 `member` 가 설정되지 않았기 때문에, `findTeam.getMembers()` 의 값이 없다.&#x20;

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);

// team.getMembers().add(member);

Team findTeam = em.find(Team.class, team. getId());  // 1차 캐시
List<Member> members = findTeam.getMembers();
for (Member m : members) {
    System.out.println("m.getUsername() = " + m.getUsername());
}
```

이 경우 2가지 문제가 발생한다.&#x20;

* **애플리케이션 코드 내에서 논리적 오류가 발생한다.**&#x20;
  * JPA 가 알아서 값을 입력해줄 것 같지만, 아니다..&#x20;
* **순수한 자바로 테스트 작성시 논리적 오류가 발생한다.**&#x20;

### **양방향 연관관계 주의점**&#x20;

#### 1. 순수한 객체 상태를 고려해서 항상 양쪽에 값을 설정하자.&#x20;

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);

team.getMembers().add(member);

Team findTeam = em.find(Team.class, team. getId());  // 1차 캐시
List<Member> members = findTeam.getMembers();
for (Member m : members) {
    System.out.println("m.getUsername() = " + m.getUsername());
}
```

#### 2. 연관관계 편의 메서드를 생성하자.&#x20;

`.setTeam()` 내부에 연관관계를 맺어주는 코드를 추가해 연관관계를 누락할 가능성을 줄여준다.&#x20;

```javascript
package hellojpa;

import javax.persistence.*;

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private Long teamId;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...

    // 연관관계 편의 메서드 
    // changeTeam 같이 의미있는 메서드 명을 사용하는 것도 방법이다. 
    public void setTeam(Team team) {
        this.team = team;

        team.getMembers().add(this);
    }
}
```

```java
Team team = new Team();
team.setName("TeamA");
em.persist(team);

Member member = new Member();
member.setUsername("member1");
member.setTeam(team);
em.persist(member);

// team.getMembers().add(member);

Team findTeam = em.find(Team.class, team. getId());  // 1차 캐시
List<Member> members = findTeam.getMembers();
for (Member m : members) {
    System.out.println("m.getUsername() = " + m.getUsername());
}
```

#### 3. 양방향 매핑시에 무한 루프를 조심하자. (예: toString(), lombok, JSON 생성 라이브러리)

양쪽에서 각각 서로를 호출하는 경우 무한 루프가 발생될 수 있다.&#x20;

```java
package hellojpa;

import javax.persistence.*;

@Entity
public class Member {
    @Id
    @GeneratedValue
    @Column(name = "MEMBER_ID")
    private Long id;

    @Column(name = "USERNAME")
    private String username;

//    @Column(name = "TEAM_ID")
//    private Long teamId;
    @ManyToOne
    @JoinColumn(name = "TEAM_ID")
    private Team team;

    ...
    
    @Override
    public String toString() {
        return "Member{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", team=" + team +
                '}';
    }
}
```

```java
package hellojpa;

import javax.persistence.*;
import java.util.ArrayList;
import java.util.List;

@Entity
public class Team {

    @Id
    @GeneratedValue
    @Column(name = "TEAM_ID")
    private Long id;
    private String name;

    @OneToMany(mappedBy = "team")
    private List<Member> members = new ArrayList<>();
    
    ...

    @Override
    public String toString() {
        return "Team{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", members=" + members +
                '}';
    }
}
```
