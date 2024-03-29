# JPA

## 영속성

* 데이터를 생성한 프로그램이 종료되더라도 사라지지 않는 데이터의 특성을 의미한다. \
  \-> 메모리 상의 데이터를 파일 시스템, RDB 등을 활용해 영구적으로 저장하는 것을 영속성을 부여한다 말함
* 기본적으로 애플리케이션에서 데이터베이스에 데이터를 저장하는 방법은 다음과 같다.&#x20;
  * JDBC
  * Spring JDBC(jdbcTemplate)
  * Persistence Framework(JPA, Hibernate, Mybatis)
* JPA 에서 영속성은 영속성 컨텍스트에 접근하는 인터페이스인 EntityManager 를 통해 엔티티 영속성을 관리한다.&#x20;

## API 핵심 인터페이스&#x20;

### EntityManager

* 영속성 컨텍스트&#x20;
  * EntityManager(Session) 를 생성할 때마다 하나씩 만들어지며, DB 와 어플리케이션 사이에 위치한다.&#x20;
  * 프로세스&#x20;
    * 조회할 데이터(엔티티)가 영속성 컨텍스트에 존재하는지 확인
    * 데이터가 없으면 쿼리를 생성
    * 쿼리를 DB 에 전송
    * 결과값을 영속성 컨텍스트가 받음
    * 전달받은 데이터를 엔티티로 저장
    * 엔티티 인스턴스를 리턴&#x20;
