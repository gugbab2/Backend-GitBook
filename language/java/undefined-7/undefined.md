---
description: 스
---

# 함수형 인터페이스

## 1. 함수형 인터페이스란?

* **하나의 추상 메서드를 지정하는 인터페이스이다.**  \
  **->** 람다 표현식으로 함수형 인터페이스의 추상 메서드를 구현을 직접 전달할 수 있으므로, \
  **전체 표현식을 함수형 인터페이스의 인스턴스로 취급할 수 있다.**&#x20;

```java
//해당 인터페이스만 함수형 인터페이스이다!
public interface Adder{
    int add(int a, int b);
}

public interface SmartAdder extends Adder{
    int add(double a, double b);
}

public interface Nothing{
}
```

## 2. 람다를 활용한 함수형 인터페이스

```java
Runnable r1 = () -> System.out.println("Hello World 1");
Runnable r2 = () -> new Runnable() {
    public void run(){
        System.out.println("Hello World 2");
    }
}

public static void process(Runnable r){
    r.run();
}

process(r1);
process(r2);
process(() -> System.out.println("Hello World 3"));

```

## 3. 함수 디스크립터
