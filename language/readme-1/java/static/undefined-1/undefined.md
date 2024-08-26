# 스택 영역, 힙 영역 - 기본

## 스택과 큐 자료 구조

* Stack : LIFO, Last In First Out\
  \-> 잘못된 재고 관리&#x20;
* Queue : FIFO, First In First Out\
  \-> 올바른 재고 관리

## 스택 영역&#x20;

```java
public class JavaMemoryMain1 {
    public static void main(String[] args) {
        System.out.println("main start");
        method1(10);
        System.out.println("main end");
    }

    static void method1(int m1) {
        System.out.println("method1 start");
        int cal = m1 * 2;
        method2(cal);
        System.out.println("method1 end");
    }

    static void method2(int m2) {
        System.out.println("method2 start");
        System.out.println("method2 end");
    }
}

```

#### 위의 코드가 실행되는 과정

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.00.32.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.00.47.png" alt=""><figcaption></figcaption></figure>

## 스택 영역과 힙 영역

```java
public class JavaMemoryMain2 {
    public static void main(String[] args) {

        System.out.println("main start");
        method1();
        System.out.println("main end");

    }

    static void method1() {
        System.out.println("method1 start");
        Data data1 = new Data(10);
        method2(data1);
        System.out.println("method1 end");
    }

    static void method2(Data data2) {
        System.out.println("method2 start");
        System.out.println("data.value = " + data2.getValue());
        System.out.println("method2 end");

    }
}
```

#### 위의 코드가 실행되는 과정

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.07.01.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.07.46.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.08.09.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../.gitbook/assets/스크린샷 2024-01-29 21.11.36.png" alt=""><figcaption></figcaption></figure>

