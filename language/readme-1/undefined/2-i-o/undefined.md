# 문자 인코딩

## 컴퓨터와 데이터&#x20;

개발자가 개발하며 다루는 데이터는 크게 0101 로 되어있는 바이너리 데이터(또는 byte 기반 데이터) 와 "ABC" 와 같은 문자로 되어 있는 텍스트 데이터를 다룬다.

그런데 텍스트 데이터가 어떤 원리를 사용해서 만들어지는지 제대로 이해하지 못하면, 실무에서 한글 글자가 이상하게 깨져 나올 때, 근본적인 원인을 찾아서 해결하기 어렵다.&#x20;

본격적으로 입출력을 다루기 전에, 가장 기본적인 컴퓨터가 데이터를 저장하는 원리부터 시작해서, 실무에 꼭 필요한 문자 인코딩까지 기본 이른을 확실히 이해하고 넘어가자.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 12.59.53.png" alt=""><figcaption></figcaption></figure>

컴퓨터는 메모리 반도체로 만들어져 있는데, 이것은 쉽게 이야기해서 수 많은 전구들이 모여있는 것이다. 이 전구들은 사실 트랜지스터라고 불리는 아주 작은 전자 스위치이다.&#x20;

이 트랜지스터들이 모여 메모리를 구성한다. 우리가 흔히 말하는 RAM 은 이런 방식으로 만들어진 메모리의 한 종류이다.&#x20;

컴퓨터가 정보를 저장하거나, 처리할 때 이 "전구들" 을 켜고 끄는 방식으로 데이터를 기록하고 읽어들인다. 이 과정은 매우 빠르게 일어나며, 현대의 컴퓨터 메모리는 초당 수십억 번의 데이터 접근을 처리할 수 있다.

여기서 핵심은 메모리라는 것은 단순히 전구를 켜고 끄는 방식으로 작동한다는 점이다. 그렇다면 여기에 우리가 사용하는 10진수 숫자 데이터를 어떻게 메모리에 저장할 수 있을까?&#x20;

### 2진수&#x20;

* 전구를 켜고 끈다는 것은 0과 1만 나타낼 수 있는 2진수로 표현할 수 있다.
* 전구 1개와 같이 2가지로만 표현할 수 있는 것을 1비트(bit) 라고 한다.
* 1bit 를 추가할 때마다 표현할 수 있는 숫자는 2배씩 늘어난다.

### 숫자 저장 예시

* 그렇다면 일반적으로 사용하는 10진수 100을 컴퓨터에 저장한다면 어떻게 될까?
* 컴퓨터는 10진수를 이해하지 못하기 때문에, 10진수 100을 2진수 1100100 으로 변경해서 저장한다.
* 음수를 표현해야 하는 경우 가장 앞 비트를 부호를 표현하는데 사용된다.
* bit 를 다룰 때 사용하는 2진수는 사람이 직관적으로 이해하기가 어렵다.

## 컴퓨터와 문자 인코딩1

간단한 수학 공식을 사용하면 사람이 사용하는 10진수를 컴퓨터가 사용하는 2진수로 쉽게 변경할 수 있다. \
따라서 컴퓨터는 10진수를 2진수로 변경해서 메모리(전구) 에 저장할 수 있다.&#x20;

그렇다면 숫자가 아닌 문자는 어떻게 메모리에 저장할 수 있을까? 컴퓨터는 전구를 켜고 끄는 2진수만 알고 있다. \
10진수는 정해진 수학 공식을 사용하면 쉽게 2진수로 변환할 수 있지만, 문자 'A' 를 2진수로 변경하는 수학 공식은 세상에 없다..&#x20;

때문에, 초창기 컴퓨터 과학자들은 문자 집합을 만들고, 각 문자에 숫자를 연결시키는 방법을 생각해냈다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-20 14.14.20.png" alt="" width="563"><figcaption></figcaption></figure>

예를 들어, 우리가 문자 'A' 를 저장하면 컴퓨터는 문자 집합을 통해 'A' 의 숫자 값 65를 찾아내고, 65를 메모리에 저장하게 된다. (물론 65는 2진수로 변경하여 저장한다)&#x20;

