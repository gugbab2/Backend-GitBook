# StringBuilder, StringBuffer

## 불변객체인 String 의 단점을 해결한 문자열 클래스

* `String` 클래스는 불변객체이기 때문에 Thread Safe 하지만,  문자열 추가 시 기존 객체를 사용하는 것이 아닌 새로운 객체를 생성하고 추가 된 문자열로 속성을 초기화한다.&#x20;
  * 이는 변경에 자주 일어나는 문자열 객체에 대해서 객체의 생성, 삭제가 반복적으로 일어나기 때문에, 메모리 사용이 비효율 적이라는 단점이 있었다.&#x20;
* 이러한 단점을 해결한 클래스가 `StringBuiler`, `StringBuffer` 이다.

## StringBuilder, StringBuffer 의 내부 구조 및 동작 방식

* **내부 구조(char\[])**
  * `StringBuilder`, `StringBuffer` 는 내부적으로는 **`char[]` 배열을 사용해 문자열 데이터를 저장한다.**&#x20;
  * **이 배열은 초기 생성 시에 특정 크기(16)로 할당되며, 문자열이 추가되거나 변경될 때 이 배열의 내용을 수정한다.**&#x20;
* **동작 방식**&#x20;
  * **버퍼 관리**&#x20;
    * 동작 원리 : 초기 생성 시에 `char[]` 배열이 할당되며, 문자열이 추가되거나 수정될 때 이 배열에 값을 직접 추가한다.&#x20;
    * 자동 확장 : 만약 문자열이 배열의 현재 크기를 초과하면, 기존 배열의 크기를 두 배로 확장 한 배열을 생성한 후, 기존 배열의 내용을 복사하고, 그 후 추가적인 문자를 저장한다.&#x20;
    * 새로운 객체 생성 없음 : 이러한 과정에서 배열의 크기가 충분할 경우 새로운 객체를 생성하지 않고, 기존 배열의 값만 변경하기 때문에, 성능이 효율적이다.&#x20;
  * **동기화(`StringBuffer` 만 해당)**
    * `StringBuffer`는 스레드 안전성을 보장하기 위해, 각 메서드에 `synchronized` 키워드를 사용하여 내부 `char[]` 배열에 대한 접근을 제어합니다.
    * **`synchronized` 키워드**: 이 키워드는 특정 메서드나 코드 블록이 한 번에 하나의 스레드만 접근할 수 있도록 제한합니다.

## StringBuilder 클래스 구현 예시&#x20;

```java
public class StringBuilder {
    private char[] value;
    private int count;

    public StringBuilder() {
        value = new char[16]; // 기본 크기 16으로 초기화
    }

    public StringBuilder append(String str) {
        if (str == null) str = "null";
        int len = str.length();
        ensureCapacityInternal(count + len);
        str.getChars(0, len, value, count);
        count += len;
        return this;
    }

    private void ensureCapacityInternal(int minimumCapacity) {
        if (minimumCapacity - value.length > 0)
            expandCapacity(minimumCapacity);
    }

    private void expandCapacity(int minimumCapacity) {
        int newCapacity = value.length * 2 + 2;
        if (newCapacity - minimumCapacity < 0)
            newCapacity = minimumCapacity;
        value = Arrays.copyOf(value, newCapacity);
    }
}

```
