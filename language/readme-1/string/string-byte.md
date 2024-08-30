# String 문자열을 byte 로 변환하기

* 한글을 사용하는 우리나라에서는 아래의 String 생성자를 많이 사용할 수 밖에 없다.
* 왜냐면, 대부분의 언어에서는 문자열을 변환할 때 기본적으로 영어로 해석하려 하기 때문이다.

```java
String(byte[] bytes)
String(byte[] bytes, String charsetName)
```

## String 문자열을 byte 로 변환하기

```java
byte[] getBytes()    // 기본 캐릭터 셋의 바이트 배열을 생성
byte[] getBytes(Charset charset)    // 지정한 캐릭터 셋 객체 타입으로 바이트 배열을 생성    
byte[] getBytes(String charsetName) // 지정한 이름의 캐릭터 셋을 갖는 바이트 배열을 java
```

* 생성자의 매개 변수로 받는 byte 배열을 어떻게 생성 할지는 고민할 필요가 없다.
* String 클래스에는 현재의 문자열 값을 배열로 변환하는 getBytes() 메서드가 있기 때문이다.
* 보통 캐릭터 셋을 잘 알고 있거나, 같은 프로그램 내에서 문자열을 byte 배열로 만들 때에는 가장 첫번째 메서드를 사용한다.
* 하지만, 다른 시스템에서 전달 받은 문자열을 byte 배열로 변환할 때에는 두번째나 세번째 메서드를 사용하는 것이 좋다.\
  \-> 다른 캐릭터 셋으로 문자열이 셋팅되어 있을 수 있기 때문이다.

> #### 캐릭터 셋(Charset)
>
> * 언어를 표현하는 타입? 정도로 생각하면 된다.
> * 자바뿐 아니라, 어떤 프로그래밍 언어를 사용할 경우에는 특수문자(알파벳 외의 글자)를 표시할 일이 생긴다.
> * 한글도 기본적으로 알파벳이 아니기 때문에, 고유의 캐릭터 셋을 가진다.
> * **최근 한글을 처리하기 위해서 사용되는 가장 많이 사용되는 캐릭터 셋은 UTF-16이다.**\
>   **(과거에는 UTF-8, EUC-KR 을 많이 사용했다.)**

```java
public void convert(){
    try{
        String korean = "한글";
    
        byte[] bytes = korean.getBytes();
        printByBytes(bytes);
    
        String korean2 = new String(bytes);
        System.out.println("korean2 = " + korean2);
    
    
    }catch (Exception e){
        e.printStackTrace();
    }
}

public void convertUTF16(){
    try{
        String korean = "한글";

        byte[] bytes = korean.getBytes("UTF-16");
        printByBytes(bytes);

        // 문자열을 특정 캐릭터 셋으로 설정해줄때는, String 객체를 생성시 매핑되는 캐릭터 셋을 설정해 주어야 문자열이 깨지지 않는다.
        String korean2 = new String(bytes, "UTF-16");
        System.out.println("korean2 = " + korean2);


    }catch (Exception e){
        e.printStackTrace();
    }
}

private static void printByBytes(byte[] bytes) {
    for (byte data : bytes){
        System.out.print(data + " ");
    }
    System.out.println();
}
```

* 문자열을 byte 로 변환 후 다시 문자열로 변환하는 코드는 위와 같다.
* convert() 메서드는 가장 기본적인 메서드이다.
* convertUTF16 메서드를 살펴보게 되면 getBytes() 메서드 호출 시 특정 캐릭터 셋을 설정해주게 되는데,\
  **byte 배열을 문자열로 변환시 캐릭터 셋을 함께 지정해주지 않는다면 문자는 깨지게 된다.**