메모리에 저장된 문자를 불러올 때는 반대로 작동한다. 메모리에 저장된 숫자 값 65를 불러오고 문자 집합을 통해서 문자 'A' 를 찾아서 화면에 출력한다.&#x20;

* 문자 인코딩 : 문자 집합을 통해 문자를 숫자로 변환하는 것&#x20;
* 문자 디코딩 : 문자 집합을 통해 숫자를 문자로 변환하는 것

### ASCII 문자 집합

각 컴퓨터 회사가 독자적인 문자 집합을 사용한다면, 서로 다른 컴퓨터 간 호환이 되지 않는 문제가 발생한다.

이러한 문제를 해결하기 위해서 ASCII 라는 표준 문자 집합이 1960년도에 개발되었다.

초기 컴퓨터에는 주로 영문 알파벳, 숫자, 키보드의 특수문자, 스페이스, 엔터와 같은 기본적인 문자만 표현하면 충분하다. \
따라서 7비트를 사용해 총 128가지 문자를 표현할 수 있는 ASCII 공식 문자 집합이 만들어졌다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-20 14.18.43.png" alt="" width="563"><figcaption></figcaption></figure>

### ISO\_8859\_1

서유럽을 중심으로 컴퓨터 사용 인구가 늘어나면서, 서유럽 문자를 표현하는 문자 집합이 필요해졌다.&#x20;

#### ISO\_8859\_1

* 1980 년도
* 기본 ASCII 에 서유럽 문자의 추가 필요
* 국제 표준화 기구에서 서유럽 문자를 추가한 새로운 문자 규격을 만듬&#x20;
* ISO\_8859\_1, LATIN1, ISO-LATIN-1 등으로 불림&#x20;
  * 1바이트 문자 집합 -> 256 가지의 표현 가능&#x20;
  * 기존 ASCII 유지&#x20;
  * ASCII 에 128가지 문자 추가함 (주로 서유럽 문자, 추가 특수 문자들) 예) `À, Á, Â, Ã, Ä, Å`
* 기본 ASCII 문자 집합과 호환 가능&#x20;

### 한글 문자 집합

한국에도 컴퓨터 사용 인구가 늘어나면서, 한글을 표현할 수 있는 문자 집합이 필요해졌다.&#x20;

#### EUC-KR

* 1980 년도
* 초창기 등장한 한글 문자 집합
* 모든 한글을 담는 것 보다는 자주 사용하는 한글 2350개만 포함해서 만들었다.
* 한글의 글자는 아주 많기 때문에, 256 가지만 표현할 수 있는 1 byte 로 표현하는 것은 불가하다.
* &#x20;2 byte 를 사용하면 65536 가지의 표현 가능
* ASCII + 자주 사용하는 한글 2350 개 + 한국에서 자주 사용하는 기타 글자
  * 한국에서 자주 사용하는 한자 4888개
  * 일본어 가타카나 등도 함께 포함
* ASCII 는 1 byte, 한글은 2 byte 를 사용한다.
* 기존 ASCII 문자 집합과 호환 가능

#### MS949

* 1990 년도
* MS 가 EUC-KR 을 확장하여 만든 문자 집합
* 한글 초성, 중성, 종성 모두 조합하면 가능한 한글의 수는 총 11,172자&#x20;
* EUC-KR 은 "쀍", "삡" 과 같은 드물게 사용하는 음절을 표현하지 못함
* 기존 EUC-KR 과 호환을 이루면서, 한글 11,172 자를 모두 수용하도록 만든 것이 MS949
* EUC-KR 과 마찬가지로 ASCII 는 1 byte, 한글은 2 byte 를 사용함
* 기존 ASCII 문자 집합과 호환 가능
* 윈도우 시스템에서 계속 사용됨

## 컴퓨터와 문자 인코딩2

### 전세계 문자 집합

전세계적으로 컴퓨터 인구가 늘어나면서, 전세계 문자를 대부분 다 표현할 수 있는 문자 집합이 필요해졌다.

#### 문제점

