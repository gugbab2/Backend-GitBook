# 일급 컬렉션(First Class Collection) 소개와 사용해야하는 이유

> 참고 링크&#x20;
>
> [https://jojoldu.tistory.com/412](https://jojoldu.tistory.com/412)

## 일급 컬렉션이란?&#x20;

일급 컬렉션이란 단어는 소트웍스 앤솔로지의 객체지향 생활체조 파트에서 언급되었다.&#x20;

설명이 너무 어려워 간단히 설명하면, 아래의 코드를&#x20;

```java
Map<String, String> map = new HashMap<>();
map.put("1", "A");
map.put("2", "B");
map.put("3", "C");
```

아래와 같이 Wrapping 하는 것을 이야기한다.&#x20;

```java
public class GameRanking {

    private Map<String, String> ranks;

    public GameRanking(Map<String, String> ranks) {
        this.ranks = ranks;
    }
}
```

**`Collection` 을 `Wrapping` 하면서, 그 외 다른 멤버 변수가 없는 상태를 일급 컬렉션이라 한다.**&#x20;

Wrapping 함으로써 다음과 같은 장점을 가지게 된다.&#x20;

1. **비지니스에 종속적이 자료구조**&#x20;
2. **Collection 의 불변성을 보장**&#x20;
3. **상태와 행위를 한곳에서 관리**&#x20;
4. **이름이 있는 컬렉션**&#x20;

## 장점1 - 비지니스에 종속적인 자료구조

예를 들어 당므과 같은 조건으로 로또 복권 게임을 만든다고 가정하자.&#x20;

로또 복권은 다음의 조건을 가지고 있다.&#x20;

* 6개의 번호가 존재&#x20;
  * 보너스 번호는 이번 예제에서 제외&#x20;
* 6개의 번호는 서로 중복되지 않아야 함&#x20;

일반적으로 이런 일은 서비스 메서드에서 진행한다. \
그래서 구현을 해보면 아래와 같은 코드가 된다.&#x20;

<figure><img src="../../.gitbook/assets/image (186).png" alt=""><figcaption></figcaption></figure>

서비스 메서드에서 비지니스 로직을 처리했다. \
이럴 경우 큰 문제가 있다.&#x20;

**로또 번호가 필요한 모든 장소에서 검증로직이 들어가야만 한다.** \
**(현재 코드에서 로또 번호 생성과 검증은 다른 라인에서 이루어진다)**&#x20;

* `List<Long>` 으로 된 데이터는 모두 검증 로직이 필요할까?&#x20;
* 신규 입사자분들은 어떻게 이 검증로직이 필요한지 알 수 있을까?&#x20;

등등 **모든 코드와 도메인을 알고 있지 않다면** 언제든 문제가 발생할 여지가 있다.&#x20;

그렇다면 이 문제를 어떻게 깔끔하게 해결할 수 있을까?&#x20;

* 6개의 숫자로만 이루어져야 하고
* 6개의 숫자는 서로 중복되지 않아야 한다.&#x20;

이런 자료구조는 없을까? \
없으니 직접 만들면 된다.&#x20;

아래와 같이 **해당 조건으로만 생성할 수 있는 자료구조**를 만들면 위에서 언급한 문제들이 모두 해결된다.&#x20;

그리고 이런 클래스를 우린 **일급 컬렉션**이라고 부른다.

<figure><img src="../../.gitbook/assets/image (187).png" alt=""><figcaption></figcaption></figure>

이제 로또 번호가 필요한 모든 로직은 이 일급 컬렉션만 있으면 된다. \
&#xNAN;**(로또 번호 생성 시 검증이 자동으로 이루어진다)**&#x20;

<figure><img src="../../.gitbook/assets/image (188).png" alt=""><figcaption></figcaption></figure>

**비지니스에 종속적인 자료구조가 만들어져, 이후 발생할 문제가 최소화되었다.**&#x20;

## 장점2 - 불변&#x20;

일급 컬렉션은 컬렉션의 불변을 보장한다.&#x20;

여기서 `final` 을 사용하면 안되나요? 라고 할 수 있다. \
하지만, Java 의 `final` 은 정확히 불변을 만드는 것이 아닌, **재할당만 금지**한다.&#x20;

아래 테스트 코드를 참고해보자.&#x20;

```java
@Test
public void final도_값변경이_가능하다() {
    //given
    final Map<String, Boolean> collection = new HashMap<>();

    //when
    collection.put("1", true);
    collection.put("2", true);
    collection.put("3", true);
    collection.put("4", true);

    //then
    assertThat(collection.size()).isEqualTo(4);
}
```

이를 실행해보면 `Map` 에 값을 추가해도 막을 수 없다.&#x20;

결론적으로 `collection` 은 비어있는 `HashMap` 으로 선언되었음에도 값이 변경될 수 있다는 것이다.&#x20;

<figure><img src="../../.gitbook/assets/image (189).png" alt=""><figcaption></figcaption></figure>

추가로 테스트해보자.&#x20;

```java
@Test
public void final은_재할당이_불가능하다() {
    //given
    final Map<String, Boolean> collection = new HashMap<>();

    //when
    collection = new HashMap<>();

    //then
    assertThat(collection.size()).isEqualTo(4);
}
```

이 코드는 바로 컴파일 에러가 발생한다.&#x20;

`final` 로 할당된 코드에 재할당할순 없기 때문이다.&#x20;

<figure><img src="../../.gitbook/assets/image (190).png" alt=""><figcaption></figcaption></figure>

확인한 것처럼 Java 의 `final` 은 재할당만 금지한다. \
이외에도 `member.setAge(10)` 과 같은 코드 역시 작동해버리니, 불변이라고 볼 수 없다.&#x20;

요즘과 같이 소프트웨어 규모가 커지고 있는 상황에서 불변 객체는 매우 중요하다. \
각각의 객체들이 절대 값이 바뀔일이 없다는게 보장되면 그만큼 코드를 이해하고 수정하는데 사이드 이팩트가 최소화되기 때문이다.&#x20;

Java 에서는 `final` 로 그 문제를 해결할 수 없기 때문에, 일급 컬렉션(First Class Collection) 과 래퍼 클래스(Wrapper Class) 등의 방법으로 해결해야만 한다.&#x20;

그래서 아래와 같이 컬렉션의 값을 변경할 수 있는 메서드가 없는 컬렉션을 만들면 불변 컬렉션이 된다.&#x20;

<figure><img src="../../.gitbook/assets/image (191).png" alt=""><figcaption></figcaption></figure>

이 클래스는 생성자와 `getAmountSum()` 외의 다른 메서드가 없다. \
즉, 이 클래스의 사용법은 새로 만들거나 값을 가져오는 것 뿐이다. \
`List` 라는 컬렉션에 접근할 수 있는 방법이 없기 때문에 값을 변경/추가가 안된다.&#x20;

이렇게 일급 컬렉션을 사용하면, 불변 컬렉션을 만들 수 있다.&#x20;

## 장점3 - 상태와 행위를 한 곳에서 관리&#x20;

일급 컬렉션의 세번째 장점은 **값과 로직이 함께 존재한다는 것이다.**&#x20;

예를 들어 여러 Pay 들이 모여있고, NaverPay 금액의 합이 필요하다고 가정해보자. \
일반적으로는 아래와 같이 작성한다.&#x20;

* `List` 에 데이터를 담고&#x20;
* `Service` 혹은 `Util` 클래스에서 필요한 로직 수행&#x20;

<figure><img src="../../.gitbook/assets/image (192).png" alt=""><figcaption></figcaption></figure>

이 상황에서 문제가 있다. \
**결국 `Pays` 라는 컬렉션과 계산 로직은 서로 관련이 있는데, 이를 코드로 표현이 안된다.**&#x20;

`Pay` 타입의 상태에 따라 지정된 메서드에서만 계산되길 원하는데, 현재 상태로는 강제할 수 있는 수단이 없다.&#x20;

지금은 `Pay` 타입이 `List` 라면 사용될 수 있기 때문에, 히스토리를 모르는 사람들은 실수할 여지가 많다.&#x20;

* 똑같은 기능을 하는 메서드를 중복으로 생성할 수 있다.&#x20;
  * 히스토리가 관리 안된 상태에서 신규화면이 추가되어야 할 경우 계산 메서드가 있다는 것을 몰라 다시 만드는 경우가 빈번하다.&#x20;
  * 만약 기존 화면의 계산 로직이 변경 될 경우, 신규 인력은 2개의 메서드의 로직을 다 변경해야 하는지, 해당 화면만 변경해야 하는지 알 수 없다.&#x20;
  * 관리 포인트가 증가할 확률이 높다.&#x20;
* 계산 메서드가 누락될 수 있다.&#x20;
  * 리턴 받고자 하는 것이 `Long` 타입의 값이기 때문에, 꼭 이 계산식을 써야한다고 강제할 수 없다.&#x20;

결국 **네이버페이 총 금액을 뽑기 위해서 이렇게 해야한다는 계산식을 컬렉션과 함께 두어야 한다.**\
만약 네이버페이 외에 카카오페이의 총 금액도 필요하다면 코드의 복잡성이 높아질 가능성이 높다.&#x20;

그래서 이 문제 역시 일급 컬렉션으로 해결한다.&#x20;

```java
public class PayGroups {
    private List<Pay> pays;

    public PayGroups(List<Pay> pays) {
        this.pays = pays;
    }

    public Long getNaverPaySum() {
        return pays.stream()
                .filter(pay -> PayType.isNaverPay(pay.getPayType()))
                .mapToLong(Pay::getAmount)
                .sum();
    }
}
```

만약 다른 결제 수단들의 합이 필요하다면 아래와 같이 람다식으로 리팩토링 가능&#x20;

```java
public class PayGroups {
    private List<Pay> pays;

    public PayGroups(List<Pay> pays) {
        this.pays = pays;
    }

    public Long getNaverPaySum() {
        return getFilteredPays(pay -> PayType.isNaverPay(pay.getPayType()));
    }

    public Long getKakaoPaySum() {
        return getFilteredPays(pay -> PayType.isKakaoPay(pay.getPayType()));
    }

    private Long getFilteredPays(Predicate<Pay> predicate) {
        return pays.stream()
                .filter(predicate)
                .mapToLong(Pay::getAmount)
                .sum();
    }
}
```

**이렇게 `PayGroups` 라는 일급 컬렉션이 생김으로 상태와 로직이 한곳에서 관리 된다.**&#x20;

<figure><img src="../../.gitbook/assets/image (195).png" alt=""><figcaption></figcaption></figure>

## 장점4 - 이름이 있는 컬렉션&#x20;

마지막 장점은 컬렉션에 이름을 붙일 수 있다는 것이다. \
이 장점에 대해서는 크게 메리트를 못 느낄수 있지만, \
도메인의 이해 측면에서는 장점이라고 볼 수 있다.&#x20;

같은 Pay 들의 모임이지만 네이버페이 List 와 카카오페이 List 는 다르다. \
그렇다면 이 둘을 구분하려면 어떻게 해야할까? \
가장 흔한 방법은 변수명을 다르게 하는 것이다.&#x20;

<figure><img src="../../.gitbook/assets/image (196).png" alt=""><figcaption></figcaption></figure>

위 코드의 단점은 아래와 같다.&#x20;

* 검색의 어려움&#x20;
  * 네이버페이 그룹이 어떻게 사용되는지 검색 시 변수명으로만 검색할 수 있다.&#x20;
  * 이 상황에서 검색은 거의 불가능하다.&#x20;
  * 네이버페이의 그룹이라는 뜻은 개발자마다 다르게 지을 수 있다.&#x20;
  * 네이버페이 그룹은 어떤 검색어로 검색이 가능할까?&#x20;
* 명확한 표현이 불가능&#x20;
  * 변수명에 불과하기 때문에 의미를 부여하기가 어렵다.&#x20;
  * 이는 개발팀/운영팀간의 의사소통시 보편적인 언어로 사용하기가 어렵다.&#x20;
  * 중요한 값임에도 이를 표현할 명확한 단어가 없다.&#x20;

이 문제 역시 일급 컬렉션을 해결할 수 있다.&#x20;

네이버페이 그룹과 카카오페이 그룹 각각의 일급 컬렉션을 만들면 이 컬렉션 기반으로 용어사용과 검색을 하면 된다.&#x20;

<figure><img src="../../.gitbook/assets/image (197).png" alt=""><figcaption></figcaption></figure>
