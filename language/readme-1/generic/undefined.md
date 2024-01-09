# 제네릭 기본

## 제레릭이란?

* 참조형 객체의 타입 형 변환에서 발생할 수 있는 문제점을 사전에 방지하기 위한 Java5 부터 제공된 문법이다. \
  \-> _**실수를 미리 방지할 수 있는 좋은 방법**_
* 사용방법은 아래와 같다.
  * \<T> 를 통한 제네릭 지정을 통해서 명시적으로 타입을 지정할 수 있다.
  * 명시적인 타입 지정을 통해서, 인스턴스 변수 사용시 형변환을 해주지 않아도 되는 장점이 있다.&#x20;
  * CharSequence charSequence = dto3.getObject(); 코드를 통해서 다형성으로 캐스팅이 가능하다.

```java
public class GenericCastingDTO<T>  implements Serializable {
    private T object;

    public void setObject(T object) {
        this.object = object;
    }

    public T getObject(){
        return object;
    }
}

public class GenericSample {
    GenericCastingDTO<String> dto1;
    GenericCastingDTO<StringBuilder> dto2;
    GenericCastingDTO<StringBuffer> dto3;

    public void checkGenericDTO(){
        dto1 = new GenericCastingDTO<String>();
        dto1.setObject(new String());

        dto2 = new GenericCastingDTO();
        dto2.setObject(new StringBuilder());

        dto3 = new GenericCastingDTO();
        dto3.setObject(new StringBuffer());

        String string = dto1.getObject();
        StringBuilder stringBuilder = dto2.getObject();
        CharSequence charSequence = dto3.getObject();
    }

}
```

## 제네렉 타입의 이름 정하기

* 제네릭 타입의 이름은 어떠한 단어가 들어가도 상관이 없지만, 자바에서 정의한 기본 규칙은 있다.
  * E : 요소(element)
  * K : key
  * N : 숫자
  * T : 타입
  * V : 값
  * S,U,V : 두번째, 세번째, 네번째에 선언된 타입