* EUC-KR 이나 MS949 같은 한글 문자표를 PC 에 설치하지 않으면 다른 나라 사람들은 한글로 작성된 문서를 열어볼 수 없다.
* 우리도 마찬가지로, 히브리어, 아랍어를 보려만 각 나라의 문자 집합이 필요하다.
* 한 문서 안에 영어, 한글, 중국어, 일본어, 히브리어, 아랍어를 함께 저장해야 한다면 문제가 생긴다.
* 1980년대 말, 다양한 문자 인코딩 표준이 존재했지만, 이들은 모두 특정 언어 또는 문제 세트를 대상으로 했기 때문에, 국제적으로 호환성 문제가 많았다.

#### 유니코드의 등장

* 이를 해결하기 위해서 전 세계의 모든 문자들을 단일 문자 세트로 표현할 수 있는 유니코드(Unicode) 표준이 1990년대에 도입되었다.
* 전세계의 모든 문자와 기호를 하나의 표준으로 통합하여 표현할 수있는 문자 집합을 만든 것
* UTF-16, UTF-8  의 시작
* 두 표준이 비슷하게 등장했고, 초반에는 UTF-16 이 인기

#### UTF-16

* 1990 년도
* &#x20;16bit(2byte) 기반
* 자주 사용하는 기본 다국어들은 2byte 로 표현, 2byte 는 65536 가지를 표현할 수 있다.
  * 영어, 유렵 언어, 한국어, 중국어 일본어 등이 2byte 를 사용한다.
* 그 외는 문자는 4byte 로 표현 4byte 는 42억 가지를 표현할 수 있다.
  * 고대 문자, 이모지, 중국어 확장 한자 등 ..
* **단점 : ASCII 영문도 2byte 를 사용한다. ASCII 와 호환되지 않음**
  * UTF-16 을 사용한다면, 영문의 경우 다른 문자 집합 보다 2배의 메모리를 더 사용한다.
  * 웹에 있는 문서의 80% 는 영문 문서이다.
  * ASCII 와 호환되지 않는다는 점도 큰 단점 중 하나이다. (하위 호환의 부재)
* 초반에는 UTF-16 이 인기, 이 시기에 등장한 자바도 언어 내부적으로 문자를 표현할 때, UTF-16을 사용함, \
  그래서 자바의 `char` 타입이 2byte 를 사용함
  * 자바 내부에서 영문으로 된 문자를 저장할 때 2byte 저장한다는게 상당히 비효율적이다..
  * 자바 9 부터는영문으로 된 문자를 저장한다면 `char` 타입이 아닌 `byte` 타입으로 바꾸어서 저장한다.
* 대부분의 문자를 2byte 로 처리하기 때문에, 계산이 편리함

#### UTF-8

* 1990 년도
* 8bit(1byte) 기반, 가변길이 인코딩
* 1byte - 4byte 를 사용해서 문자를 인코딩
  * 1byte : ASCII, 영문, 기본 라틴 문자
  * 2byte : 그리스어, 히브리어, 라틴 확장 문자
  * 3byte : 한글, 한자, 일본어
  * 4byte : 이모지, 고대문자 등 ..
* 단점 : 상대적으로 사용이 복잡하다.
  * UTF-16 은 대부분의 기본 문자들이 2바이트로 표현되기 때문에, 문자열의 특정 문자에 접근하거나 문자 수를 세는 작업이 상대적으로 간단함.
  * 반면, UTF-8 에서는 각 문자가 가변 길이로 인코딩되므로 이런 작업이 더 복잡하다.
* 단점 : ASCII 를 제외한 일부 언어에서 더 많은 용량 사용
  * UTF-8 은 ASCII 문자를 1바이트로, 비 ASCII 문자를 2-4바이트로 인코딩한다.
  * 한글, 한자, 아랍어, 히브리어와 같은 문자들은 UTF-8 에서 3바이트 또는 4바이트로 차지한다.
  * 반면 UTF-16 에서는 이들 문자가 대부분 2바이트로 인코딩된다.
* **장점 : ASCII 문자는 1바이트로 표현, ASCII 호환**
* **현대의 "사실상 표준" 인코딩 기술**
  * 1990년도 후반 \~ 2000년도 초반에 인터넷과 웹이 빠르게 성장하면서 저변 확대
  * 2008년 W3C 웹 표준에 UTF-8 채택
  * 현재 대부분의 웹사이트와 애플리케이션에서 기본 인코딩으로 사용

### 정리

