# 절차 지향 프로그래밍

## 절차 지향 프로그래밍&#x20;

#### 절차 지향 프로그래밍

* 말 그대로 절차를 지향하며, 쉽게 실행 순서를 중요하게 생각하는 방식이다.
* 프로그램의 흐름을 순차적으로 따르며 처리하는 방식이다.
* 즉 "어떻게" 를 중심으로 프로그래밍 한다.

#### 객체 지향 프로그래밍&#x20;

* 말 그대로 객체를 지향하며, 쉽게 객체를 중요하게 생각하는 방식이다.
* 실제 세계의 사물이나 사진을 객체로 보고, 이러한 객체들 간의 상호작용을 중심으로 프로그래밍하는 방식이다.&#x20;
* 즉 "무엇을" 중심으로 프로그래밍 한다.&#x20;

#### 둘의 중요한 차이

* 절차 지향은 데이터와 해당 데이터에 대한 처리 방식이 분리되어 있다.
* 객체 지향은 데이터와 해당 데이터의 대한 행동(메서드) 가 하나의 '객체' 안에 포함되어 있다.

```java
public class MusicPlayerData {
    int volume = 0;
    boolean isOn = false;
}


public class MusicPlayerMain2 {
    public static void main(String[] args) {
        MusicPlayerData data = new MusicPlayerData();

        // 음악 플레이어 켜기
        on(data);
        // 볼륨 증가
        volumeUp(data);
        // 볼륨 증가
        volumeUp(data);
        // 볼륨 감소
        volumeDown(data);
        // 음악 플레이어 상태
        showStatus(data);
        // 음악 플레이어 끄기
        off(data);

    }

    static void on(MusicPlayerData data) {
        data.isOn = true;
        System.out.println("음악 플레이어를 시작합니다");
    }

    static void off(MusicPlayerData data) {
        data.isOn = false;
        System.out.println("음악 플레이어를 종료합니다.");
    }

    static void volumeUp(MusicPlayerData data) {
        data.volume++;
        System.out.println("음악 플레이어 볼륨 : " + data.volume);
    }

    static void volumeDown(MusicPlayerData data) {
        data.volume--;
        System.out.println("음악 플레이어 볼륨 : " + data.volume);
    }

    static void showStatus(MusicPlayerData data) {
        System.out.println("음악 플레이어 상태 확인");
        if (data.isOn) {
            System.out.println("음악 플레이어 ON, 볼륨 : " + data.volume);
        } else {
            System.out.println("음악 플레이어 OFF");
        }
    }
}

```

