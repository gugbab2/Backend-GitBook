# 실무에서 외래키를 사용하지 않는 이유

## 외래키를 사용하는 제약조건.&#x20;

* 참조되는 테이블이 먼저 생성되어 있어야 하고, 기본키 이거나 유일해야 한다.\
  \-> 외래키는 데이터 무결성을 지키기 위해서 사용된다. \
  \-> 무결성이란 현재 데이터 값이 틀리지 않았다는 것을 의미한다.&#x20;
* 기존의 외래키로 사용되던 컬럼값을 변경하려 할 때, 연관된 모든 테이블의 값을 바꾸어 주어야 한다. \
  \-> 일단 너무 귀찮다. ..\
  \-> 예를 들어서 외래키를 연결하기 위해서는 연결하려는 값이 먼저 만들어져 있어야 하고 결론적으로 생성,수정,삭제할 때 순서를 생각해야 한다.&#x20;
* 외래키 설정에 따라 서브 테이블에서 데이터 수정, 변경이 이루어질 시 의도하지 않은 데이터의 변경이 이루어 질 수 있다.
* 외래키를 연결해두면 설정하고 생성, 수정, 삭제가 이루어 질때는 디비 내에서 무결성을 검사해야 하기 때문에, 디비 내 성능 저하가 된다는 이야기도 있다. \
  \-> 하지만 무결성을 체크하지 않으면 디비를 사용하는 의미가 없고 외래키가 아니어도 무결성을 체크해야 한다.&#x20;
*   무결성, 정합성을 희생시켜 개발 편의와 안정성, 확장성을 도모하기 위함이라고 할 수 있습니다.
