# 디폴트 메서드

## 디폴트 메서드가 등장한 이유&#x20;

자바 8에서 **디폴트 메서드(default method)** 가 등장하기 전에는 **인터페이스에 메서드를 새로 추가**하는 순간, 이미 배포된 기존 구현 클래스들이 해당 메서드를 구현하지 않았기 때문에, 전부 컴파일 에러를 일으키게 되는 문제가 있었다.&#x20;

이 때문에 특정 인터페이스를 이미 많은 클래스에서 구현하고 있는 상황에서, 인터페이스에 새 기능을 추가하려면 기존 코드를 일일이 모두 수정해야 했다.&#x20;

**디폴트 메서드**는 이러한 문제를 해결하기 위해 등장했다. 자바 8부터는 **인터페이스에서 메서드 본문을 가질 수 있도록 허용해 주어, 기존 코드를 깨뜨리지 않고 새 기능을 추가**할 수 있게 되었다.&#x20;

### 예제1&#x20;

#### 인터페이스와 구현 클래스&#x20;

먼저 기존 코드에서 알림 기능을 처리하는 `Notifier` 인터페이스와 세 가지 구현체(`EmailNotifier`, `SMSNotifier`, `AppPushNotifier`)가 있다고 하자. `Notifier` 는 단순히 메시지를 알리는 `notify()` 메서드 한 가지만 정의하고 있고, 각 구현체는 해당 기능을 구현한다.

```java
package defaultMethod.ex1;

public interface Notifier {
    void notify(String message);
}
```

```java
package defaultMethod.ex1;

public class EmailNotifiler implements Notifier{
    @Override
    public void notify(String message) {
        System.out.println("[EMAIL] " + message);
    }
}
```

```java
package defaultMethod.ex1;

public class SMSNotifier implements Notifier{
    @Override
    public void notify(String message) {
        System.out.println("[SMS] " + message);
    }
}
```

```java
package defaultMethod.ex1;

public class AppPushNotifier implements Notifier{
    @Override
    public void notify(String message) {
        System.out.println("[APP] " + message);
    }
}
```

```java
package defaultMethod.ex1;

import java.util.List;

public class NotifierMainV1 {

    public static void main(String[] args) {
        List<Notifier> notifiers = List.of(new EmailNotifiler(), new SMSNotifier(), new AppPushNotifier());
        notifiers.forEach(n -> n.notify("서비스 가입을 환영합니다."));
//        for (Notifier notifier : notifiers) {
//            notifier.notify("서비스 가입을 환영합니다");
//        }
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 12.06.10.png" alt=""><figcaption></figcaption></figure>

### 예제2&#x20;

#### 인터페이스에 새로운 메서드를 추가했을 때 발생하는 문제&#x20;

요구사항이 추가되었다. 알림을 **미래의 특정 시점**에 자동으로 발송하는 스케줄링 기능을 추가해야 한다고 해보자. 그래서 `Notifier` 인터페이스에 `scheduleNotification()` 메서드를 추가하자.

```java
package defaultMethod.ex2;

import java.time.LocalDateTime;

public interface Notifier {
    // 알림을 보내는 기본 기능
    void notify(String message);

    // 신규 기능
    void scheduleNotification(String message, LocalDateTime scheduleTime);
}
```

```java
package defaultMethod.ex2;

import java.time.LocalDateTime;

public class EmailNotifiler implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("[EMAIL] " + message);
    }

    @Override
    public void scheduleNotification(String message, LocalDateTime scheduleTime) {
        System.out.println("[EMAIL 전용 스케줄링] message: " + message + ", time: " + scheduleTime);
    }
}
```

```java
package defaultMethod.ex2;

public class SMSNotifier implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("[SMS] " + message);
    }
}
```

```java
package defaultMethod.ex2;

public class AppPushNotifier implements Notifier {
    @Override
    public void notify(String message) {
        System.out.println("[APP] " + message);
    }
}
```

```java
package defaultMethod.ex2;

import java.time.LocalDateTime;
import java.util.List;

public class NotifierMainV2 {

