# 배열

## 배열의 기초&#x20;

* 배열은 다음과 같이 선언할 수 있다.&#x20;
  * 선언 시 대괄호 안 내용은 비워두어야 한다.&#x20;

```java
int [] lottoNumber;
int lottoNumber []; 
```

* 배열은 다음과 같이 초기화 할 수 있다.&#x20;
  * 선언한 배열의 크기를 알 수 없기 때문에, 초기화하는 부분에서 배열의 크기를 지정해준다. \
    \-> **배열은 무조건 초기화 시 크기가 지정되어야만 한다. 중간에 배열의 크기를 변경할 수 없다 .. 때문에, 이러한 단점을 보완하기 위해서 Collection 이라는 것이 존재한다.**&#x20;
* 배열도 참조 자료형이기 때문에, 신규로 생성 시 `new` 키워드를 붙여주어야 한다.&#x20;

```java
int [] lottoNumber = new int[7];
int [] lottoNumber = {1,2,3,4,5,6,7};    // 배열을 초기화하는 다른 방법
```

* 배열은 인덱스를 통해 각 요소에 접근할 수 있으며, 인덱스는 0부터 시작된다.&#x20;
  * **각 요소는 배열 타입의 기본값으로 초기화 된다.** \
    **-> 참조 자료형의 경우 null 값으로 초기화 된다.** \
    \-> 아래 코드의 경우는 `int` 타입의 기본값으로 초기화 되었다.&#x20;
  * 이전에 지역 변수는 초기화를 사용하지 않는다고 이야기 했었는데, **배열은 예외적으로 지역 변수라 할지라도 크기만 지정되어 있다면, 기본값으로 초기화되며, 문제가 발생하지 않는다.**&#x20;
* **범위를 넘어서는 인덱스에 접근 시 `java.lang.ArrayIndexOutOfBoundException` 을 볼 수 있다..**&#x20;

```java
int [] lottoNumber = new int[7];
for(int i=0; i<lottoNumber.length; i++){
    System.out.println(lottoNumber[i]); // 0
}
```

## 2차원 배열&#x20;

* 2차원 배열은 다음과 같이 선언과 초기화를 할 수 있다.&#x20;
* 아래 2차원 배열은 다음과 같이 해석 할 수 있다.&#x20;
  * `twoDim[0]` = int 배열&#x20;
  * `twoDim[0][0]` = int 값&#x20;

```java
int [][]twoDim = new int[2][3];
int [][]twoDim = {{1,2,3}, {4,5,6}};    // 2차원 배열을 초기화하는 다른 방법 
```

* 2차원 배열을 for 루프를 사용해서 출력해보는 코드를 살펴보자&#x20;

```java
int [][]twoDim = {{1,2,3}, {4,5,6}};
System.out.println("twoDim.length=" + twoDim.length);
System.out.println("twoDim[0].length=" + twoDim[0].length);

for(int i=0; i<twoDim.length; i++){
    for(int j=0; j<twoDim[i].length; j++){
        System.out.println("value"+"["+i+"]"+"["+j+"]"+"=" + twoDim[i][j]);
    }
}
```

* JDK 5 부터 추가된 개선된 for 문을 사용해 위 코드를 개선해보자&#x20;
  * 하지만 개선된 for 문은 배열의 위치를 모른다는 것이다.. 이를 위해서 임시 변수를 사용해야 하는데, 그렇게 불편하게 사용할 것이라면 그냥 for 문을 사용하는게 낫지 않을까?

```java
int [][]twoDim = {{1,2,3}, {4,5,6}};
System.out.println("twoDim.length=" + twoDim.length);
System.out.println("twoDim[0].length=" + twoDim[0].length);

for(int i=0; i<twoDim.length; i++){
    for(int j=0; j<twoDim[i].length; j++){
        System.out.println("value"+"["+i+"]"+"["+j+"]"+"=" + twoDim[i][j]);
    }
}

for(int[] Dim : twoDim){
    for(int i : Dim){
         System.out.println("value=" + i);
    }
}
```
