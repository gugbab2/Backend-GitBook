# Object.equals() && .hashCode()

### equals() && hashCode() 오버라이딩 할 때?

* 기본 자료형은 '==' 연산자를 사용해 같은지 다른지를 비교할 수 있지만 참조자료형은 안된다..
* 때문에 equals() && hashCode() 를 오버라이딩 해서 같은지 비교해주도록 해야한다.

### equals() && hashCode() 를 함께 오버라이딩 해야하는 이유?

* equals() 메소드를 오버라이딩 해서 객체가 서로 같다고 할 수 있겠지만, 그 값이 같다고 해서 그 객체의 주소 값이 같지는 않기 때문이다.\
  \-> 따라서, 같은 hashCode() 메소드 결과를 갖도록 하기 위해서 오버라이딩 해서 사용해야 한다.\
  \-> equals() 메서드를 통해서 인스턴스 변수의 값이 같은지 판단할 수 있지만, hash 를 사용하는 HashSet, HashMap 등의 클래스에서는 같은 값이라고 판단하지 않기에, 항상 오버라이딩해서 사용해야 한다.
* 기본적으로 Object 클래스의 hashCode() 메서드는 개체의 주소를 기반으로 해시코드를 생성하지만, 이 메모리는 직접적인 값이 아닌, JVM 이 개체의 메모리 위치를 기반으로 계산한 고유의 정수값이다. \
  \-> Java 는 개체의 주소값을 직접적으로 알 수 있는 방법은 없다..

### equals() && hashCode() 오버라이딩 시 주의 점

* equals() 메서드의 대칭성 : A.equals(B) 가 true 라면, B.equals(A) 도 true 이어야 한다.
* null 비교 허용 : equals() 메서드에서는 null 과의 비교를 허용해야 하며 null 과 비교시 항상 false 를 반환해야 한다.\
  \-> NullPointException 을 방지하고, null 과 비교시 언제나 false 를 반환하기 때문에, 일관성을 유지할 수 있다.
* hashCode() 일치 : equals() 메서드를 통해서 두 객체가 같다고 판단되면 두 객체의 hashCode() 는 언제나 동일해야 한다.

### 인텔리제이에서 기본적으로 생성해주는 equals(), hashCode() 형식.

* equals() : 객체의 각각의 인스턴스 변수를 비교해주는 메서드
* hashCode() : 객체의 고유값(주소)을 나타내는 메서드\
  \-> 객체의 메모리 주소를 16진수로 리턴한다.

```java
public class MemberDTO {
    String name = "";
    String email = "";
    String phone = "";

    public MemberDTO(String name) {
        this.name = name;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        MemberDTO memberDTO = (MemberDTO) o;

        if (!Objects.equals(name, memberDTO.name)) return false;
        if (!Objects.equals(email, memberDTO.email)) return false;
        return Objects.equals(phone, memberDTO.phone);
    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + (email != null ? email.hashCode() : 0);
        result = 31 * result + (phone != null ? phone.hashCode() : 0);
        return result;
    }
}
```