#### UTF-8 이 현대의 사실상 표준 인코딩 기술이 된 이유

* **저장 공간 절약과 네트워크 효율성**
  * UTF-8 은 ASCII 문자를 포함한 많은 서양 언어의 문자에 대해 1바이트를 사용한다.
  * 반면에 UTF-16 은 최소 2바이트를 사용하므로, 주로 ASCII 문자로 이루어진 영문 텍스트에서는 UTF-8이 2배는 효율적이다. 특히 데이터를 네트워크로 전달할 때는 매우 큰 효율의 차이를 보인다.
  * 참고로 웹이 있는 문서의 80% 이상은 영문 문서이다.
* **ASCII 와의 호환성**
  * UTF-8 은 ASCII 와 호환된다. UTF-8 로 인코딩 된 텍스트에서 ASCII 범위에 있는 문자는 기존 ASCII 와 동일한 방식으로 처리된다.
  * 이로 인해 수 많은 레거시 시스템과도 호환을 유지하면서도 전 세계의 모든 문자를 표현할 수 있다.
* **결론 : UTF-8 을 사용하자!**&#x20;

#### 참고 : 한글 윈도우의 경우 기존 윈도우와 호환성 때문에, 기본 인코딩을 MS949 로 유지한다. 한글 윈도우 기본 인코딩을 UTF-8 로 변경하려고 노력중이다.&#x20;

## 문자 집합 조회

문자 집합을 사용해서 문자 인코딩을 어떻게 하는지 코드로 알아보자. 먼저 사용 가능한 문자 집합을 조회해보자.&#x20;

#### 사용 가능한 문자 집합 조회

```java
package charset;

import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.Set;
import java.util.SortedMap;

public class AvailatbleCharsetMain {
    public static void main(String[] args) {
        
        // 이용 가능한 모든 Chatset 자바 + OS
        SortedMap<String, Charset> charsets = Charset.availableCharsets();
        for (String charsetName : charsets.keySet()) {
            System.out.println("charsetName = " + charsetName);
        }

        System.out.println("=====");
        // 문자로 조회(대소문자 구분X), MS949, ms949
        Charset charset1 = Charset.forName("MS949");
        System.out.println("charset1 = " + charset1);
        
        // 별칭 조회 
        Set<String> aliases1 = charset1.aliases();
        for (String alias : aliases1) {
            System.out.println("alias1 = " + alias);
        }

        // UTF-8 문자로 조회
        Charset charset2 = Charset.forName("UTF-8");
        System.out.println("charset2 = " + charset2);

        // UTF-8 상수로 조회
        Charset charset3 = StandardCharsets.UTF_8;
        System.out.println("charset3 = " + charset3);

        // 시스템의 기본 Charset
        Charset charset4 = Charset.defaultCharset();
        System.out.println("charset4 = " + charset4);
    }
}
```

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 13.48.24.png" alt=""><figcaption></figcaption></figure>

#### 이용가능한 모든 문자 집합 조회

`Charset.availableCharsets()` 를 사용하면 이용가능한 모든 문자 집합을 조회할 수 있다.

여기에는 자바가 기본으로 제공하는 문자 집합과 OS가 제공하는 문자 집합을 포함한다.

#### Charset.forName()

특정 문자 집합을 지정해서 찾을 때는 `Charset.forName(...)` 을 사용하면 된다. 인자로 문자 집합의 이름이나 별칭을 사용하면 되는데, 대소문자는 구분하지 않는다.\
별칭은 `aliases()` 메서드를 사용하면 구할 수 있다.\
예를 들어, MS949는 MS949, ms949, ms\_949, windows-949, windows949, x-windows-949 등으로 찾을 수 있다.

#### StandardCharsets.UTF\_8

자주 사용하는 문자 집합은 `StandardCharsets` 에 상수로 지정되어 있다.

```java
public final class StandardCharsets {
      public static final Charset US_ASCII = sun.nio.cs.US_ASCII.INSTANCE;
      public static final Charset ISO_8859_1 = sun.nio.cs.ISO_8859_1.INSTANCE;
      public static final Charset UTF_8 = sun.nio.cs.UTF_8.INSTANCE;
      public static final Charset UTF_16BE = new sun.nio.cs.UTF_16BE();
      public static final Charset UTF_16LE = new sun.nio.cs.UTF_16LE();
      public static final Charset UTF_16 = new sun.nio.cs.UTF_16();
}
```

