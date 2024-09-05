# 연산자

## 기본 연산자&#x20;

* `=`, `+`, `-`, `*`, `/`, `%` : 순서대로 대입, 더하기, 빼기, 곱하기, 나누기, 나머지 연산자이다.&#x20;

## 간단한 연산을 수행하는 대입 연산자들&#x20;

* `+=` : 기존 값에 우측 항의 값을 더함
* `-=` : 기존 값에 우측 항의 값을 뺌
* `*=` : 기존 값에 우측 항의 값을 곱함
* `/=` : 기존 값을 우측 항의 값으로 나눔&#x20;
* `%=` : 기존 값을 우측 항의 값으로 나눈 나머지&#x20;

## 단항 연산자&#x20;

* `+` : 단항 플러스 연산자
* `-` : 단항 마이너스 연산자
* `++` : 증가 연산자&#x20;
  * `++` 연산자는 변수 앞, 뒤에 붙이는 가에 따라서 연산되는 시점이 달라진다.&#x20;
* `--` : 감소 연산자&#x20;
  * `--` 연산자는 변수 앞, 뒤에 붙이는 가에 따라서 연산되는 시점이 달라진다.&#x20;
* `!` :  논리 부정 연산자

## 비교 연산자&#x20;

#### 동치 비교 연산자&#x20;

* 특징&#x20;
  * 동치 비교 연산자는 모든 타입에서 사용이 가능하다. (주의점은 같은 카테고리의 타입끼리 사용해야 한다)
  * **`참조 자료형`의 경우 그 레퍼런스 값이 같은지 다른지를 비교한다.**
* 종류&#x20;
  * `==` : 같음&#x20;
  * `!=` : 같지 않음&#x20;

#### 대소 비교 연산자&#x20;

* 특징&#x20;
  * **대소 비교 연산자는 `boolean`, `참조 자료형`에서는 사용할 수 없다.**&#x20;
    * Java 는 C++ 와 같이 연산자 오버로딩을 지원하지 않기 때문에, 대소 비교 연산자를 참조 자료형에서 사용할 수 없다.&#x20;
* 종류&#x20;
  * `>` : 왼쪽 값이 큼&#x20;
  * `>=` : 왼쪽 값과 같거나 큼&#x20;
  * `<` : 왼쪽 값이 작음&#x20;
  * `<=` : 왼쪽 값과 같거나 큼&#x20;

## 논리 연산자&#x20;

* `&&` : AND 결합&#x20;
* `||` : OR 결합&#x20;
  * Java에서 `||` 연산자는 \*\*단락 평가 (short-circuit evaluation)\*\*를 사용합니다. 이는 **논리 OR 연산자**의 특징으로, 앞 항의 조건이 `true`일 경우 뒤 항의 조건은 평가되지 않는다는 의미입니다.

## 기본자료형의 형 변환&#x20;

* 기본자료형, 참조자료형 모두 괄호로 묶어 주어서 형 변환을 할 수 있다.&#x20;
* **기본 자료형 중에서 형 변환이 전혀 되지 않는 것이 있는데, 바로 boolean 타입이다.**&#x20;
* **그리고, 기본자료형 -> 참조자료형 or 참조자료형 -> 기본자료형으로의 형 변환은 절대 안된다.**&#x20;
* 형 변환 시 범위가 더 큰 타입으로 변환하면 아무런 문제가 없다.. 하지만 범위가 작은 타입으로 변환하게 되면, 쓰레기 값이 나올 수 있으니 형 변환 시 타입의 범위를 생각해보자

## 비트연산자&#x20;

* 비트 연산자는 비트 단위로 데이터를 조작하는 데 사용된다. 주로 정수형 데이터에 대해 사용된다.&#x20;

### 1.   AND(&)

* 두 비트가 모두 1일 때 결과가 1이 된다.&#x20;

<pre class="language-java"><code class="lang-java"><strong>int a = 5;    // 0101
</strong>int b = 3;    // 0011 
int result = a &#x26; b;    // 0001(1) 
</code></pre>

### 2. OR(|)

* 두 비트.중 하나라도 1이면 결과가 1이 된다.&#x20;

```java
int a = 5;    // 0101
int b = 3;    // 0011 
int result = a | b;    // 0111(7)
```

### 3. XOR(^)

* 두 비트가 서로 다를 때 결과가 1이 된다.&#x20;

```java
int a = 5;  // 0101
int b = 3;  // 0011
int result = a ^ b; // 0110 (6)]
```

### 4. NOT(\~)

* 비트를 반전시킨다. 1은 0으로, 0은 1로 바뀐다.&#x20;

```java
int a = 5;   // 0101
int result = ~a; // 1010 (-6)
```

### 5. 비트 왼쪽 시프트(<<)&#x20;

* 비트를 왼쪽으로 지정한 만큼 이동시킨다. 오른쪽 빈자리는 0으로 채워진다.&#x20;

```java
int a = 5;  // 0101
int result = a << 1; // 1010 (10)
```

### 6. 비트 오른쪽 시프트(>>)&#x20;

* 비트를 오른쪽으로 지정한 만큼 이동시킨다. 왼뽁 빈자리는 부호 비트로 채워진다. (음수일 경우 1, 양수일 경우 0)&#x20;

```java
int a = 5;    // 0101
int result = a >> 1;    // 0010(2) 
```

### 7. 비트 부호 없는 오른쪽 시프트(>>>)&#x20;

* 비트를 오른쪽으로 지정한 만큼 이동시키며, 왼쪽 빈 자리는 항상 0으로 채워진다. (부호 비트에 상관 없이 0으로 채운다)&#x20;

```java
int a = -5;  // 11111111111111111111111111111011
int result = a >>> 1; // 01111111111111111111111111111101 (2147483645)
```