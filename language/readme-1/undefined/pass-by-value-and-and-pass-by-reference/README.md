# Pass by Value && Pass By Reference

Pass By Value : 값만 전달한다.(**기본자료형**)\
\-> **기존 값에 영향이 없다.**

Pass By Reference : 값이 아닌, 객체의 참조를 전달한다.(**참조자료형**)\
\-> **기존 값에 영향을 미친다.**\
**\* 참조 : 객체의 주소라고 생각하면 이해가 쉽다.**&#x20;

```java
public class Main {
    public static void main(String[] args) {
        PassBy passBy = new PassBy();
        passBy.callPassByValue();    //기존 값 유지.. 
        passBy.callByReference();    //기존 값 변경..
    }
}

// callPassByValue
public void callPassByValue(){
    int a = 10;
    String b = "b";
    System.out.println("before passByValue");
    System.out.println("a = " + a);
    System.out.println("b = " + b);

    passByValue(a,b);
    System.out.println("after passByValue");
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}

public void passByValue(int a, String b){
    a = 20;
    b = "z";
    System.out.println("in passByValue");
    System.out.println("a = " + a);
    System.out.println("b = " + b);
}

// callByReference
public void callByReference(){
    MemberDTO memberDTO = new MemberDTO("Gugbab2");
    System.out.println("before passByReference");
    System.out.println("member.name = " + memberDTO.name);

    passByReference(memberDTO);
    System.out.println("after passByReference");
    System.out.println("member.name = " + memberDTO.name);
}

public void passByReference(MemberDTO member){
    member.name = "Kyeongho";
    System.out.println("in passByReference");
    System.out.println("member.name = " + member.name);
}
```

