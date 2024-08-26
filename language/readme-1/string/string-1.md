# String 클래스에서 자주 사용되는 메서드

### 문자열의 길이 확인

* **int .length()** : 문자열의 길이 확인

### 문자열이 비어있는지 확인

* **boolean .isEmpty()** : 문자열이 비어있는지 확인

### 문자열이 같은지 비교

* **boolean .equals(Object obj)** : 매개변수와 문자열이 같은지 확인
  * 자바에서 객체는 == 가 아닌 equals 메서드를 통해서 비교할 수 있다.
  * 하지만 String 에서는 == 으로도 비교가 가능하다.\
    \-> 이유는, Constant Pool 이라는 것이 자바에는 존재하기 때문이다.**(String literal 만 해당된다!)**\
    \-> **자바에서는 객체들을 재사용하기 위해서 Constant Pool 이 존재하고, String 의 경우 동일한 값을 갖는 객체가 있으면, 이미 만든 객체를 재사용한다.**\
    **-> text, text2 는 같은 값을 같는 객체이다.**

```java
public void checkCompare(){

    String text = "Chack value";
    String text2 = "Chack value";
    // 아래와 같이 String 객체를 생성할 경우 값이 같더라도 다른 객체로 인식한다. 
    // String text2 = new String("Chack value");

    if(text == text2){
        System.out.println("text == text2 result is same");        // O
    }else{
        System.out.println("text == text2 result is different");   // X
    }

    if(text.equals("Chack value")){
        System.out.println("text.equals(text2) result is same");   // O
    }
}
```

* **int .compareTo(String str)** : 보통 정렬을 할 때 사용하는 메서드로, 매개변수로 넘겨준 String 객체가 알파벳 순으로 앞에 있으면 양수를, 뒤에 있으면 음수를 리턴한다.

```java
public void checkCompareTo(){
    
    String text = "a";
    String text2 = "b";
    String text3 = "c";
    
    System.out.println(text2.compareTo(text));      // 1
    System.out.println(text2.compareTo(text3));     // -1
    System.out.println(text.compareTo(text3));      // -2

}
```

### **특정 조건에 맞는 문자열이 있는지 확인**

* **boolean .startWith(String prefix)** : 해당 문자열이 매개변수로 시작하는지 확인.
* **boolean .endWith(String prefix)** : 해당 문자열이 매개변수로 끝나는지 확인.
* **boolean .contains(CharSequence s)** : 매개변수로 넘어온 값이 해당 문자열에 존재하는지 확인.

### 문자열 내에서 위치를 찾아내는 방법

* **int .indexOf(int ch)** : 가장 왼쪽부터 문자열이나, char 을 찾는다.\
  \-> _char 은 정수형으로 매개변수로 char 을 넘겨주면 자동적으로 형변환이 일어난다._
  * **int .indexOf(String str)**
  * **int .indexOf(int str, int fromIndex)**
  * **int .indexOf(String str, int fromIndex)**
* **int .lastIndexOf(int ch)** : 가장 오른쪽부터 문자열이나, char 을 찾는다.
  * **int .lastIndexOf(Strint str)**
  * **int .lastIndexOf(int str, int fromIndex)**
  * **int .lastIndexOf(String str, int fromIndex)**

### 문자열의 값 일부를 추출하는 방법

* **char .charAt(int index)** : 특정 위치의 char 값을 리턴한다.

### char 배열 값을 문자열로 변경하는 방법

* **static String .copyValueOf(char\[] data)** : char 배열 값을 문자열로 변경.

### 문자열을 char 배열로 변경하는 방법

* **char\[] .toCharArray()** : 문자열을 char 배열로 변경.

### 문자열의 일부 값을 잘라내는 방법

* **String .subString(int beginIndex)** : beginIndex 부터 끝까지 대상 문자열을 잘라낸다.
* **String .subString(int beginIndex, int endIndex)** : beginIndex 부터 endIndex까지 대상 문자열을 잘라낸다.

### 문자열을 여러개의 String 배열로 나누는 방법

* **String\[] .split(String regex)** : regex 에 있는 정규 표현식에 맞추어 문자열을 잘라 String 배열로 리턴한다.
* **String\[] .split(String regex, Int limit)** : regex 에 있는 정규 표현식에 맞추어 문자열을 잘라 String 배열로 리턴한다. 이때 배열의 크기가 limit 보다 커서는 안된다.

### 문자열의 내용을 교체하는 방법

* **String .trim()** : 시작과 끝의 공백을 제거한다.
* **String replace(CharSequance target, CharSequance replacement)** : target 을 replacement 로 교체
* **String .replaceAll(String regex, String replacement)** : 해당 문자열의 내용 중 regex 에 표현된 정규 표현식에 표현되는 모든 내용을 replacement 로 교체한다.

### 기본자료형을 문자열로 변환하는 방법

* **static String .valueOf(기본자료형 ..)** : 기본자료형을 문자열로 변경한다.
