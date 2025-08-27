# HashMap 이해

#### 참고 링크&#x20;

* [https://strong-park.tistory.com/entry/HashMap%EC%9D%B4-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94-%EC%9B%90%EB%A6%AC%EB%A5%BC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-putVal%EB%A9%94%EC%86%8C%EB%93%9C](https://strong-park.tistory.com/entry/HashMap%EC%9D%B4-%EB%8F%99%EC%9E%91%ED%95%98%EB%8A%94-%EC%9B%90%EB%A6%AC%EB%A5%BC-%ED%8C%8C%ED%97%A4%EC%B3%90%EB%B3%B4%EC%9E%90-putVal%EB%A9%94%EC%86%8C%EB%93%9C)
* [https://mangkyu.tistory.com/430](https://mangkyu.tistory.com/430)

## 1. HashSet 초기화&#x20;

`HashMap` 은 배열로 되어 있어서 CRUD 작업에 O(1) 이 걸린다고 알고 있다.&#x20;

`HashMap` 의 기본 생성자를 보면 다음과 같이 초기용량(`capacity` = 16) 과 로드팩터(0.75) 를 설정한다는 것을 알 수 있다.&#x20;

* 로드 팩터는 용량 대비 실제 데이터 저장 사이즈의 비율(`size/capacity)` 을 의미하며,&#x20;
* 해당 비율이 지정된 비율을 넘어서면 `capacity` 를 2배씩 증가한다. &#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-25 23.59.13.png" alt=""><figcaption></figcaption></figure>

`HashMap` 은 내부적으로 버킷의 배열로 이루어져 있다.&#x20;

* 실제로는 `Node` 라는 클래스 배열로 선언되어 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.00.51.png" alt=""><figcaption></figcaption></figure>

`Node` 는 `HashMap` 의 내부클래스로 아래처럼 저장되어 있다.&#x20;

* `hash` : 키 값의 해시코드
* `key` : 버킷에 저장된 키&#x20;
* `value` : 키에 매핑된 값&#x20;
* `next` : 해시 충돌과 연관된 필드&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.05.17.png" alt=""><figcaption></figcaption></figure>

## 2. HashMap 의 초기화와 버킷 생성

`put()` 메서드는 내부적으로 `hash()` 메서드를 호출한 뒤, `putVal()` 메서드를 호출한다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.07.21.png" alt=""><figcaption></figcaption></figure>

### hash() : 해시 생성

`hash()` 메서드를 살펴보면 키 객체의 `hashCode()` 메서드를 호출한 후 비트 연산자를 통해 **새로운 해시 코드를 생성한다.**&#x20;

* 키 객체에서 오버라이딩한 `hashCode()` 그대로 사용하는 것이 아니다!&#x20;
* 그 이유는 해시 코드의 분포를 개선하고, 해시 충돌을 최소화하고자 함이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 00.08.12.png" alt=""><figcaption></figcaption></figure>

### putVal() : 데이터 저장 메서드 분석

`putVal()` 메서드는 크게 3가지 경우의 조건문으로 나뉜다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 19.03.02.png" alt=""><figcaption></figcaption></figure>

#### 1. if ((tab = table) == null || (n = tab.length) == 0)

* 현재 `table` 이 할당되지 않았거나, 길이가 0이라면 `resize()` 를 호출해서 새로운 `table` 을 할당&#x20;
* 처음 데이터를 넣을 때 테이블을 생성하기 위해서 실행 \
  (초기화 시점에 `table` 을 할당하지 않는것을 알 수 있다)

#### 2. if ((p = tab\[i = (n - 1) & hash]) == null)

* **비트 연산을 사용해 나머지 연산처럼 구현하고(`(n-1) & hash`),** \
  **()**
* 해당 인덱스에 버킷의 데이터가 없다면 새로운 `Node` 를 할당&#x20;
* 현재 버킷에 처음 들어가는 데이터일 경우 실행&#x20;

#### 3. else&#x20;

* value 를 대체하거나, 해시충돌이 발생할 때 실행&#x20;

### resize() :  데이터 추가를 위한 준비

처음 데이터를 넣었을 때는 `table` 이 할당되지 않은 상태이기 때문에,\
`resize()` 를 통해 `tab` 변수를 초기화 해주어야 한다.&#x20;

그러면 `resize()` 메서드 호출이 어떻게 되는지 살펴보자.

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 19.06.09.png" alt=""><figcaption></figcaption></figure>

#### 1. 현재 사용중인 table 의 용량과 임계값을 old 변수에 할당하고, new 변수의 값을 0으로 초기화해준다.&#x20;

```java
Node<K,V>[] oldTab = table;
int oldCap = (oldTab == null) ? 0 : oldTab.length;
int oldThr = threshold;
int newCap, newThr = 0;
```

#### 2. new 변수에 값을 초기값으로 할당해준다.&#x20;

```java
else {    // zero initial threshold signifies using defaults
    newCap = DEFAULT_INITIAL_CAPACITY;
    newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
}
```

* 기본 용량 상수 초기화 부분을 살펴보면 그냥 16으로 초기화하는 것이 아니라, 비트 연산을 사용하는 것을 볼 수 있다.&#x20;
* 이렇게 표현하는 의도는 **`HashMap` 의 용량이 2의 거듭제곱이어야 한다는 중요한 설계 규칙을 시각적으로 제공한다.**

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 19.10.05.png" alt=""><figcaption></figcaption></figure>

* 기본 로드펙터 상수는 0.75로 초기화 되어 있는 것을 확인할 수 있다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-26 19.16.09.png" alt=""><figcaption></figcaption></figure>

#### 3. 임계값을 새로 계산된 값으로 할당하고, `newCap` 의 용량만큼 새로운 테이블을 생성한다.&#x20;

```java
threshold = newThr;
@SuppressWarnings({"rawtypes","unchecked"})
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
table = newTab;
```

* `threshold` : `resize()` 가 필요한 시점의 임계값 (용량 \* 로드펙터)&#x20;
* 임계값은 `HashMap` 의 용량에 따라 동적으로 조절되고, 해시 충돌 등의 상황에서 동적으로 조절된다.&#x20;

**가장 처음 `resize()` 를 통해서 버킷이 생성된다고 생각하면 된다.**

## 3. HashMap 에 데이터 저장하기

### 데이터 저장하기1 (첫번째 데이터)

`resize()` 후 아직까지는 버킷을 생성했을 뿐, 데이터를 버킷에 넣지는 않았다.&#x20;

그렇다면 이제 데이터가 저장되는 과정을 살펴보자.&#x20;

가장 처음 임의의 데이터를 넣어보자.

```java
// map.put 이라는 행위에 집중하자. 
map.put(new Person("gugbab1", 29), "person1"); 
```

위 코드를 실행하게 되면 `putVal()` 메서드의 2번째 조건문에 들어가게 된다.&#x20;

* `table` 변수는 `resize()` 메서드를 통해 크기가 16인 `Node` 배열로 초기화되었다는 것을 기억하자.&#x20;
* 인덱스 `i` 는 비트 연산(`&`) 을 통해 나머지 연산과 동일한 결과를 갖는다.&#x20;

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

가장 처음 데이터를 넣게 되면 들어있는 원소가 없으므로, 버킷에 노드를 삽입하는 코드가 실행된다.&#x20;

### 데이터 저장하기2 (기존과 다른 데이터)

그러면 두 번째 데이터를 넣어면 어떻게 되는지도 살펴보자.&#x20;

<pre class="language-java"><code class="lang-java"><strong>// map.put 이라는 행위에 집중하자. 
</strong>map.put(new Person("gugbab2", 30), "person2"); 
</code></pre>

위 코드를 실행하게 되면 해시 충돌이 발생하지 않는 경우 `putVal()` 메서드의 2번째 조건문에 들어가게 된다.&#x20;

* 해당 케이스에서는 해시 충돌이 발생하지 않았다고 가정하자.&#x20;

```java
if ((p = tab[i = (n - 1) & hash]) == null)
    tab[i] = newNode(hash, key, value, null);
```

해시 충돌이 발생하지 않은 경우, 해당 인덱스에 가장 처음 들어가는 데이터이기 때문에, 버킷에 노드를 삽입하는 코드가 실행된다.&#x20;

### 데이터 저장하기3 (동등한 세번째 데이터, value 대체)&#x20;

그렇다면 동등한 Key 값을 가진 데이터를 저장할 땐 어떻게 동작하는지 살펴보자.&#x20;

```java
// map.put 이라는 행위에 집중하자. 
map.put(new Person("gugbab1", 29), "person3"); 
```

`putVal()` 메서드의 첫번째, 두번째 조건에 걸리지 않기 때문에 `else` 부분으로 오게되고, \
`else` 부분은 3가지 부분으로 나뉜다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-27 11.01.09.png" alt=""><figcaption></figcaption></figure>

#### 1. 기존 노드와 해시 값이 같고, 키 값이 동등한 경우 e 에 기존 노드를 할당한다.&#x20;

* 동등한 객체일 경우, 값(`value`) 대체&#x20;

#### 2. TreeNode 클래스인 경우 트리 기반의  삽입 로직을 수행&#x20;

* 해시 충돌이 발생했는데, `p` 가 트리노드 기반일 경우&#x20;

#### 3. 연결 리스트인 경우, 반복문을 통해서 노드를 탐색하면서 새로운 노드를 추가

* 해시 충돌이 발생했는데, `p` 가 트리노드 기반이 아닐 경우

해당 케이스의 경우 첫번째 조건을 타게 되고, \
`e` 값을 기존 노드 값으로 초기화 후 `e` 가 `null` 이 아닐 경우 값을 대체해주는 과정을 거친다.&#x20;

```java
if (p.hash == hash && 
    ((k = p.key) == key || (key != null && key.equals(k))))
    e = p;
```

**동등한 키에 대해서 새로 들어온 값으로 덮어쓴다.**&#x20;

* `map.put()` 메서드에 같은 키를 사용해 값을 넣었을 경우, 반환 값으로 기존 값이 리턴되는 이유가 \
  `oldValue` 가 리턴되는 부분 때문이다.&#x20;

```java
if (e != null) { // existing mapping for key
    V oldValue = e.value;
    if (!onlyIfAbsent || oldValue == null)
        e.value = value;
    afterNodeAccess(e);
    return oldValue;
}
```

### 데이터 저장하기4 (해시충돌)&#x20;

동등한 객체를 집어넣을 때는 값(`value`) 을 덮어쓰는 것을 확인했다. \
그러면 해시가 동일하고 내용은 다른 상황에서 어떻게 동작할까?&#x20;

결론적으로 각 `Node` 는 기본적으로 연결리스트로 저장한 뒤, \
연결리스트의 길이가 일정 임계값을 넘으면 해당 버킷의 노드들을 `Tree` 구조로 변환한다.&#x20;

일반적으로 `HashMap` 은 내부적으로 배열(`Node` 의 배열)로 되어 있어서 CRUD 작업에 O(1) 이 걸린다고 알고 있다.&#x20;

하지만 해시가 충돌하는 경우 배열의 인덱스마다 새로운 자료구조를 사용해 데이터를 저장하게 되고 자료구조에 따라 다음과 같은 성능 차이가 난다.&#x20;

* 연결 리스트 : O(N)
* Tree : O(log N)

그러면 해시값이 동일한 네번째 데이터를 넣어면 어떻게 되는지도 살펴보자.&#x20;

<pre class="language-java"><code class="lang-java"><strong>// 해시 값이 동일하다고 가정! 
</strong><strong>// 여기서 말하는 해시값은 객체 내부 hashCode() 메서드가 아니라, 
</strong><strong>// HashMap.hash() 메서드에서 나온 값을 의미
</strong>map.put(new Person("gugbab4", 100), "person4"); 
</code></pre>

일단 해시 충돌이 난 상황이고, 트리노드 기반이 아니기 때문에 3번째 `else` 문부터 시작할 것이다.&#x20;

`else` 문을 순서대로 살펴보자.&#x20;

```java
else {
    for (int binCount = 0; ; ++binCount) {
        if ((e = p.next) == null) {
            p.next = newNode(hash, key, value, null);
            if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                treeifyBin(tab, hash);
            break;
        }
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            break;
        p = e;
    }
}
```

`binCount` 는 연결리스트의 노드 개수를 의미한다. 0부터 시작해서 연결 리스트를 순회하는 반복문으로서 사용된다.&#x20;

```java
for (int binCount = 0; ; ++binCount) {
```

현재 노드 `p.next` 를 `e` 에 할당한 뒤, `null` 인 상황이므로 `next` 에 새로운 노드를 생성한다.&#x20;

```java
if ((e = p.next) == null) {
    p.next = newNode(hash, key, value, null);
    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
        treeifyBin(tab, hash);
    break;
}
```

하지만, 해시충돌이 계속되고 연결리스트의 길이가 임계값(`TREEIFY_THRESHOLD - 1`)에 다다를 경우, 버킷의 노드들을 트리노드로 변환한다.&#x20;

* 임계값이 8을 넘으면 `treeifyBin()` 메서드가 실행되면서 기존의 노드들을 트리노드로 변환한다.&#x20;
* 원래 Java7 까지는 노드를 연결리스트로만 저장해서 시간복잡도가 O(N) 으로 고정이었다.&#x20;
* 하지만 Java8 부터는 임계값을 넘어설 경우 트리노드로 변환되며 시간복잡도가 O(logN) 으로 변경되었다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-27 11.38.07.png" alt=""><figcaption></figcaption></figure>

하지만, `treeifyBin()` 메서드가 실행된다고 해서 무조건 연결리스트가 트리노드 구조로 변경되는 것은 아니다.

`HashMap` 전체 크기가 64보다 적은 경우는 먼저 해시 버킷 배열을 확장한다.&#x20;

* 왜냐하면 해시 버킷 배열 크기 자체가 작은 경우에는 충돌이 자주 발생할 수밖에 없기 때문이다.&#x20;

<figure><img src="../../../.gitbook/assets/스크린샷 2025-08-27 11.48.08.png" alt=""><figcaption></figcaption></figure>

Java 는 이러한 방향으로 `HashMap` 내부를 구현하여 최적화를 진행하고 있으며, 버킷 충돌이 자주 발생하는 경우 트리화를 통해 시간 복잡도를 O(log N) 까지 낮출 수 있었다.&#x20;

**하지만 이 때 키가 `String`, `Number` 클래스 같은 `Comparable` 타입이어야만 트리로 변환이 가능하다는 점은 주의할 필요가 있다!**