#### Charset.defaultCharset()

현재 시스템에서 사용하는 기본 문자 집합을 반환한다.

## 문자 인코딩 예제1&#x20;

`Charset` (문자 집합)을 사용해서 실제 문자 인코딩을 해보자.

```java
package charset;

import java.nio.charset.Charset;
import java.util.Arrays;

import static java.nio.charset.StandardCharsets.*;

public class EncodingMain1 {
    private static final Charset EUC_KR = Charset.forName("EUC-KR");
    private static final Charset MS_949 = Charset.forName("MS949");

    public static void main(String[] args) {
        System.out.println("== ASCII 영문 처리 ==");
        encoding("A", US_ASCII);
        encoding("A", ISO_8859_1);
        encoding("A", EUC_KR);
        encoding("A", UTF_8);
        encoding("A", UTF_16BE);

        System.out.println("== 한글 지원 ==");
        encoding("가", EUC_KR);
        encoding("가", MS_949);
        encoding("가", UTF_8);
        encoding("가", UTF_16BE);

        System.out.println("== 문자 byte 변경 ==");
        String str = "A";
        byte[] bytes = str.getBytes();  // 문자 집합을 지정하지 않으면 기본 문자 집합을 사용한다.
        System.out.println("bytes = " + Arrays.toString(bytes));
    }

    private static void encoding(String text, Charset charset) {
        byte[] bytes = text.getBytes(charset);
        System.out.printf("%s -> [%s] 인코딩 -> %s %sbyte\n", text, charset, Arrays.toString(bytes), bytes.length);
    }
}
```

* 문자를 컴퓨터가 이해할 수 있는 숫자(byte) 로 변경하는 것을 문자 인코딩이라 한다.
* String.getByte(CharSet) 메서드를 사용하면 String 문자를 byte 배열로 변경할 수 있다.
* 이때 중요한 점이 있는데, 문자를 byte 로 변경하려면 문자 집합이 필요하다는 점이다. 따라서 어떤 문자 집합을 참고해서 byte 로 변경할지 정해야 한다. String.getByte() 의 인자로 Chatset 객체를 전달하면 된다.
  * **문자 집합을 지정하지 않으면 현재 시스템에서 사용하는 기본 문자 집합을 인코딩에 사용한다.**

#### **실행 결과**

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.14.28.png" alt=""><figcaption></figcaption></figure>

## 문자 인코딩 예제2

이번에는 문자를 인코딩은 물론이고, 디코딩까지 해보자. 그리고 좀 더 다양한 예시를 찾아보자.

```java
package charset;

import java.nio.charset.Charset;
import java.nio.charset.StandardCharsets;
import java.util.Arrays;

import static java.nio.charset.StandardCharsets.*;

public class EncodingMain2 {
    private static final Charset EUC_KR = Charset.forName("EUC-KR");
    private static final Charset MS_949 = Charset.forName("MS949");

    public static void main(String[] args) {
        System.out.println("== 영문 ASCII 인코딩 ==");
        test("A", US_ASCII, US_ASCII);
        test("A", US_ASCII, ISO_8859_1);
        test("A", US_ASCII, EUC_KR);
        test("A", US_ASCII, MS_949);
        test("A", US_ASCII, UTF_8);
        test("A", US_ASCII, UTF_16BE);  // 디코딩 실패 케이스

        System.out.println("== 한글 인코딩 - 기본 ==");
        test("가", US_ASCII, US_ASCII);
        test("가", ISO_8859_1, ISO_8859_1);
        test("가", EUC_KR, EUC_KR);
        test("가", MS_949, MS_949);
        test("가", UTF_8, UTF_8);
        test("가", UTF_16BE, UTF_16BE);

        System.out.println("== 한글 인코딩 - 복잡한 문자 ==");
        test("뷁", US_ASCII, US_ASCII);
        test("뷁", ISO_8859_1, ISO_8859_1);
        test("뷁", EUC_KR, EUC_KR);
        test("뷁", MS_949, MS_949);
        test("뷁", UTF_8, UTF_8);
        test("뷁", UTF_16BE, UTF_16BE);

        System.out.println("== 한글 인코딩 - 디코딩이 다른 경우 ==");
        test("가", EUC_KR, MS_949);
        test("뷁", MS_949, EUC_KR);      // 인코딩 가능, 디코딩 X
        test("가", EUC_KR, US_ASCII);    // 인코딩 가능, 디코딩 X
        test("가", MS_949, UTF_8);    // 인코딩 가능, 디코딩 X
        test("가", UTF_8, MS_949);    // 인코딩 가능, 디코딩 X

        System.out.println("== 영문 인코딩 - 디코딩이 다른 경우");
        test("A", EUC_KR, UTF_8);
        test("A", MS_949, UTF_8);
        test("A", UTF_8, MS_949);
        test("A", UTF_8, UTF_16BE);
    }

    private static void test(String text, Charset encodingCharset, Charset decodingCharset) {
        byte[] encoded = text.getBytes(encodingCharset);
        String decoded = new String(encoded, decodingCharset);
        System.out.printf("%s -> [%s] 인코딩 -> %s %sbyte -> [%s] 디코딩 -> %s\n",
                text, encodingCharset, Arrays.toString(encoded), encoded.length,
                decodingCharset, decoded);
    }
}
```

