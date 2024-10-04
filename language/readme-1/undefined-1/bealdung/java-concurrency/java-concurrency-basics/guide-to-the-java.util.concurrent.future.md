# Guide to the java.util.concurrent.Future

> 참고 링크&#x20;
>
> [https://www.baeldung.com/java-future](https://www.baeldung.com/java-future)\
> [https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/Future.html)

## 1. Future 인스턴스 생성&#x20;

* Future 클래스는 비동기 연산에 대한 상태 및 결과를 확인할 수 있다.&#x20;
* 오래 걸리는 작업(메서드) 에 대해서 비동기 통신을 사용하게 되면 완료될 때까지 기다리지 않고 다른 프로세스를 실행시킬 수 있기 때문에,   Future 클래스를 통한 비동기 처리는 좋은 선택지가 될 수 있다.&#x20;
* Future 의 비동기적 특성을 활용하는 작업의 예는 다음과 같다.&#x20;
  * 계산 집약적 프로세스(수학적, 과학적 계산)&#x20;
  * 대용량 데이터 구조 조작
  * 원격 메서드 호출(파일 다운로드, HTML 스크래핑, 웹서비스)

### 1-1. FutureTask 를 사용하여 Futures 구현&#x20;

* 예를 들어, `Integer` 의 제곱을 계산하는 매우 간단한 클래스를 만든다 생각하자.&#x20;
* `Callable` 은 결과를 반환하는 작업을 나타내는 인터페이스이며, 단일 `call()` 메서드가 있다. 여기서는 람다 표현식을 사용하여 나타냈다.&#x20;
* &#x20;`Callable` 인스턴스를 생성하고 `submit` 의 매개변수로 전달하면, 내부적으로 `FutureTask` 객체를 생성하고, 이 `FutureTask` 가 쓰레드 풀에서 실행될 작업을 나타낸다.&#x20;
  * 구체적으로는 `Callable`, `Runnable` 작업을 받아서 이를 감싸는 `FutureTask` 를 만들고, 이를 쓰레드 풀에 제출한다.
  * 이 `FutureTask` 는 비동기 작업의 결과를 처리하고, 그 결과를 `Future` 인터페이스를 통해 반환한다.&#x20;

```javascript
public class SquareCalculator {
    
    private ExecutorService executor = Executors.newSingleThreadExecutor(); 
    
    public Future<Integer> calculate(Integer input) {
        return executor.submit(() -> {
            Thread.sleep(1000);
            return input * input;
        });
    }
}
```

