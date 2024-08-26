# 어노테이션의 사용

## 어노테이션 선언

```java
@Target(ElementType.METHOD)                                    //1
@Retention(RetentionPolicy.RUNTIME)                            //2
public @interface UserAnnotion {                               //3
    public int number();                                       //4
    public String text() default "This is first annotion";     //5
}
```

* 1 : 해당 어노테이션의 사용 대상 정의
* 2 : 해당 어노테이션의 유지 정보를 지정
* 3 : interface, class 와 같이 @interface 로 선언하게 되면, @UserAnnotion 어노테이션으로 사용할 수 있다.
* 4 : 메서드를 어노테이션 안에 선언하게되면, 해당 어노테이션을 사용할 때 해당 항목에 대한 타입으로 지정 가능하다.
* 5 : default 는 해당 어노테이션의 기본값이 된다.

## 어노테이션 사용

```java
public class UserAnnotationSample{
    @UserAnnotion(number=0)
    public static void main(String[] args){
        UserAnnotationSample sample = new UserAnnotationSample();
    }
    
    @UserAnnotion(number=1)
    public void annotationSample1(){
    }
    
    @UserAnnotion(number=2, text="second")
    public void annotationSample2(){
    }
    
    @UserAnnotion(number=3, text="third")
    public void annotationSample3(){
    }
}
```

## 어노테이션 선언한 값 확인(거의 사용할 일은 없다..)

```java
public checkAnnotations(class useClass){
    Method[] methods = useClass.getDeclaredMethods();
    for(Method tempMethod : methods){
        UserAnnotation annotation = tempMethod.getAnnotation(UserAnnotation.class);
        if(annotation != null){
            int number = annotation.number();
            String text = annotation.text();
            
            System.out.println(number + "," + text);
        }    
    }
}
```
