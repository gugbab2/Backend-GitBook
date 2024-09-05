# 트랜잭션(TRANSACTION)

## 1. 트랜잭션이란?

* 컴퓨터 프로그래밍에서 트랜잭션은 “더이상 분할이 불가능한 업무처리 단위” 를 의미한다.
* 이것은 하나의 작업을 위해 더이상 분할될 수 없는 명령들의 모음, \
  → 즉, 한꺼번에 수행되어야 할 일련의 연산모음을 의미한다.
* 다음의 상황을 살펴보자
  1. A는 매달 부모님에게 생활비를 송금받는다. 어느 날, 부모님이 A에게 생활비를 송금해 주기 위해 ATM을 이용했고 여느날 처럼 A의 계좌로 생활비를 송금했다.
  2. 그러나 모종의 이유로 부모님 계좌에서는 생활비가 차감되었는데, A의 통장에는 생활비가 입금되지 않았다.
* 위 상황은 ‘계좌이체’ 라는 상황을 풀어 쓴 것이다.
  * **‘계좌이체’ 라는 행위는 출금과 임금이라는 행위로 나뉘어져 있다. (만약 출금이 이루어졌는데, 입금이 안되어 있다면 치명적인 상황이다)**
* **따라서 두 과정은 동시에 실패하던지, 동시에 성공해야만 한다. (하나로 묶음으로 인해 Atomic 함을 의미한다)**
* **이 과정을 묶는 방법이 트랜잭션이다.**
* 아래와 같이 트랜잭션은 데이터 거래에 있어서 안정성을 확보하는 방법이다!
  * 데이터 처리 중 문제가 있다면 트랜잭션 이전 상황으로 돌리고,
  * 데이터 처리 중 문제가 없을때만 그 결과를 반영한다.

```sql
START TRANSACTION
    -- 이 블록안의 명령어들은 마치 하나의 명령어 처럼 처리됨
    -- 성공하던지, 다 실패하던지 둘중 하나가 됨.
    A의 계좌로부터 인출;
    B의 계좌로 입금;
COMMIT
```

## 2. MySQL 트랜잭션

* MySQL 에서 트랜잭션은 데이터를 바꾸는 일종의 작업 단위이다. \
  → 사실은 우리가 입력하는 모든 종류의 쿼리는 각각의 트랜잭션이다. \
  → MySQL 에서 자동으로 commit 을 하게 된다.
* 작업의 단위는 하나의 쿼리가 아니라, 사람이 정하는 기준이다.

## 3. 트랜잭션 특징

* 원자성
  * 원자성은 트랜잭션이 데이터베이스에 모두 반영되던가, 아니면 전혀 반영되지 않아야 한다는 것이다.
  * 트랜잭션은 사람이 설계한 논리적 단위로서, 트랜잭션은 작업단위 별로 나뉘어져야 데이터 싱크의 문제가 없다.
  * 원자성이 지켜지지 않는다면, 사람이 이해하기 힘든 상황이 펼쳐지고, 오작동 했을 시 원인을 찾기가 매우 어렵다.
* 일관성
  * 일관성은 트랜잭션의 처리 결과가 항상 일관적이어야 한다는 것을 의미한다.
  * 트랜잭션이 진행되는 동안 데이터가 변경되더라도, 업데이트 된 데이터로 진행되는 것이 아닌, 본래의 데이터 진행되어야 한다.
* 독립성
  * 독립성은 둘 이상의 트랜잭션이 동시에 실행될 때,
  * 어떤 하나의 트랜잭션이라도, 다른 트랜잭션의 연산에 끼어들 수 없다는 것을 의미한다.
* 영구성
  * 트랜잭션이 완료되었을 경우, 그 트랜잭션의 결과는 영구적으로 반영되어야 한다.