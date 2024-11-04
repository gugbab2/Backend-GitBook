# Reactor pattern 이란?

> 참고 링크
>
> [https://i5on9i.blogspot.com/2013/11/reactor-pattern.html](https://i5on9i.blogspot.com/2013/11/reactor-pattern.html)

## 1. 정의&#x20;

* 일단 정의를 살펴보면,&#x20;
* 요청을 처리하는 쪽(서버) 으로 동시에 요청이 들어온 service request 를 처리하기 위한 event handling pattern 이라고 한다.
* 이 때, 서버에서는 들어온 요청들을 분류해서 각 요청들이 적절한 request hendler 에 dispatch 한다.&#x20;
  * 이 때, 동시에 여러개의 요청을 처리하는 것이 아니라, 하나의 대한 처리를 끝내고 그 다음 것을 처리하는 방식으로 하나씩 처리한다.&#x20;
* Reactor pattern 을 이용한 여러 server 가 있는데, Node.js 도 그 중 하나이다.&#x20;

## 2. 기존의 ThreadPool 을 사용하는 케이스&#x20;

<figure><img src="../../../.gitbook/assets/image (148).png" alt=""><figcaption></figcaption></figure>

* 기존의 ThreadPool 을 사용하는 케이스를 고려 해 보자.&#x20;

1. ServerSocket 으로 request A 가 들어오면 Thread 를 할당 해 준다.&#x20;
2. 그럼 이 Thread 는 그 socket 을 가지고 read,write 작업(I/O) 를 할 것이다. 그런데 이 와중에 ServerSocket 에 request B 가 들어오면, 컨텍스트 스위칭이 일어난다. &#x20;
3. 그럼 새로 들어온 request 에 대해 Thread 를 배분해주고, 또 이 socket 으로 read, write 작업을 할 것이다.&#x20;
4. 그러면서 A 의 작업을 하기 위해 중간에 다시 컨텍스트 스위칭을 하고, 그리다 다시 B 를 작업하려고 컨텍스트 스위칭을 반복하며 모든 작업이 완료될 것이다.&#x20;

* 그런데 request A 가 들어와서 Thread 가 만들어진 후 또 다른 request 가 들어오지 않는 상황을 고려해보자.
  * 그럼 request A 를 처리하다가 중간에 request 가 들어왔는지 확인하기 위해서 컨텍스트 스위칭을 해야  한다.&#x20;
  * 만약 요청이 들어왔다면 Thread 를 만들어주고, 그렇지 않다면 다시 request A 를 처리하러 돌아갈 것이다.&#x20;
* 요청이 들어왔는지 확인하기 위해서 컨텍스트 스위칭이 이루어지는 시간이 아깝다 ..

```java
class Server implements Runnable {
   public void run() {
       try {
           ServerSocket ss = new ServerSocket(PORT);
           while (!Thread.interrupted())
               new Thread(new Handler(ss.accept())).start();
        // or, single-threaded, or a thread pool
       } catch (IOException ex) { /* ... */ }
   }
   static class Handler implements Runnable {
       final Socket socket;
       Handler(Socket s) { socket = s; }
       public void run() {
           try {
               byte[] input = new byte[MAX_INPUT];
               socket.getInputStream().read(input);
               byte[] output = process(input);
               socket.getOutputStream().write(output);
           } catch (IOException ex) { /* ... */ }
       }
       private byte[] process(byte[] cmd) { /* ... */ }
    }
}
```

## 3. Reactor Pattern 을 사용하는 케이스&#x20;

* 위와 같이 ThreadPool 사용 예제에서 발생하는 컨텍스트 스위칭 문제를 해결하기 위해서 나온 것이 Reactor Pattern 이다.&#x20;
* Reactor 는 event 가 발생하기를 기다리고, event 가 발생하면 Reactor 는 이 event 를 적절한 event handler 에 넘겨주는 역할을 하게 된다.&#x20;
* Reactor Pattern 에는 중요한 2가지 요소가 있다.&#x20;
  * **하나는, event 를 받고 전달해주는 Reactor**
  * **둘째는, Reactor 가 보낸 event 를 실제로 처리하는 Handler 가 있다.**
