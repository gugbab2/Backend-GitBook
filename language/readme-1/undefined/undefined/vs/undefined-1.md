# 객체 지향 프로그래밍

## 객체 지향 프로그래밍

### 다시 한번 객체 지향의 개념에 대해서 생각해보자

* 말 그대로 객체를 지향하며, 쉽게 객체를 중요하게 생각하는 방식이다.
* 실제 세계의 사물이나 사진을 객체로 보고, 이러한 객체들 간의 상호작용을 중심으로 프로그래밍하는 방식이다.&#x20;
* 즉 "무엇을" 중심으로 프로그래밍 한다.&#x20;

#### 프로그램을 작성하는 절차도 중요하지만, 객체 지향에서는 온전한 객체를 정의하고, 구현하는 것이 가장 중요하다.&#x20;

### 다음의 음악 플레이어 객체를 생각해보자

* 프로그램의 실행 순서보다 음악 플레이어 클래스를 만드는 것 자체에 집중해야 한다.&#x20;
* 음악 플레이어가 어떤 속성(데이터) 을 가지고 있고, 어떤 기능(메서드) 를 제공하는지 초점을 맞추어야 한다.
* 현실 세계의 객체의 속성과 기능을 생각해서 만들어야만 한다!

#### 음악 플레이어

* 속성 : valume, isOn
* 기능 : on(), off(), volumeUp(), volumeDown(), showStatus()

### 코드 설명(객체 지향의 특징)

* MusicPlayer 를 사용하는 입장에서는, MusicPlayer 의 데이터인 Valume, isOn 같은 속성(데이터)을 전혀 사용하지 않는다.
* MusicPlayer 를 사용하는 입장에서는, 이제 MusicPlayer 내부에 어떤 속성(데이터)이 있는지 전혀 몰라도 된다.&#x20;
* MusicPlayer 를 사용하는 입장에서는, 단순하게 MusicPlayer 가 제공하는 기능 중에 필요한 기능을 호출해서 사용하기만 하면 된다.

#### 캡슐화

* MusicPlayer 를 보면 음악 플레이어를 구성하기 위한 속성과 기능이 마치 하나의 캡슐에 쌓여 있는 것 같다. \
  이렇게 속성과 기능을 하나로 묶어서 필요한 기능을 메서드를 통해 외부에 제공하는 것을 캡슐화라고 한다.
* 속성과 기능이  한 곳에 있기 때문에, 변경도 더 쉬워진다.
* 예를 들어서 MusicPlayer 내부 코드가 변하는 경우, MusicPlayer 를 사용하는 입장에서 코드를 변경하지 않아도 된다. \
  \-> **단! MusicPlayer 를 사용하는 입장에서, MusicPlayer 의 속성을 변경하지 않아야 한다.**\
  **(setter 를 소극적으로 사용해야 하는 이유 : 캡슐화를 깨뜨린다.)**

```java
public class MusicPlayer {
    
    int volume = 0;
    boolean isOn = false;

    void on() {
        isOn = true;
        System.out.println("음악 플레이어를 시작합니다");
    }

    void off() {
        isOn = false;
        System.out.println("음악 플레이어를 종료합니다.");
    }

    void volumeUp() {
        volume++;
        System.out.println("음악 플레이어 볼륨 : " + volume);
    }

    void volumeDown() {
        volume--;
        System.out.println("음악 플레이어 볼륨 : " + volume);
    }

    void showStatus() {
        System.out.println("음악 플레이어 상태 확인");
        if (isOn) {
            System.out.println("음악 플레이어 ON, 볼륨 : " + volume);
        } else {
            System.out.println("음악 플레이어 OFF");
        }
    }
}

public class MusicPlayerMain4 {
    public static void main(String[] args) {
        MusicPlayer player = new MusicPlayer();

        // 음악 플레이어 켜기
        player.on();
        // 볼륨 증가
        player.volumeUp();
        // 볼륨 증가
        player.volumeUp();
        // 볼륨 감소
        player.volumeDown();
        // 음악 플레이어 상태
        player.showStatus();
        // 음악 플레이어 끄기
        player.off();
    }
}
```
