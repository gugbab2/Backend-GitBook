# AssertJ Exception Assertions

## 1. 개요&#x20;

* 이번에는 AssertJ 라이브러리의 예외 전용 Assert 에 대해서 알아보자.&#x20;

## 2. AssertJ 를 사용하지 않은 예제&#x20;

* 예외가 발생했는지 테스트하려면 예외를 catch 해야 한다.&#x20;
* 아래 코드에서 예외가 발생하지 않는다면 테스트는 성공하게 되는데, 테스트에서 예외가 발생해야만 한다면 예외가 발생하지 않는 테스트는 실패한 테스트이다.&#x20;

```java
try {
    // ...
} catch (Exception e) {
    // assertions
}
```

## 3. AssertJ 를 사용하는 예제&#x20;

### **assertThatThrownBy()** <a href="#bd-1-using-assertthatthrownby" id="bd-1-using-assertthatthrownby"></a>

* 범위를 벗어난 항목을 인덱싱하면 IndexOutOfBoundsException 이 발생하는지 확인하는 코드이다.

```java
assertThatThrownBy(() -> {
    List<String> list = Arrays.asList("String one", "String two");
    list.get(2);    // 범위를 벗어난 항목 인덱싱
}).isInstanceOf(IndexOutOfBoundsException.class)
  .hasMessageContaining("Index: 2, Size: 2");
```

* 다양한 표준 AssertJ 메서드가 존재한다.&#x20;

```java
.hasMessage("Index: %s, Size: %s", 2, 2)
.hasMessageStartingWith("Index: 2")
.hasMessageContaining("2")
.hasMessageEndingWith("Size: 2")
.hasMessageMatching("Index: \\d+, Size: \\d+")
.hasCauseInstanceOf(IOException.class)
.hasStackTraceContaining("java.io.IOException");
```

### **assertThatExceptionOfType()** <a href="#bd-2-using-assertthatexceptionoftype" id="bd-2-using-assertthatexceptionoftype"></a>

* 위 예제와 비슷하지만, 처음부터 예외 유형을 지정할 수 있다.&#x20;

```java
assertThatExceptionOfType(IndexOutOfBoundsException.class)
  .isThrownBy(() -> {
      // ...
}).hasMessageMatching("Index: \\d+, Size: \\d+");
```

### **assertThatIOException 및 기타 일반 유형**  <a href="#bd-3-using-assertthatioexception-and-other-common-types" id="bd-3-using-assertthatioexception-and-other-common-types"></a>

* _assertThatIllegalArgumentException()_
* _assertThatIllegalStateException()_
* _assertThatIOException()_
* _assertThatNullPointerException()_

```java
assertThatIOException().isThrownBy(() -> {
    // ...
});
```

_..._&#x20;
