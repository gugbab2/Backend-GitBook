# G1GC 오라클 공식 레퍼런스 정리

#### 참고 링크&#x20;

[https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html#GUID-0394E76A-1A8F-425E-A0D0-B48A3DC82B42](https://docs.oracle.com/en/java/javase/21/gctuning/garbage-first-g1-garbage-collector1.html#GUID-0394E76A-1A8F-425E-A0D0-B48A3DC82B42)

## 1. Garbage-First(G1) 가비지 컬렉터 소개 <a href="#jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42" id="jsgct-guid-0394e76a-1a8f-425e-a0d0-b48a3dc82b42"></a>

G1(Garbage-First) 가비지 컬렉터는 다중 프로세서 시스템과 대용량 메모리 환경을 위해 설계되었다. 목표는 높은 처리량을 유지하면서도 STW 를 예측 가능하게, 그리고 짧게 가져가는 것이다. 이를 위해 G1 은 최소한의 설정으로 최적의 성능을 제공하려 한다.&#x20;

G1 이 특히 목표로 하는 환경 및 애플리케이션의 특징은 다음과 같다.&#x20;

* 힙의 크기가 수십 GB 이상으로 매우 크고, 사용중인 데이터가 Java 힙의 50% 이상을 차지하는 경우&#x20;
* 객체 할당 및 승격 속도가 시간에 따라 크게 변동할 수 있는 경우&#x20;
* 힙 내부에 파편화가 상당한 경우&#x20;
* 예측 가능한 STW 를 통해 긴 STW 를 피해야 하는 경우&#x20;

G1 은 애플리케이션이 실행되는 **동시에 일부 작업을 수행**하여 짧은 STW 를 달성한다. 이는 애플리케이션에 할당될 수 있는 프로세서 자원을 가비지 컬렉션에 일부 할애하는 방식이다. **결과적으로 G1 은 다른 처리량 중심 GC 보다 STW 가 훨씬 짧지만, 애플리케이션 전체 처리량은 낮아지는 경향이 있다.** JDK 9 부터 G1 은 Default GC 이다.&#x20;

## 2. 기본 개념&#x20;

G1 은 각 STW 시 목표 중지 시간을 모니터링한다.&#x20;

다른 컬렉터와 마찬가지로 G1 은 힙을 신세대와 구세대로 나눈다. 공간 회수 작업은 가장 효율적인 신세대에 집중되며, 구세대에서도 간헐적으로 공간 회수가 수행된다.&#x20;

