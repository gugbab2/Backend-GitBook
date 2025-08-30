# CuncurrentHashMap 이해

#### 참고 링크&#x20;

* [https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/concurrent/ConcurrentHashMap.html)
* [https://curiousjinan.tistory.com/entry/java-concurrent-hash-map-cas](https://curiousjinan.tistory.com/entry/java-concurrent-hash-map-cas)
* [https://devlog-wjdrbs96.tistory.com/269](https://devlog-wjdrbs96.tistory.com/269)

## 1. ConcurrentHashMap 의 소개

#### 탄생 배경&#x20;

`ConcurrentHashMap` 은 자바 1.5에서 도입되었으며, 주로 멀티스레드 환경에서의 성능 문제를 해결하기 위해서 설계되었다. 기존의 `HashMap`, `Hashtable` 은 멀티스레드 환경에서 데이터 무결성은 보장하지 못하거나, 성능이 크기 저하되는 문제가 있었다. 특히 `Hashtable` 은 모든 메서드에 대해 동기화를 사용하여 안정성을 확보했지만, 그로 인해 성능이 크게 떨어졌다.&#x20;

이를 해결하기 위해서 `ConcurrentHashMap` 이 도입되었으며, 부분적인 락(lock) 을 사용하여 성능을 향상시키면서도 Thread Safety 를 보장할 수 있었다.&#x20;

#### 사용 방법(실시간 데이터 처리 시스템)&#x20;

* 실시간 데이터 처리 시스템에는 여러 스레드가 데이터 스트림을 소비하고, 각 데이터의 처리 상태를 추적해야 할 수 있다. `ConcurrentHashMap` 을 사용하여 각 데이터 ID 의 처리 상태를 안전하게 관리할 수 있다.&#x20;
* 상태 업데이트 : 데이터가 처리될 때마다 `updateStatus` 메서드를 사용하여 현재 상태를 업데이트 한다. `put` 메서드는 Thread Safety 하게 상태를 변경할 수 있도록 보장한다.&#x20;
* 상태 조회 : `getStatus` 메서드를 사용하여 데이터의 처리 상태를 실시간으로 조회한다. 만약 해당 데이터가 존재하지 않는 경우, 기본 상태 "UNKNOWN" 을 반환한다.&#x20;

```java
import java.util.concurrent.ConcurrentHashMap;

public class RealTimeDataProcessor {

    private final ConcurrentHashMap<String, String> dataStatus = new ConcurrentHashMap<>();

    // 데이터 처리 상태 업데이트
    public void updateStatus(String dataId, String status) {
        dataStatus.put(dataId, status);
    }

    // 데이터 처리 상태 조회
    public String getStatus(String dataId) {
        return dataStatus.getOrDefault(dataId, "UNKNOWN");
    }

    public static void main(String[] args) {
    
        RealTimeDataProcessor processor = new RealTimeDataProcessor();

        // 스레드 A가 데이터 처리 시작
        new Thread(() -> {
            processor.updateStatus("data1", "PROCESSING");
            // 데이터 처리 로직...
            processor.updateStatus("data1", "COMPLETED");
        }).start();

        // 스레드 B가 데이터 상태를 실시간으로 조회
        new Thread(() -> {
            System.out.println("Data1 처리 상태: " + processor.getStatus("data1"));
        }).start();
    }   
}
```

## 2. ConcurrentHashMap 클래스와 putVal 메서드 살펴보기&#x20;

#### 핵심 코드인 `putVal` 메서드를 살펴보자.&#x20;

`ConcurrentHashMap` 의 핵심 기능은 동시성을 안전하게 관리하는 것이다. `ConcurrentHashMap` 에서는 읽기 작업은 별다른 동기화 없이 빠르게 수행되도록 설계되어 있어 문제가 없다. 하지만 쓰기 작업에서는 여러 스레드가 동시에 같은 키에 접근하거나 데이터를 수정하려고 할 때 동시성 문제가 발생할 수 있다.&#x20;

이 때문에 쓰기 작업을 처리하는 putVal 메서드가 굉장히 중요한 역할을 한다. 이 메서드는 여러 스레드가 같은 키에 쓰기를 시도할 때 동시성 제어를 통해 안전하게 데이터를 삽입하거나 업데이트 할 수 있도록 설계되었다. 이 메서드는 락(lock), CAS(Compare-And-Swap) 연산, 노드 동기화 등의 기법을 사용하여 멀티 스레드 환경에서도 데이터의 일관성과 안전성을 보장한다.&#x20;

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // 키와 값이 null일 수 없으므로 NullPointerException을 던집니다.
    if (key == null || value == null) throw new NullPointerException();
    
    // 키의 해시코드를 계산하고 이를 스프레드(spread = 비트연산)하여 해시 테이블에서의 위치를 분산시킵니다.
    int hash = spread(key.hashCode());
    int binCount = 0;

    // 해시 테이블이 초기화되었는지, 그리고 각 버킷을 처리할 수 있는지 확인합니다.
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh; K fk; V fv;
        // 해시 테이블이 아직 초기화되지 않았거나 크기가 0이면 초기화합니다.
        if (tab == null || (n = tab.length) == 0)
            tab = initTable();
        // 계산된 인덱스의 버킷이 비어 있으면 CAS(Compare-And-Swap)로 새로운 노드를 추가합니다.
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
                break;  // 락 없이 빈 버킷에 노드를 추가한 경우 반복을 종료합니다.
        }
        // 버킷이 리사이징 중인 경우 해당 버킷을 리사이징 작업에 참여시킵니다.
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        // 락 없이 첫 번째 노드를 확인하여 조건부로 노드를 삽입할 수 있는지 확인합니다.
        else if (onlyIfAbsent 
                 && fh == hash
                 && ((fk = f.key) == key || (fk != null && key.equals(fk)))
                 && (fv = f.val) != null)
            return fv; // 조건에 맞는 기존 값을 반환합니다.
        else {
            V oldVal = null;
            // 첫 번째 노드가 비어있지 않다면, 노드를 수정하기 위해 해당 버킷을 동기화합니다.
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        // 연결 리스트를 순회하여 키가 존재하는지 확인하고, 값을 갱신하거나 새 노드를 추가합니다.
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value; // 기존 값이 존재하면 덮어씁니다.
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key, value);
                                break; // 새 노드를 연결 리스트에 추가합니다.
                            }
                        }
                    }
                    // 노드가 트리 구조인 경우 트리 노드에 대해 처리합니다.
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value; // 트리 노드의 값을 갱신합니다.
                        }
                    }
                    // 예약된 노드에 대한 재귀적 업데이트 오류를 방지합니다.
                    else if (f instanceof ReservationNode)
                        throw new IllegalStateException("Recursive update");
                }
            }
            if (binCount != 0) {
                // 노드 수가 특정 임계치(TREEIFY_THRESHOLD)를 초과하면 연결 리스트를 트리 구조로 변환합니다.
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal; // 기존 값을 반환합니다.
                break;
            }
        }
    }
    // 전체 노드 수를 증가시키고, 필요 시 테이블을 리사이징합니다.
    addCount(1L, binCount);
    return null; // 새로운 값을 추가한 경우 null을 반환합니다.
}
```

### CAS(Compare-And-Swap) 연산이란?&#x20;

CAS 는 컴퓨터 과학에서 비동기적 동시성 제어 기법 중 하나로, 여러 스레드가 동시에 데이터를 수정하려고 할 때 데이터의 일관성을 보장하는 방법이다. CAS 는 원자정(atomic) 연산으로, 멀티스레드 환경에서 안전하게 데이터를 읽고 쓸 수 있도록 한다.&#x20;

#### CAS 의 작동 원리 : 세 가지 값을 사용하여 연산 수행&#x20;

연산에 사용되는 3가지 값

* 메모리 주소 : 변경하고자 하는 데이터의 위치를 가리킨다.&#x20;
* 기존 예상 값(Old Value) : 현재 메모리 주소에 저장된 값으로 예상되는 값이다.&#x20;
* 새로운 값(New Value) : 메모리 주소에 저장하고자 하는 값이다.&#x20;

연산 순서&#x20;

1. CAS 연산은 먼저 특정 메모리 주소의 현재 값을 읽는다. 이 값을 기대값(expected value) 라고 한다.&#x20;
2. 읽어들인 현재 값이 스레드가 기대한 값과 동일한지 비교한다.&#x20;
   1. 만약 현재 값이 기대값과 일치한다면, 메모리 위치의 값을 새로운 값으로 원자적(atomic) 으로 교체한다.&#x20;
   2. 만약 현재 값이 기대값과 일치하지 않으면, 다른 스레드가 그 사이에 값을 변경했다는 의미이기 때문에 교체를 실패하고, 다시 시도하거나 다른 적절한 처리를 한다.&#x20;

#### CAS 연산은 하드웨어 수준에서 원자적으로 수행된다.&#x20;

원자적 연산의 정의&#x20;

* 원자적 연산이란 완전히 실행되거나 전혀 실행되지 않는 연산을 말한다. 즉, 연산이 시작되면 중간에 다른 연산이 개입할 수 없고, 연산 도중에 중단되지 않고 완료된다.&#x20;

하드웨어 수준에서 CAS 연산이 원자적인 이유&#x20;

* 대부분의 현대 CPU 는 CAS 연산을 지원하는 특수한 명령어를 가지고 있다.&#x20;
* 해당 명령어는 메모리의 특정 주소에서 값을 읽고, 기대값과 비교한 후, 두 값이 일치하면 새 값으로 교체하는 과정을 단일 명령어로 처리한다.&#x20;
* **이 과정은 단일 명령어로 처리되기 때문에, CAS 연산을 실행하는 동안 다른 어떤 연산도 해당 메모리 주소에 접근할 수 없다. 이는 하드웨어적으로 보장된 원자성을 의미한다.**&#x20;

CAS 연산 중 다른 스레드가 값을 바꾸지 못하는 이유&#x20;

* CAS 명령어가 실행되는 동안, 해당 메로리 주소에 대한 독점적 접근권을 가진다. **즉, CAS 명령어가 실행되는 동안에는 다른 스레드나 프로세스가 해당 메모리 주소를 읽거나 쓸 수 없다. (하드웨어 단에서 제공하는 제약!)**
* ~~이를 수행하기 위한 내부적인 기술이 있지만, 그 내용은 패스..~~

#### 왜 CAS 가 중요한가?&#x20;

1. 락이 필요 없다.&#x20;
   1. CAS 는 전통적인 락(lock) 매커니즘을 사용하지 않고도 동시성을 제어할 수 있다. 락을 사용하지 않기 때문에, 데드락(교착 상태) 이 발생하지 않고, 락을 획득하고 해제하는 비용을 줄일 수 있다.&#x20;
2. 높은 성능&#x20;
   1. CAS 는 락보다 성능이 뛰어난 경우가 많다. 특히, 데이터 충돌이 적은 환경에서는 락을 사용하는 것보다 더 빠르게 동작한다.&#x20;
   2. ~~반면에 데이터 충돌이 높은상황에서는 단점으로 작용할 수 있다..~~

#### CAS 한계점&#x20;

1. 스핀 락&#x20;
   1. 데이터 충돌이 많은 상황에서는 스레드들이 계속해서 재시도하면서 CPU 자원을 낭비하는 스핀락 현상이 발생한다.&#x20;
   2. 때문에, 데이터 충돌이 많을 것으로 판단되는 환경이라면 CAS 를 사용하지 않는 것이 좋다.&#x20;

### ConcurrentHashMap 에서는 어떻게 CAS 를 사용할까?&#x20;

```java
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    // 여기서 사용된다.
    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
        break; // 락 없이 빈 버킷에 노드를 추가한 경우 반복을 종료합니다.
}
```

#### `ConcurrentHashMap` 의 `putVal` 메서드 내부의 CAS 동작 분석&#x20;

1. `tabAt(tab, i)` 로 계산된 인덱스 `i` 에 해당하는 버킷이 `null` 인지를 확인한다.&#x20;
   1. 여기서도 `HashMap.putVal()` 과 마찬가지로 비트 연산을 통해서 인덱스를 구한다. \
      ~~(자세한 설명 패스 .. )~~
2. `tabAt(tab, i)` 의 결과가 `null` 이면 해당 인덱스에 노드가 없음을 의미한다.&#x20;
   1. 버킷이 비어있는 경우 : `tabAt(tab, i)` 의 결과가 `null` 이라면, 이 인덱스 위치에 노드가 없다는 것을 의미한다.&#x20;
   2. 이 경우 `casTabAt` 메서드가 실행된다.&#x20;
   3. `casTabAt` 은 CAS 연산을 사용하여 해당 위치가 여전히 `null` 일 때만 새로운 노드를 원자적으로 삽입하려고 시도한다.&#x20;
3. `casTabAt` 메서드를 사용하면 내부적으로 네이티브 메서드가 실행된다.&#x20;
   1. 현재 tab\[i] 의 값이 여전히 null 인지 확인한다. (즉, 다른 스레드가 아직 해당 위치에 노드를 삽입하지 않았는지 확인)
   2. 값이 `null` 이면, 새로운 노드로 원자적으로 대체한다.&#x20;
   3. 성공하면 `true` 를 반환하고, 그렇지 않으면(다른 스레드가 그 사이에 노드를 삽입한 경우) `false` 를 반환한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-30 13.51.45.png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-30 13.50.51.png" alt=""><figcaption></figcaption></figure>

#### ConcurrentHashMap 에서 putVal 메서드에서 CAS 를 사용하는 이유&#x20;

여기서 CAS 를 사용하는 이유는 명시적 락(lock) 을 사용하지 않고도 스레드 안정성(Thread Safety) 을 보장하기 위해서다. 여러 스레드가 동시에 같은 버킷에 노드를 삽입하려고 할 수 있지만, CAS 는 그중 오직 하나의 스레드만이 빈 버킷에 노르를 성공적으로 삽입할 수 있도록 보장한다.&#x20;

## 3. ConcurrentHashMap 의 putVal 메서드 분석&#x20;

#### 1. 입력 값 검증 및 해시 계산

~~패스 ..~~&#x20;

#### 2. 무한 루프와 테이블 초기화&#x20;

~~패스 ..~~&#x20;

#### 3. 비어있는 버킷에 노드 추가(CAS 연산)&#x20;

```java
// 계산된 인덱스의 버킷이 비어 있으면 CAS(Compare-And-Swap)로 새로운 노드를 추가합니다.
else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
    if (casTabAt(tab, i, null, new Node<K,V>(hash, key, value)))
        break;  // 락 없이 빈 버킷에 노드를 추가한 경우 반복을 종료합니다.
}
```

* **버킷 검사**&#x20;
  * 계산된 인덱스 `i` 에 있는 버킷이 비어 있는지 확인한다. 비어 있다면 `tabAt` 메서드를 통해 해당 위치의 값을 가져온다.&#x20;
* **CAS 연산을 통한 노드 추가**
  * 버킷이 비어 있으면 `casTabAt` 메서드를 사용해 원자적으로 새로운 노드를 추가한다. CAS 는 멀티스레드 환경(동시성 환경) 에서 락을 사용하지 않고도 안전하게 값을 업데이트 할 수 있는 메커니즘이다. 만약 CAS 가 성공하면, 노드가 추가되었으므로 `break` 를 통해 반복문을 종료한다.&#x20;

#### 4. 리사이징 상태 확인 및 지원

```java
// 버킷이 리사이징 중인 경우 해당 버킷을 리사이징 작업에 참여시킵니다.
else if ((fh = f.hash) == MOVED)
    tab = helpTransfer(tab, f);