#### 실행 결과

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.22.00.png" alt=""><figcaption></figcaption></figure>

#### 영문 ASCII 인코딩&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.23.24.png" alt=""><figcaption></figcaption></figure>

* 영문 'A' 를 인코딩하면 1byte 를 사용하고 숫자 65가 된다.
* 숫자 65를 디코딩하면 UTF-16 을 제외하고 모두 디코딩이 가능하다.
  * UTF-16 의 경우 디코딩의 실패해서 � 라는 특수문자가 출력되었다.&#x20;
* ASCII 는 UTF-16 을 제외한 대부분의 문자 집한에 호환된다.&#x20;

#### 한글 인코딩 - 기본&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.26.03.png" alt=""><figcaption></figcaption></figure>

* 한글 '가' 는 ASCII, ISO-8859-1 로 인코딩 할 수 없다.&#x20;
  * 이 경우 흥미롭게도 숫자 63이 되는데 63은 ASCII 로 ? 라는 뜻이다. 한마디로 모르는 이상한 문자가 인코딩 되었다는 뜻이다.&#x20;
* EUC-KR, MS949, UTF-8, UTF-16 은 한글 인코딩 디코딩이 잘 수행되는 것을 확인할 수 있다.&#x20;
  * 2byte : EUC-KR, MS949, UTF-16
  * 3byte : UTF-8
* 한글 '가'는 EUC-KR, MS949 모두 같은 값을 반환한다 따라서 서로 호환된다.

#### 한글 인코딩 - 복잡한 문자

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.28.44.png" alt=""><figcaption></figcaption></figure>

* EUC-KR은 자주 사용하는 한글 2350개만 표현할 수 있다. 따라서 '뷁'과 문자는 문자 집합에 없으므로 인코딩할 수 없다.
* MS949, UTF-8, UTF-16은 모든 한글을 표현할 수 있다. 따라서 '뷁'과 같은 문자도 인코딩 할 수 있다.

#### 한글 인코딩 - 디코딩이 다른 경우

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.29.50.png" alt=""><figcaption></figcaption></figure>

* '가'와 같이 자주 사용하는 한글은 EUC-KR, MS949 서로 호환된다. 따라서 EUC-KR로 인코딩해도 MS949로 디코딩 할 수 있다. MS949는 EUC-KR을 포함한다.
* '뷁'과 같이 특수한 한글은 MS949로 인코딩 할 수 있지만, EUC-KR의 문자 집합에 없으므로 EUC-KR로 디코딩 할 수 없다.
* 한글을 인코딩할 때 UTF-8과 EUC-KR(MS949)는 서로 호환되지 않는다.

#### 영문 인코딩 - 디코딩이 다른 경우

<figure><img src="../../../../.gitbook/assets/스크린샷 2025-05-04 14.31.13.png" alt=""><figcaption></figcaption></figure>

* ASCII에 포함되는 영문은 UTF-16을 제외한 대부분의 문자 집합에서 호환된다.
