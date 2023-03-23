# 접근제어자

### 메소드, 변수 접근 제어자

public : 누구나 접근할 수 있도록 할 때 사용한다.

protected : 같은 패키지 내에 있거나 상속받은 경우에만 접근할 수 있다.

package-private(default) : 아무런 접근 제어자를 적어주지 않을 때.

private : 해당 클래스 내에서 접근 가능.&#x20;

### 클래스 접근제어자

public 으로 선언된 클래스가 소스 내에 있다면, 그 소스 파일의 이름은 public 인 클래스 이름과 동일해야 한다.

```
// 한 클래스 파일 내에 두개의 public class 는 존재할 수 없다.
package c.javapackage;
public class PublicClass{
   public static void main(String args[]){
   }
}

public class PublicSecondClass{
}
```