```

* **리사이징 상태 확인** : 만약 `f.hash` 가 `MOVED` 값이라면, 이는 테이블이 현재 리사이징 중임을 의미한다. 리사이징은 해시 테이블이 특정 임계치에 도달하여 크기를 늘려야 할 때 발생한다.&#x20;
* ~~**리사이징 지원** : `helpTransfer` 메서드를 호출하여, 현재 스레드가 리사이징 작업을 돕도록 한다. 이는 병렬 리사이징을 통해 여러 스레드가 동시에 테이블 크기를 조정할 수 있도록 한다.~~&#x20;

#### 5. 조건부로 기존 노드 확인 및 반환&#x20;

```java
// 락 없이 첫 번째 노드를 확인하여 조건부로 노드를 삽입할 수 있는지 확인합니다.
else if (onlyIfAbsent 
        && fh == hash
        && ((fk = f.key) == key || (fk != null && key.equals(fk)))
        && (fv = f.val) != null)
   return fv; // 조건에 맞는 기존 값을 반환합니다.
```

* **조건부 확인** : `onlyIfAbsent` 가 `true` 일 경우, 즉 값이 이미 존재하는 경우(삽입하지 않으려는 경우), 첫 번째 노드를 락 없이 확인한다. 이 최적화는 성능을 높이기 위해 락 없이 작업을 완료할 수 있을 때 사용된다.&#x20;
* **기존 값 확인** : 키가 이미 존재하고, 값이 `null` 이 아닌 경우 기존 값을 반환하고 메서드를 종료한다. 이 경우 추가 작업이 필요하지 않으므로, 빠르게 종료할 수 있다.&#x20;

#### 6. 락을 사용한 노드 삽입 및 업데이트&#x20;

```java
else {
        V oldVal = null;
        // 첫 번째 노드가 비어있지 않다면, 노드를 수정하기 위해 해당 버킷을 동기화합니다.
        synchronized (f) {
            if (tabAt(tab, i) == f) {
                if (fh >= 0) {
                    binCount = 1;
                    // 연결 리스트를 순회하여 키가 존재하는지 확인하고, 값을 갱신하거나 새 노드를 추가합니다.
                    for (Node<K,V> e = f;; ++binCount) {
                        K ek;
                        if (e.hash == hash &&
                            ((ek = e.key) == key ||
                             (ek != null && key.equals(ek)))) {
                            oldVal = e.val;
                            if (!onlyIfAbsent)
                                e.val = value; // 기존 값이 존재하면 덮어씁니다.
                            break;
                        }
                        Node<K,V> pred = e;
                        if ((e = e.next) == null) {
                            pred.next = new Node<K,V>(hash, key, value);
                            break; // 새 노드를 연결 리스트에 추가합니다.
                        }
                    }
                }
                // 노드가 트리 구조인 경우 트리 노드에 대해 처리합니다.
                else if (f instanceof TreeBin) {
                    Node<K,V> p;
                    binCount = 2;
                    if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key, value)) != null) {
                        oldVal = p.val;
                        if (!onlyIfAbsent)
                            p.val = value; // 트리 노드의 값을 갱신합니다.
                    }
                }
                // 예약된 노드에 대한 재귀적 업데이트 오류를 방지합니다.
                else if (f instanceof ReservationNode)
                    throw new IllegalStateException("Recursive update");
            }
        }
        if (binCount != 0) {
            // 노드 수가 특정 임계치(TREEIFY_THRESHOLD)를 초과하면 연결 리스트를 트리 구조로 변환합니다.
            if (binCount >= TREEIFY_THRESHOLD)
                treeifyBin(tab, i);
            if (oldVal != null)
                return oldVal; // 기존 값을 반환합니다.
            break;
        }
    }
