# CLI 환경에서 .java 파일 컴파일 && 실행

자바를 사용하며 인텔리제이, 이클립스 등 툴을 사용하며 개발을 했었기에, 내부적으로 어떻게 컴파일 되고 실행이 되는지에 대해 관심을 가져본 적이 없다.

간단한 .java 파일을 컴파일하고 실행해보며 그 과정을 들여다보고자 한다.

```java
package org.example;

public class Main {
    public static void main(String[] args) {
        System.out.println("Hello world!");
    }
}
```

## 문제 상황

Main.java 파일을 컴파일 하기 위해서 javac Main.java 명령어를 사용해 컴파일을 하게 되면 Main.class 파일이 생긴 것을 볼 수 있다.\
(해당 클래스 파일은 바이너리 파일로 컴퓨터가 이해할 수 있는 형태로 재구성 된 형태로 생각하면 된다.)

해당 클래스 파일을 실행하기 위해서 java Main 명령어를 실행하게 되면 다음의 에러 메시지를 띄우면 정상적으로 실행되지 않는 것을 볼 수 있다.\
"Error : Could not find or load main class Main"

<figure><img src="../../../../.gitbook/assets/Screen Shot 2023-02-18 at 14.12.52.png" alt=""><figcaption></figcaption></figure>

Main.java 파일을 들여다 보면 패키지가 org.example 로 설정된 것을 볼 수 있다.\
자바 명령어를 실행할 때는 FQCN 로 실행시켜야 정상 작동한다.\
(FQCN(Full Qualify Class Name) → FULL 패키지 이름 + 클래스 이름)

하지만 FQCN 형식인 java org.example.Main 으로 실행시켜도 동일한 오류가 발생할 것이다..\
왜냐하면, CLASSPATH 환경 변수가 없기 때문에 -classpath 또는 -cp 옵션을 사용하여 경로를 제안 하지 않으므로 기본적으로 Java는 현재 디렉토리에서만 검색합니다 .\
\-> **기본적으로 자바는 해당 패키지의 가장 상위 패키지(root) 에서 실행을 해야만 한다는 약속이 있다.(그냥 약속이다!)**

현재 우리는 org.example 디렉토리 안에 들어와 있는 상태로, 해당 클래스 파일을 찾을 수 없다.\
만약 java 디렉토리에서 java org.example.Main 으로 실행시킨다면 아래와 같이 정상적으로 작동하는 것을 볼 수 있다.

<figure><img src="../../../../.gitbook/assets/Screen Shot 2023-02-18 at 14.28.11.png" alt=""><figcaption></figcaption></figure>

## 추가적으로..

만약 .java 파일에 패키지가 없다면 어떻게 될까?

별다른 고민 없이 컴파일 한 디렉토리에서 클래스파일을 실행시켜도 아래와 같이 정상적으로 작동하는 것을 볼 수 있다.

<figure><img src="../../../../.gitbook/assets/Screen Shot 2023-02-18 at 14.30.45.png" alt=""><figcaption></figcaption></figure>
