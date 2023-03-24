# Object.equals() && .hashCode()

### equals() && hashCode() 오버라이딩 할 때?

* 기본 자료형은 '==' 연산자를 사용해 같은지 다른지를 비교할 수 있지만 참조자료형은 안된다..
* 때문에 equals() && hashCode() 를 오버라이딩 해서 같은지 비교해주도록 해야한다.

### equals() && hashCode() 를 함께 오버라이딩 해야하는 이유?

* equals() 메소드를 오버라이딩 해서 객체가 서로 같다고 할 수 있겠지만, 그 값이 같다고 해서 그 객체의 주소 값이 같지는 않기 때문이다. \
  \-> 따라서, 같은 hashCode() 메소드 결과를 갖도록 하기 위해서 오버라이딩 해서 사용해야 한다.

### 인텔리제이에서 기본적으로 생성해주는 equals(), hashCode() 형식.

* equals() : 객체의 각각의 인스턴스 변수를 비교해주는 메서드
* hashCode() : 객체의 고유값(주소)을 나타내는 메서드\
  \-> 객체의 메모리 주소를 16진수로 리턴한다. &#x20;

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
