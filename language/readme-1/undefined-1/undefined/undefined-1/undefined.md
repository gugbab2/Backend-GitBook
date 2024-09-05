# 비트 연산자 활용 예제

#### 비트 연산자 사용 이유

* **효율적인 권한 관리**&#x20;
  * 이러한 권한을 하나의 정수 값으로 표현할 수 있다. 각 비트는 특정 권한을 나타내며, 비트 연산을 통해 권한을 쉽게 조작하고 확인할 수 있다.&#x20;
  * 예를 들어, `int` 타입의 변수 하나로 32 개의 권한을 표현할 수 있다.&#x20;
* **성능 최적화**&#x20;
  * 비트 연산은 CPU 에서 매우 빠르게 처리된다. 논리 연산은(`AND`, `OR`, `XOR`) 과 비트 이동은 기본 연산으로 수행 속도가 빠르다.&#x20;
  * 권한을 비트 플래그로 표현하면 성능이 중요한 어플리케이션에서 유리할 수 있다.&#x20;
* **메모리 절약**&#x20;
  * 각 권한을 비트 플래그로 표현하면, 권한 정보를 작은 메모리 공간에 저장할 수 있다.&#x20;
  * 예를 들어, 32 개의 권한을 관리할 때, 4바이트(`int`) 만 있으면 된다.&#x20;

## 예제1  (권한 관리 시스템)&#x20;

```java
public class Permissions {
    public static final int READ = 1;       // 0001
    public static final int WRITE = 2;      // 0010
    public static final int EXECUTE = 4;    // 0100
    public static final int DELETE = 8;     // 1000
}
```

```java
public class PermissionService {

    //  특정 권한이 있는지 확인하는 메서드 
    public boolean hasPermission(int userPermissions, int permission){
        return (userPermission & permission) == permission;
    }
    
    public static void main(String[] args) {
        PermissionService permissionService = new PermissionService();

        // 사용자의 현재 권한 설정 (읽기 + 쓰기 + 삭제)
        int userPermissions = Permissions.READ | Permissions.WRITE | Permissions.DELETE;    // 1011

        // 읽기 권한 확인
        System.out.println("Has READ permission: " + permissionService.hasPermission(userPermissions, Permissions.READ)); // true

        // 쓰기 권한 확인
        System.out.println("Has WRITE permission: " + permissionService.hasPermission(userPermissions, Permissions.WRITE)); // true

        // 실행 권한 확인
        System.out.println("Has EXECUTE permission: " + permissionService.hasPermission(userPermissions, Permissions.EXECUTE)); // false

        // 삭제 권한 확인
        System.out.println("Has DELETE permission: " + permissionService.hasPermission(userPermissions, Permissions.DELETE)); // true
    }
            
}
```

## 예제2 (안드로이드 화면 조정 플래그)&#x20;

```java
import android.app.Activity;
import android.os.Bundle;
import android.view.Window;
import android.view.WindowManager;

public class MainActivity extends Activity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);

        // 플래그 설정: FLAG_FULLSCREEN 및 FLAG_KEEP_SCREEN_ON 옵션 설정
        int flags = WindowManager.LayoutParams.FLAG_FULLSCREEN | WindowManager.LayoutParams.FLAG_KEEP_SCREEN_ON;

        // Window에 플래그 적용
        Window window = getWindow();
        window.setFlags(flags, flags);

        setContentView(R.layout.activity_main);
    }
}

```
