# File 기본

## File 생성자&#x20;

* File(File parent, String child) : 이미 생성되어 있는 File 객체와 그 경로의 하위 경로 이름으로 새로운 File 객체를 생성한다. \
  \-> child 라고 되어 있는 값은 경로가 될 수도 있고, 파일 이름이 될 수도 있다.&#x20;
* File(Srting pathName) : 지정한 경로 이름으로 File 객체를 생성한다.&#x20;
* File(String parent, String child) : 상위 경로와 하위 경로로 File 객체를 생성한다.&#x20;
* File(URI uri) : URI 따른 File 객체를 생성한다.&#x20;

```java
public class FileSample{
    public static void main(String[] args){
        FileSample sample = new FileSample();
        // 윈도우에서 보통 디렉토리 구조는 C 드라이브에서 시작된다.
        // 또한, 문자열에서 \기호는 뒤에오는 글자에 따라서 특수기호로 인식하기에 
        // \\ 같이 두개를 써주어야 한다. -> 유닉스 개열에서는 /로 디렉토리를 구분한다. 
        String pathName = "C:\\godofjava\\text";
        sample.checkPath(pathName);
    }    
    
    public void checkPath(String pathName){
        File file = new File(pathName);
        System.out.println(pathName + "is exists? =" + file.exists());
    }
    
    public void makeDir(String pathName){
        File file = new File(pathName);
        System.out.println("Make " + pathName + "result = " + file.mkdir());
    }
}
```

## File 클래스를 이용해 파일을 처리하자&#x20;

```java
public class FileManageClass{
    public static void main(String[] args){
        FileManageClass sample = new FileManageClass();
        String pathName = file.separator + "godofjava" + File.separator + "text";
        String fileName = "test.txt";
        
        sample.checkFile(pathName, fileName);
    }
    
    public void checkFile(String pathName, String fileName){
        File file = new File(pathName, fileName);
        try{
            System.out.println("Create result = " + file.createNewFile());
        } catch(IOException e){
        
        }
    }
}
```
