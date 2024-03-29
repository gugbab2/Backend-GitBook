# ORM vs SQL Mapper

* ORM, SQL Mapper 모두 데이터베이스와 애플리케이션 간의 상호작용을 돕는 도구이다.&#x20;

## ORM

* 특징
  * ORM 은 객체 지향 프로그래밍 언어의 객체와 관계형 데이터베이스의 테이블 간 매핑을 자동으로 처리한다.&#x20;
  * 개발자는 SQL 쿼리를 직접 작성하지 않고도 객체를 통해서 데이터베이스에 접근할 수 있다.&#x20;
  * ORM 은 일반적으로 객체 지향 프로그래밍 언어의 문법과 구조를 따르기 때문에, 개발자들이 더 직관적으로 코드를 작성할 수 있도록 해준다.
  * ORM 은 데이터베이스의 테이블과 객체 간의 관계를 관리하며, 객체의 CRUD 작업을 처리하는 다양한 메서드를 제공한다.&#x20;
* 장점
  * 객체지향적인 코드 작성이 가능하다. \
    \-> 쿼리에 집중하지 않기 때문에, 직관적이며 비지니스 로직에 충실한 코드 작성이 가능하다.&#x20;
  * 재사용성 및 유지보수 편의가 증가한다.\
    \-> DBMS 벤더에 독립적이다.&#x20;
  * ORM 은 보안적인 측면에서 SQL 인젝션을 방지하는데 도움을 줄 수 있다.&#x20;
* 단점
  * ORM 은 객체와 데이터베이스와 매핑하기 위해 많은 리소스를 사용할 수 있다. \
    \-> 복잡한 쿼리나 대량의 데이터를 다룰 경우 성능 저하가 발생할 수 있다.&#x20;
  * 일부 복잡한 데이터베이스 관계를 표현하기가 어려울 수 있다. \
    \-> 객체지향 모델과 관계형 데이터베이스 간에는 차이가 있기 때문에, 매핑이 어려울 수 있다.
  * 데이터 모델의 복잡도에 비례해 난이도가 증가한다.&#x20;

## SQL Mapper

* 특징
  * SQL Mapper 는 개발자가 직접 쿼리를 작성하고 실행하는 방식으로 데이터베이스와 상호작용한다.&#x20;
  * SQL Mapper 는 쿼리를 객체로 매핑하지 않고, 데이터베이스 결과를 다루기 위한 데이터 구조로 매핑한다.&#x20;
  * SQL Mapper 를 사용하면 개발자가 원하는 대로 세밀하게 SQL 제어할 수 있다.&#x20;
* 장점
  * 개발자가 직접 쿼리를 작성해 원하는 방식으로 데이터를 다룰 수 있다.&#x20;
  * 복잡한 쿼리나 대량의 데이터를 다룰 때 성능이 좋을 수 있다. \
    \-> 개발자가 최적화된 쿼리를 작성할 수 있다.&#x20;
  * SQL Mapper 는 비교적 단순한 매핑작업을 수행하기 때문에, 복잡한 데이터베이스 관계를 다루기에 적합하다.
* 단점
  * SQL Mapper 는 개발자가 직접 쿼리를 작성하기에 코드가 더 복잡해 질 수 있다...\
    \-> 이건 단점은 아닌듯 ..
  * SQL Mapper 는 데이터베이스와 밀접하게 결합되어, 데이터베이스 변경 시 코드 수정이 필요할 수 있다.&#x20;
  * SQL Mapper 는 객체지향 프로그래밍과 멀리 떨어져 있기에, 객체지향의 장점을 살리기가 어렵다.&#x20;