    public static void main(String[] args) {
        List<Notifier> notifiers = List.of(new EmailNotifiler(), new SMSNotifier(), new AppPushNotifier());
        notifiers.forEach(n -> n.notify("서비스 가입을 환영합니다."));

        // 스케줄 기능 추가
        LocalDateTime plusOneDays = LocalDateTime.now().plusDays(1);
        notifiers.forEach(n -> n.scheduleNotification("hello", plusOneDays));
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-22 12.09.12.png" alt=""><figcaption></figcaption></figure>

`scheduleNotification()` 메서드가 `Notifier` 인터페이스에 새로 추가됨에 따라, 기존에 존재하던`SMSNotifier`, `AppPushNotifier` 구현 클래스들이 **강제로** 이 메서드를 구현하도록 요구된다.

* 규모가 작은 예제에서는 그럼 나머지 두 클래스도 재정의하면 된다고 생각할 수 있다.
*   하지만 실무 환경에서 **해당 인터페이스를 구현한 클래스가 수십\~수백 개**라고 한다면, 이 전부를 수정해서 새 메서

    드를 재정의해야 한다.
*   심지어 우리가 만들지 않은, **외부 라이브러리**에서 `Notifier` 를 구현한 클래스가 있다면 그것까지 전부 깨질 수

    있어, 호환성을 깨뜨리는 매우 심각한 문제가 된다.

### 예제3 - Notifier 수정&#x20;

#### 디폴트 메서드로 문제 해결

자바 8부터 이러한 **하위 호환성** 문제를 해결하기 위해 **디폴트 메서드**가 추가되었다. 인터페이스에 메서드를 새로 추가 하면서, **기본 구현**을 제공할 수 있는 기능이다. 예를 들어, `Notifier` 인터페이스에 `scheduleNotification()` 메서드를 **default** 키워드로 작성하고 기본 구현을 넣어두면, 구현 클래스들은 이 메서드를 굳이 재정의하지 않아도 된다.

```java
package defaultMethod.ex2;

import java.time.LocalDateTime;

public interface Notifier {
    // 알림을 보내는 기본 기능
    void notify(String message);

    // 신규 기능
    default void scheduleNotification(String message, LocalDateTime scheduleTime) {
        System.out.println("[기본 스케줄링] message : " + message + ", time : " + scheduleTime);
    }
}
```

## 디폴트 메서드의 올바른 사용법&#x20;

디폴트 메서드는 강력한 기능이지만, 잘못 사용하면 오히려 코드가 복잡해지고 유지보수하기 어려워질 수 있다. 다음은 디폴트 메서드를 사용할 때 고려해야 할 주요 사항이다.

#### 1. 하위 호환성을 위해 최소한으로 사용&#x20;

* 디폴트 메서드는 주로 **이미 배포된 인터페이스**에 새로운 메서드를 추가하면서 기존 구현체 코드를 깨뜨리지 않기 위한 목적으로 만들어졌다.
* 새 메서드가 필요한 상황이고, 기존 구현 클래스가 많은 상황이 아니라면, 원칙적으로는 각각 구현하거나, 또는 추상 \
  메서드를 추가하는 것을 고려하자.
* 불필요한 디폴트 메서드 남용은 코드 복잡도를 높일 수 있다.

#### 2. 인터페이스는 여전히 추상화의 역할&#x20;

* 디폴트 메서드를 통해 인터페이스에 로직을 넣을 수 있다 하더라도, 가능한 한 로직은 구현 클래스나 별도 클래스에 두고, 인터페이스는 **계약(Contract)의 역할**에 충실한 것이 좋다.
* 디폴트 메서드는 어디까지나 **하위 호환을 위한 기능**이나, **공통으로 쓰기 쉬운 간단한 로직**을 제공하는 정도가 이상적이다.

#### 3. 다중 상속(충돌) 문제&#x20;

*   하나의 클래스가 여러 인터페이스를 동시에 구현하는 상황에서, **서로 다른 인터페이스에 동일한 시그니처의 디폴**

    **트 메서드**가 존재하면 충돌이 일어난다.
*   이 경우 **구현 클래스**에서 반드시 메서드를 재정의해야 한다. 그리고 직접 구현 로직을 작성하거나 또는 어떤 인터

    페이스의 디폴트 메서드를 쓸 것인지 명시해 주어야 한다.

```java
interface A {
    default void hello() {
        System.out.println("Hello from A");
    }
}
    
interface B {
    default void hello() {
        System.out.println("Hello from B");
    }
}
    
public class MyClass implements A, B {
    @Override
    public void hello() {
        // 반드시 충돌을 해결해야 함
        // 1. 직접 구현
        // 2. A.super.hello();
        // 3. B.super.hello();
    }
}
```

#### 4. 디폴트 메서드에 상태(state) 를 두지 않기&#x20;

* **인터페이스는 일반적으로 상태 없이 동작만 정의하는 추상화 계층이다.**\
  **(본래 목적을 잃어서는 안된다!)**
* 인터페이스에 정의하는 디폴트 메서드도 "구현"을 일부 제공할 뿐, 인스턴스 변수를 활용하거나, 여러 차례 호출시 상태에 따라 동작이 달라지는 등의 동작은 지양해야 한다.
* 이런 로직이 필요하다면 클래스(추상 클래스 등)로 옮기는 것이 더 적절하다.