```

* **락을 사용한 동기화** : 버킷의 첫 번째 노드가 존재하면 해당 노드를 락(`synchronized`) 으로 감싸 동기화한다. 이 과정은 멀티스레드 환경에서 여러 스레드가 동시에 같은 버킷에 접근할 경우 발생하는 충돌과 데이터 손상을 방지하기 위함이다.&#x20;
* **연결 리스트 처리** : 버킷이 연결 리스트로 구성된 경우, 리스트 각 노드를 순회하면서 동일한 키를 찾고, 발견하면 값을 갱신한다. 만약 키가 존재하지 않는다면 리스트 끝에 새로운 노드를 추가한다.&#x20;
* **트리 구조 처리** : 만약 노드가 트리 구조(TreeBin) 으로 되어 있다면, 트리 노드에 대해서도 키를 찾아 값을 업데이트하거나 새 노드를 추가한다.&#x20;
* 코드를 보면 `f` 가 `ReservationNode` 이면 예외를 발생시킨다. 이것은 `ConcurrentHashMap` 에서 노드 삽입 중에 발생할 수 있는 재귀적 업데이트를 방지하기 위해서 사용된다.&#x20;
  * `ConcurrentHashMap` 은 테이블이 확장될 때 여러 스레드가 동시에 기존 노드를 새로운 테이블로 옮겨야 할 때가 있다.&#x20;
  * 이 과정에서, **스레드가 노드를 옮기면서 또 다른 노드를 추가하려고 하거나, 또는 노드가 옮겨지는 동안 다시 접근하려고 하면 재귀적 갱신** 문제가 발생할 수 있다.&#x20;
  * **`ReservationNode` 는 이런 재귀적 갱신을 방지**하기 위해서 해당 인덱스가 이미 처리 중임을 표시하는 역할을 한다.&#x20;
  * **`ReservationNode` 가 발견되면 이는 해당 인덱스에 대해 특별한 처리가 이미 진행 중**이라는 뜻이므로, 더 이상의 변경이 수행되지 않도록 예외를 던진다.&#x20;

#### 7. 트리화 검사 및 노드 수 증가&#x20;

```java
if (binCount != 0) {
    if (binCount >= TREEIFY_THRESHOLD)
        treeifyBin(tab, i);
    if (oldVal != null)
        return oldVal;
    break;
}
```

~~패스 ..~~

#### 8. 노드 수 증가 및 종료 처리 (마지막)

~~패스 ..~~

> ## putVal() 내부에서 CAS, Synchronized 연산을 구분하는 이유
>
>
>
> ### CAS&#x20;
