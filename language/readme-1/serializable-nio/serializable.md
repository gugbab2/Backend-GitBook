# Serializable

## Serializable 기본

* 기존의 자바의 클래스들을 살펴보며 **Serializable** 인터페이스를 확장한 모습을 많이 볼 수 있었다.
* **해당 인터페이스는 구현해야 할 메서드가 없다...**\
  **-> 해당 인터페이스를 확장하는 것 만으로 파일 I/O, 네트워킹 프로그래밍이 가능해진다.**
* 해당 인터페이스를 확장 후 다음과 같이 serialVersionUID 값을 지정해주는 것을 권장한다.\
  \-> 만약 지정하지 않는다면, 자바 소스가 컴파일 될 때 자동으로 생성된다.\
  \-> 반드시 아래와 같은 형식으로 지정해야만 자바에서 인식한다.

```java
static final long serialVersionUID = 1L;
```

* 왜 위와 같은 값을 지정해야할까?
  * 이 값은, 해당 객체의 버전을 명시하는데 사용된다.
  * 예를 들어, A 서버가 B서버로 SampleDTO 객체를 전송하려 할때, 두 서버 모두에 SampleDTO 를 가지고 있어야 한다.\
    \-> 만약 A 서버 DTO 가 3개의 변수, B 서버 DTO 가 4개의 변수를 가지고 있는다고 하면, serialVersionUID 값에 따라서 다른 클래스로 인식한다.\
    **-> 즉, 클래스 이름이 같더라도, 해당 UID 가 다르면, 다른 클래스로 인식한다.**\
    **-> 또, 같은 UID라 하더라도, 변수의 개수, 타입이 다르면 다른 클래스로 인식한다.**

## 객체를 저장하자

* 다음과 같이 DTO 객체를 만들고\
  **-> Serializable 인터페이스를 구현하지 않았다면 I/O & 네트워크 통신 시 정상적으로 실행되지 않는다..**\
  **-> I/O & 네트워크 통신 시에는 Serializable 인터페이스 구현이 필수적이다.**

```java
import java.io.Serializable;

public class SerialDTO implements Serializable {
    private String bookName;
    private int bookOrder;
    private boolean bestSeller;
    private long soldPerDay;

    public SerialDTO(String bookName, int bookOrder, boolean bestSeller, long soldPerDay) {
        super();
        this.bookName = bookName;
        this.bookOrder = bookOrder;
        this.bestSeller = bestSeller;
        this.soldPerDay = soldPerDay;
    }

    @Override
    public String toString() {
        return "SerialDTO{" +
                "bookName='" + bookName + '\'' +
                ", bookOrder=" + bookOrder +
                ", bestSeller=" + bestSeller +
                ", soldPerDay=" + soldPerDay +
                '}';
    }
}

```

* 다음과 같이 데이터를 쓰고, 읽는 클래스를 만들자

```java
import java.io.*;

public class ManageObject {
    public static void main(String[] args){

        ManageObject manage = new ManageObject();
        String fullPath= "serial.obj";  //해당 파일은 객체가 바이너리로 저장되어 있어서 일반 텍스트 파일로 읽기가 힘들다.
        SerialDTO dto = new SerialDTO("GodOfJava", 1, true, 100);
        manage.saveObject(fullPath, dto);
        manage.loadObject(fullPath);

    }

    private void saveObject(String fullPath, SerialDTO dto) {
        FileOutputStream fos = null;
        ObjectOutputStream oos = null;

        try{
            fos = new FileOutputStream(fullPath);
            oos = new ObjectOutputStream(fos);
            oos.writeObject(dto);
            System.out.println("write success");

        }catch (Exception e){
            e.printStackTrace();
        }finally {
            if (oos != null) {
                try {
                    oos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }

            if (fos != null) {
                try {
                    fos.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }
    }

    public void loadObject(String fullPath) {
        FileInputStream fis = null;
        ObjectInputStream ois = null;
        try {
            fis = new FileInputStream(fullPath);
            ois = new ObjectInputStream(fis);
            Object obj = ois.readObject();
            SerialDTO dto = (SerialDTO) obj;
            System.out.println("dto = " + dto);
        } catch (Exception e) {
            e.printStackTrace();
        }finally {
            if (ois != null) {
                try {
                    ois.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
            if (fis != null) {
                try {
                    fis.close();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        }

    }
}

```

## Serializable 를 구현한 클래스르 변경해보자

```java
public class SerialDTO implement Serializable{
    private String bookType = "IT";
    // 이하 생략..
}
```

* 다음과 같이 인스턴스 변수를 추가 후 실행 시 InvalidClassException 에러가 난다.
* **이유는, 같은 UID라 하더라도, 변수의 개수, 타입 등이 다르면 다른 클래스로 인식한다.**

```java
public class SerialDTO implement Serializable{
    static final long serialVersionUID = 1L;
    
    private String bookType = "IT";
    // 이하 생략..
}
```

* **다음과 같이, serialVersionUID 를 명시적으로 선언한 것 같이, 다른 객체로 인식하게 된다.**
* **만약 데이터가 변경되면 다음과 같이 serialVersionUID를 명시적으로 선언하는 것이 권장된다.**\
  **-> 운영상황에서, serializable 한 객체의 내용이 바뀌었는데 오류가 발생하지 않는다면 데이터가 꼬일 수 있기 때문이다.**

## transient 라는 예약어는 Serializable 과 뗄 수 없는 관계다

```java
import java.io.Serializable;

public class SerialDTO implements Serializable {
    private String bookName;
    private int bookOrder;
    transient private boolean bestSeller;
    private long soldPerDay;

    ...
}

```

* transient 예약어를 사용하게 되면, Serializable 대상에 제외가 된다!(꼭 기억하자!)
