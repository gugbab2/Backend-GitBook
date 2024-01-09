# 각종 정보를 확인하는 System 클래스

* System 클래스의 가장 큰 특징은, 생성자가 없다는 것이다.&#x20;
* System 클래스는 아래의 3가지 변수를 갖는다.
  * static PrintStream err : 에러 및 오류를 출력할 때 사용
  * static InputStream in : 입력값을 처리할 때 사용
  * static PrintStream out : 출력값을 처리할 때 사용\
    \=> 위 3가지 메서드 모두 리턴타입이 입출력과 관련된 클래스이다. (IO)

## 1. 시스템 속성(Property) 값 관리

* **static String clearProperty(String key)** : key 에 저장된 시스템 속성을 제거한다.
* **static Properties getProperties()** : 현재 시스템 속성을 Properties 클래스 형태로 제공한다.
* **static String getProperty(String key)** : Key 에 지정된 문자열로 된 시스템 속성값을 얻는다.
* **static String getProperty(String key, String def)** : Key 에 지정된 문자열로된 시스템 속성값을 얻고, 만약 없으면, def에 지정된 값을 리턴한다.&#x20;
* **static void setProperties(Properties props)** : 매개변수로 넘겨주는 변수에 있는 값들을 시스템 속성에 넣는다.
* **static String setProperty(String key, String value)** : key 에 지정된 시스템 속성의 값을 value로 대체한다. \
  \=> **자바를 실행하면 Properties 객체가 생성되며, 그 값은 언제, 어디서 든지 **_**같은 JVM**_** 내에서는 꺼낼 수 있다.**

> Properties 는 java.util 패키지에 속하며, Hashtable 의 상속을 받은 클래스이다.&#x20;

## 2. 시스템 환경(Enviroment) 값 조회

* **static Map\<String, String> getenv()** : 현재 시스템 환경에 대한 Map 형태의 리턴값을 받는다.
* **static String getenv(String name)** : 지정한 name 에 해당하는 값을 받는다.\
  \=> **Properties 값은 추가할 수도, 변경할 수도 있다. 하지만 **_**env 값은 변경할 수 없고 읽을 수만 있다.**_

## 3. GC 수행(GC 는 JVM 이 필요할 때 알아서 하니, 사용하지 말자..)

* **static void gc()** : 가비지 콜렉터를 실행한다.
* **static void runFinalization** : GC 처리를 기다리는 모든 객체에 대하여 finalize() 메소드를 실행한다.&#x20;

## 4. JVM 종료(절대 사용하지 말아라...!)

* **static void exit(int status)** : 현재 실행중인 JVM 을 멈춘다.

## 5. 현재 시간 조회

* **static long currentTimeMillis()** : 현재 시간을 밀리초 단위로 리턴.
* **static log nanoTime()** : 현재 시간을 나노초 단위로 리턴.\
  \-> 시간 측정을 위해서 만든 메서드
