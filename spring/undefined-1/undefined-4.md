# 빈 생명주기 콜백

## 1. 빈 생명주기 콜백의 시작

* 디비 커넥션 풀이나, 네트워크 소켓처럼 애플리케이션 시작 시점에 필요한 연결을 미리 해두고,\
  애플리케이션 종료 시점에 연결을 모두 종료하는 작업을 진행하려면 객체의 초기화 종료 작업이 필요하다.

### 1-1. 스프링 빈의 라이프사이클

* 객체 생성 -> 의존관계 주입
* **스프링 빈은 객체를 생성하고, 의존관계 주입이 다 끝난 다음에야 필요한 데이터를 사용할 수 있는 준비가 완료된다.**\
  -> 개발자는 의존관계가 주입된 시점을 어떻게 아는가?
* 스프링은 의존관계 주입이 완료되면 스프링 빈에게 콜백 메서드를 통한 초기화 시점을 알려주는 다양한 기능을 제공한다.
* 스프링 빈의 이벤트 라이프사이클은 다음과 같다.
  * **스프링 컨테이너 생성 -> 스프링 빈 생성 -> 의존관계 주입 -> 초기화 콜백**\
    &#xNAN;**-> 사용 -> 소멸전 콜백 -> 스프링 종료**

> **객체의 생성과 초기화를 분리하자!**
>
> **생성자는 필수 정보 파라미터를 받고, 메모리를 할당해서 객체를 생성하는 책임을 가진다.**\
> **반면 초기화는 이렇게 생성된 값을 가지고 외부 커넥션을 연결하는 등 무거운 작업을 수행한다.**
>
> **때문에, 생성과 초기화를 명확하게 나누는 것이 유지보수 관점에서 좋다.**

## 2. 애노테이션 @PostConstruct, @PreDestroy

```java
public class NetworkClient {

      private String url;
      
      public NetworkClient() {
            System.out.println("생성자 호출, url = " + url); 
      }
      
      public void setUrl(String url) {
          this.url = url;
      }
      
      //서비스 시작시 호출
      public void connect() {
          System.out.println("connect: " + url);
      }
      
      public void call(String message) {
          System.out.println("call: " + url + " message = " + message);
      }
      
      //서비스 종료시 호출
      public void disConnect() {
          System.out.println("close + " + url);
      }
    
      @PostConstruct
      public void init() {
          System.out.println("NetworkClient.init"); 
          connect();
          call("초기화 연결 메시지");
     
      }
    
      @PreDestroy
      public void close() {
         System.out.println("NetworkClient.close");
         disConnect();
      }
}  
        
        
@Configuration
static class LifeCycleConfig {

    @Bean
    public NetworkClient networkClient() {
         NetworkClient networkClient = new NetworkClient();
         networkClient.setUrl("http://hello-spring.dev");
         return networkClient;
    } 
} 
```

### 2-1. @PostConstruct, @PreDestroy 애노테이션 특징

* 최신 스프링에서 권장하는 방법으로 매우 편리하다.
* 패키지를 보면 스프링의 종속된 기술이 아닌, 자바 표준의 기술이다.\
  -> 따라서 스프링이 아닌 다른 컨테이너에서도 동작한다.
* 컴포넌트 스캔과 매우 잘 어울린다.
