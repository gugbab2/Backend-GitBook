# 레거시 코드

## 레거시 코드

* 이해할 수 없고 수정하기도 힘든 코드를 지칭하는 속어처럼 사용될 때가 많다.
* 레거시 코드는 모든 개발자가 극복해야 할 난제
* 왜 시스템은 부패해가는 것일까? 왜 시스템은 깨끗한 상태에 머물러 있지 않을까?
* 때로는 우리는 고객에게 책임을 돌린다.

## Anemic Domain Model&#x20;

* 빈약한 도메인 모델
* 객체지향에서 말하는 오브젝트는 상태(state)와 행위(behavior)로 구성되어 있어야 한다.
* **자바엔터프라이즈 개발에서 흔히 사용되는 방식은 단지 상태 값, 즉 데이터만 가지는 데이터홀더 개념의 단순 오브젝트**
* **대부분의 서버사이드 아키텍처라고 제시되는 구조가 빈약한 도메인모델의 사용을 부추기고 있다는 점이 문제였다.**
* **대부분의 자바 엔터프라이즈 아키텍처가 가지고 있는 이런 구조적인 한계들은 결국 과도한 서비스 레이어의 사용을 부추긴다.**

```java
public class Box {
    private int height;
    private int width;

    public int getHeight() {
        return height;
    }

    public void setHeight(final int height) {
        this.height = height;
    }

    public int getWidth() {
        return width;
    }

    public void setWidth(final int width) {
        this.width = width;
    }
}
```

## Business Object

* 비즈니스가 분산되어 구현된 오브젝트
* 동작을 실행하거나 관리하는 클래스
* 클라이언트 데이터에 대한 요청을 DAO에 호출하는 역할도 한다.
* 모든 기능을 해당 소프트웨어 개체로 분리한다.

## Big Service Layer

* **이런 형태의 거대한 서비스 레이어(big service layer) 형태는 객체지향의 설계원칙에 맞지 않을 뿐더러 도메인로직을 여러 곳에 산재하게 만들 뿐더러 코드의 중복과 오브젝트의 재활용성을 극히 떨어뜨리게 한다.**
* 초기 스프링의 예제나 스프링 개발자들에 의해서 쓰여진 스프링 서적의 샘플 코드도 이런 구조를 그대로 사용했다는 것은 **이런 모델이 얼마나 자연스럽게 개발자들에게 받아들여지고 사용되어져 왔는지 짐작하게 해준다.**
