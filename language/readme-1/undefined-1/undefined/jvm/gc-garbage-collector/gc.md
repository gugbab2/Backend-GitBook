# GC 튜닝

> 참고 링크&#x20;
>
> [https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98-GC-%ED%8A%9C%EB%8B%9D-%EB%A7%9B%EB%B3%B4%EA%B8%B0](https://inpa.tistory.com/entry/JAVA-%E2%98%95-%EA%B0%80%EB%B9%84%EC%A7%80-%EC%BB%AC%EB%A0%89%EC%85%98-GC-%ED%8A%9C%EB%8B%9D-%EB%A7%9B%EB%B3%B4%EA%B8%B0)
>
> [https://d2.naver.com/helloworld/37111](https://d2.naver.com/helloworld/37111)
>
> [https://d2.naver.com/helloworld/6043](https://d2.naver.com/helloworld/6043)

* Java 가 C 언어에 비해 속도 차이가 나는 이유는 JVM 에 있는데, 미리 바이너리 코드로 컴파일 되는 C 언어에 비해서 자바는 바이트 코드라는 중간 단계 컴파일을 해석하는데 있어 시간이 소요되기 때문이다.&#x20;
* 그리고 무엇보다 자바 어플리케이션의 성능의 가장 큰 비중을 차지하는게 바로 GC 의 stop-the-world 이다..&#x20;
* GC 성능을 향상시키기 위해서 성능 좋은 GC 알고리즘을 선택하면 되지만, 이도 문제가 해결이 안된다면 비로서 GC 튜닝 이라는 것을 해야한다!&#x20;

## GC 튜닝의 주의점

### 첫째, GC 튜닝 옵션은 서비스의 특징 마다 적정 값이 다르다는 것이다. &#x20;

* 예를 들어 '누가 이 옵션값을 사용했을 때 성능이 잘 나왔으니, 우리도 이렇게 적용하자' 라고 생각하면 절대 안된다!
* 왜냐하면 서비스 주제와 특징마다 생성되는 객체의 크기도 다르고 살아있는 기간도 다르기 때문이다.&#x20;
* 따라서 별도의 성능 모니터링을 통해서, 어느 지점에서 GC stop-the-world 문제가 나는지 서비스마다 제각각 파악하여 GC 튜닝을 해야한다.&#x20;

### 둘째, GC 튜닝은 가장 마지막에 하는 작업이라는 것이다.&#x20;

* GC 튜닝은 필요한 선행 지식과, 신경써야할 요소, 리스크에 비해서 얻어가는 부분이 적기 때문에, GC 튜닝보다 애플리케이션 코드로써 메모리 최적화를 더 신경 쓸것을 권장하자는 의견도 있다.&#x20;
* **GC 튜닝을 근본적으로 왜할까?**&#x20;
  * **일반적으로 Java 에서 생성된 객체는 GC 를 통해서 메모리를 해제한다.**&#x20;
  * **즉, 생성된 객체가 많으면 많을수록 GC 가 처리해야 하는 대상도 많아지고, 횟수도 많아진다는 소리이다.**&#x20;
  * **그리고, GC 의 수행 횟수가 늘어나면 stop-the-world 횟수도 많아지니 성능에 영향이 가게 된다.**&#x20;
* **따라서 GC 튜닝을 하기 전에, 쓸모없는 객체 생성을 줄이는 리팩토링 작업이 선행되어야 근본적이 해결이 될 수 있다.**&#x20;

## GC 튜닝의 목표&#x20;

### 첫째, Old 영역으로 넘어가는 객체의 수 최소화하기!&#x20;

* **기본적으로 Old 영역의 크기는 Young 영역의 크기보다 훨씬 거대하다. 그 이유는 다음과 같다.**&#x20;
  * 객체의 생명주기 : **Old 영역에 있는 객체들은 비교적 긴 생명주기를 가지게 되는데**, 이 객체들은 메모리에서 긴 시간 유지될 가능성이 크므로, **이를 관리하기 위한 많은 양의 메모리 공간을 필요로 한다.**&#x20;
  * GC 성능 최적화 : **Old 영역에 큰 메모리 공간이 있으면 Full GC, Major GC 가 자주 발생하지 않도록 설계할 수 있다.** GC 는 메모리 공간을 확보하기 위한 작업으로, Old 영역의 메모리 공간이 작으면 GC 가 더 자주 실행할 수밖에 없다. 그러나, Old 영역이 충분히 크면 더 오랫동안 GC 를 피할 수 있고, GC 의 빈도를 줄여 성능을 최적화할 수 있다.&#x20;
* 따라서, Old 영역의 GC 는 Young 영역의 GC 에 비해서 상대적으로 긴 시간이 소요되기 때문에, 애초에 Old 영역으로 이동하는 객체의 수를 줄이면 Full GC 가 발생하는 빈도를 많이 줄일 수 있게 된다.&#x20;
* **이말은 즉, Young 영역의 크기를 잘 조절하여, Old 영역으로 넘어가는 빈도를 줄이면 큰 효과를 볼 수 있다는 의미이다.**&#x20;

### 둘째, Full GC 시간 줄이기&#x20;

* Full GC 의 실행 시간은 상대적으로 Minor GC 에 비해서 길기 때문에, Old 영역의 크기를 적절하게 설정하는 것도 하나의 방법이다.&#x20;
* 그렇다고 Old 영역의 크기를 막 줄여버리면 자칫 OutOfMemoryError 가 발생하거나, Full GC 횟수가 늘어날 수 있다.&#x20;
* 반대로 Old 영역의 크기를 늘리면 Full GC 횟수는 줄어들지만 실행 시간이 늘어나게 된다.&#x20;
* 즉, GC 튜닝의 포인트는 이 둘 사이를 잘 아우리는 적정 범위를 찾는 것이라고 할 수 있다.&#x20;

## GC 튜닝 맛보기 (가장 간단한 수준.. )&#x20;

### 1. GC 상황 모니터링&#x20;

* jstat 명령어는 JDK 1.6 부터 제공되는 기본 모니터링 및 분석 툴이다.&#x20;

```bash
# jstat -gcutil  명령어로 현재 실행중인 8884번 프로세스에 대해 
# 1초에 한번 씩 총 10번 GC와 관련된 정보를 출력하도록 모니터링
jstat -gcutil -t 8844 1000 10
```

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2024-09-24 19.03.25.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2024-09-24 19.04.09.png" alt=""><figcaption></figcaption></figure>

### 2.모니터링 결과 분석 후 GC 튜닝 여부 결정&#x20;

* GC 상황을 확인한 후에는, 결과를 분석하고 GC 튜닝 여부를 결정해야 한다.&#x20;
  * Minor GC 수행시간 : YGCT / YGC (0.314 / 19) = 0.016초&#x20;
  * Major GC 수행시간 : FGCT / FGC (0.291 / 3) = 0.097초&#x20;

<figure><img src="../../../../../../.gitbook/assets/스크린샷 2024-09-24 19.10.46.png" alt=""><figcaption></figcaption></figure>

* 만약 모니터링 결과가 다음의 조건에 모두 부합한다면, GC 튜닝이 굳이 필요하지는 않다.&#x20;
  * **Minor GC 처리 시간이 빠르다. (50ms 내외)**&#x20;
  * **Minor GC 주기가 빈번하지 않다. (10초 내외)**&#x20;
  * **Full GC 처리 시간이 빠르다. (1초 내외)**&#x20;
  * **Full GC 주기가 빈번하지 않다. (10분에 1회)**&#x20;

### 3. GC 알고리즘 방식 지정&#x20;

* 위의 모니터링 결과를 보고, GC 튜닝을 진행하기로 결정했다면 GC 알고리즘 방식을 결정한다.&#x20;
* 이때 서버가 여러대라면 서버에  GC 옵션들을 서로 다르게 적용하여, 현재 내 어플리케이션의 GC 알고리즘에 따른 차이를 확인하는 것이 좋다.&#x20;

### 4. 힙 메모리 크기 지정&#x20;

* JVM 의 힙 메모리는 크기에 따라서 GC 발생 횟수와 수행 시간에 영향을 끼치기 떄문에, 옵션을 통해서 조절하면 어플리케이션 성능 향상 효과를 가져올 수 있다.&#x20;
* 여기서 말하는 메모리 크기는 JVM 의 시작 크기(-Xms) 와 최대 크기(-Xmx) 를 말한다.&#x20;
* 메모리 크기와 GC 발생 횟수, GC 수행 시간의 관계는 다음과 같다.&#x20;
  * 메모리 크기가 크면&#x20;
    * GC 발생 횟수는 감소한다.&#x20;
    * GC 수행 시간은 길어진다.&#x20;
  * 메모리 크기가 작으면
    * GC 발생 횟수는 증가한다.&#x20;
    * GC 수행 시간은 감소한다.&#x20;

| 구분        | 옵션               | 설명                       |
| --------- | ---------------- | ------------------------ |
| 힙 영역 크기   | -Xms             | JVM 시작 시 힙 영역 크기         |
| 힙 영역 크기   | -Xmx             | 최대 힙 영역 크기               |
| New 영역 크기 | -XX:NewRatio     | New 영역과 Old 영역의 비율       |
| New 영역 크기 | -XX:NewSize      | New 영역의 크기               |
| New 영역 크기 | -XX:SurviorRatio | Eden 영역과 Survivor 영역의 비율 |

<figure><img src="../../../../../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

> 이 중에서 중요한 옵션은 -Xms, -Xmx, -XX:NewRatio 옵션이다.&#x20;
>
> 특히 -Xms, -Xmx 옵션은 왠만해서는 필수로 지정하길 권장되며, 그리고 NewRatio 옵션을 어떻게 설정하느냐에 따라서 GC 성능에 많은 차이가 발생한다.&#x20;
>
> NewRatio 는 New 영역과 Old 영역의 비율이다.&#x20;
>
> \-XX:+NewRatio=1 로 지정하면 (New 영역) : (Old 영역) 의 비율은 1:1이 된다.&#x20;
>
> 만약 1GC 라면  (New 영역) : (Old 영역)  = 500MB : 500MB 가 된다.&#x20;
>
> \-XX:+NewRatio=2 로 지정하면 (New 영역) : (Old 영역) 의 비율은 1:2이 된다.&#x20;
>
> 즉, 값이 커지면 커질수록 Old 영역의 크기가 커지고, New 영역의 크기가 작아진다.&#x20;

```bash
# 힙 시작 크기 256mb, 힙 최대 크기 2gb
# young 영역과 old 영역 비율 1:2 로 설정 (New 영역:Old 영역 = 1:2)
# Parallel GC 로 실행
java -Xms256m -Xmx2048m -XX:+NewRatio=2 -XX:+UseParallelGC
```

### 5. 튜닝 결과 분석&#x20;

* GC 옵션을 지정하고 24시간 이상 데이터를 수집한다.&#x20;
* 그리고 로그를 분석해 메모리가 어떻게 할당되는지 확인한다.&#x20;
* 그 다음에 GC 방식 / 메모리 크기를 변경해 가면서 최적의 옵션을 찾아가면 된다.&#x20;

<figure><img src="../../../../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

* 분석할 떄는 다음의 사항을 중심으로 살펴보는 것이 좋다. 이는 우선 순위 별로 나열되어 있다.&#x20;
  * Full GC 수행 시간
  * Minor GC 수행 시간&#x20;
  * Full GC 수행 간격
  * Minor GC 수행 간격&#x20;
  * 전체 Full GC 수행 시간
  * 전체 Minor GC 수행 시간&#x20;
  * 전체 GC 수행 시간&#x20;
  * Full GC 수행 횟수&#x20;
  * Minor GC 수행 횟수&#x20;

### 6. 전체 서버에 반영 및 종료

* 튜닝 결과가 만족스럽다면 전체 서버에 GC 옵션을 적용하고 마무리&#x20;