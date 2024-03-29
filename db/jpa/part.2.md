# 연관 관계 매핑 part.2

## 1. 연관관계 매핑시 고려사항 3가지

* 다중성 (@ManyToOne, @OneToMany @OneToOne, @ManyToMany)
* 단방향, 양방향
  * 테이블&#x20;
    * 외래 키 하나로 양쪽 조인 가능
    * 사실 방향이라는 개념이 없음
  * 객체
    * 참조용 필드가 있는 쪽으로만 참조 가능
    * 한쪽만 참조하면 단방향
    * 양쪽이 서로 참조하면 양방향\
      \-> 사실은 단방향이 두개인 것이다.\
      \-> 단방향 매핑으로 셋팅하고, 비지니스적으로 필요시 추가해야 한다.&#x20;
* 연관관계의 주인
  * 외래 키를 가지고 있는 Entity 에서 관리해야 한다. &#x20;

## 2. 다대일(권장하는 패턴)

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 17.37.50.png" alt="" width="375"><figcaption><p>단방향</p></figcaption></figure>

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 17.41.25.png" alt="" width="375"><figcaption><p>양방향</p></figcaption></figure>

* 가장 많이 사용하는 연관관계이다.
* 외래키가 있는 쪽이 연관관계의 주인

## 3. 일대다(권장하지 않음..)

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 17.43.58.png" alt="" width="375"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 18.00.30.png" alt="" width="375"><figcaption></figcaption></figure>

* 일대다 단방향은, 일(1)이 연관관계의 주인이 되는 패턴이다.
  * _**객체와 데이터베이스의 패러다임이 틀어짐으로 권장하지 않는 패턴이다..**_
* 단점&#x20;
  * **실무에서 테이블이 한두개가 아닌데, 특별한 케이스가 생겨난다면, 운영이 어려워진다..**&#x20;
  * **때문에, 다대일 단방향으로 설계하고, 필요하다면 양방향 설정을 해주는 패턴을 권장한다.** \
    **-> **_**만약 다대일 양방향을 사용하지 않더라도, 일대다 단방향으로 설정할바에는, 다대일 양방향이 운영 측면에서 유리하다.**_
* 일대일 양방향은 그냥 사용하지 마라 ;;\
  \-> JPA 스펙상 있는 것이 아닌, 억지로 만드는 패턴이다 ;;\
  \-> _**다대일 양방향을 사용하자.**_

## 4. 일대일(권장하는 패턴)

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 18.07.01.png" alt="" width="375"><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/스크린샷 2023-07-02 18.08.44.png" alt="" width="375"><figcaption></figcaption></figure>

* 일대일 관계는 그 반대도 일대일이다.
* 주 테이블이나, 대상 테이블 중 어느곳에도 외래키를 선택할 수 있다. \
  \-> _**외래 키에 데이터베이스 유니크 제약조건을 추가해야 한다.**_&#x20;

## 5. 다대다 (실무에서는 절대 사용해서는 안된다;;)

* 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다..
* 때문에, 연결 테이블을 추가해서, 일대다 / 다대일 관계로 풀어내야 한다.&#x20;
* 편리해 보이지만 실무에서는 사용하면 안된다..
  * 연결 테이블이 단순히 연결만 하고 끝나지 않는다.&#x20;
  * **주문시간, 수량 같은 데이터가 들어올 수 있다.** \
    **-> 다대다는 변경사항을 추가할 수 없다 ;;**
  * **결론은 연결 테이블을 Entity 로 승격해서 일대다 / 다대일 관계로 풀어가야 한다.**&#x20;
