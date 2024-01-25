# 기본형 vs 참조형

## 기본형 vs 참조형1 - 기본

#### 기본형 vs 참조형 &#x20;

* 기본형 타입(Primitive Type) : 사용하는 값을 변수에 직접 넣을 수 있는 기본형이라 부른다.\
  \-> 바로 값을 사용할 수 있다. \
  \-> 바로 연산에 사용할 수 있다.&#x20;
* 참조형 타입(Reference Type) : 데이터가 접근하기 위한 참조(주소) 를 저장하는 타입을 참조형이라 부른다. \
  \-> 주소를 통해 객체 찾아서 값에 접근할 수 있다. \
  \-> 바로 연산을 할 수 없다.&#x20;

#### 쉽게 이해하는 팁&#x20;

* 기본형을 제외한 나머지는 모두 참조형&#x20;
* 참조형인 클래스만 개발자가 직접 정의할 수 있다.&#x20;

## 기본형 vs 참조형2 - 변수 대입

* **대원칙 : 자바는 항상 변수의 값을 복사해서 대입한다.** \
  **-> 기본형, 참조형 모두 항상 변수의 있는 값을 복사해서 대입한다.** \
  **-> 참조형의 경우 객체의 위치를 가리키는 참조값만 복사된다.**

```java
// 기본형 
int a = 10;
int b = a;    // a라는 변수의 값을 복사해서 대입한다. 

// 참조형
Student s1 = new Student();    // s1 = x001
Student s2 = s1;               // s2 = x001
```



## 기본형 vs 참조형3 - 메서드 호출

* **대원칙 : 자바는 항상 변수의 값을 복사해서 대입한다.** \
  **-> 기본형, 참조형 모두 항상 변수의 있는 값을 복사해서 대입한다.** \
  **-> 참조형의 경우 객체의 위치를 가리키는 참조값만 복사된다.**

```java
// 기본형 
public class MethodChange1 {
    public static void main(String[] args) {
        int a = 10;
        System.out.println("메서드 호출 전 : a = " + a);
        changePrimivate(a);    // a 값을 복사해서 대입하는 것이기 때문에, a 의 값에 영향이 없다. 
        System.out.println("메서드 호출 후 : a = " + a);
    }
    
    static void changePrimivate(int x){
        x = 20;
    }
}

// 참조형
package ref;

public class MethodChange2 {
    public static void main(String[] args) {
        Data dataA = new Data();
        dataA.value = 10;

        System.out.println("메서드 호출 전 : dataA.value = " + dataA.value);
        changeReference(dataA);    // dataA 참조값을 넘겨주기 때문에, dataA.value 값을 변경하면 영향을 받는다. 
        System.out.println("메서드 호출 후 : dataA.value = " + dataA.value);
    }

    static void changeReference(Data dataX){
        dataX.value = 20;
    }
}

```

