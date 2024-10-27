# 문자 인코딩

## 1. 컴퓨터와 데이터&#x20;

* 개발자가 개발하며 다루는 데이터는 크게 0101 로 되어있는 바이너리 데이터(또는 byte 기반 데이터) 와 "ABC" 와 같은 문자로 되어 있는 텍스트 데이터 두 가지이다.&#x20;
* 컴퓨터는 메모리 반도체로 만들어져 있는데, 이것은 쉽게 이야기해서 수 많은 전구들이 모여있는 것이다. 이 전구들은 사실 트랜지스터라고 불리는 아주 작은 전자 스위치이다.&#x20;
* 이 트랜지스터들이 모여 메모리를 구성한다. 우리가 흔히 말하는 RAM 은 이런 방식으로 만들어진 메모리의 한 종류이다.&#x20;
* 컴퓨터가 정보를 저장하거나, 처리할 때 이 '전구들' 을 켜고 끄는 방식으로 데이터를 기록하고 읽어들인다. 이 과정은 매우 빠르게 일어나며, 현대의 컴퓨터 메모리는 초당 수십억 번의 데이터 접근을 처리할 수 있다.&#x20;
* 여기서 핵심은 메모리라는 것은 단순히 전구를 켜고 끄는 방식으로 작동한다는 점이다. 그렇다면 여기에 우리가 사용하는 10진수 숫자 데이터를 어떻게 메모리에 저장할 수 있을까?&#x20;

### 2진수&#x20;

* 전구를 켜고 끈다는 것은 0과 1만 나타낼 수 있는 2진수로 표현할 수 있다.&#x20;
* 전구 1개와 같이 2가지로만 표현할 수 있는 것을 1비트(bit) 라고 한다.&#x20;
* 1bit 를 추가할 때마다 표현할 수 있는 숫자는 2배씩 늘어난다.&#x20;

### 숫자 저장 예시&#x20;

* 그렇다면 일반적으로 사용하는 10진수 100을 컴퓨터에 저장한다면 어떻게 될까?&#x20;
* 컴퓨터는 10진수를 이해하지 못하기 때문에, 10진수 100을 2진수 1100100 으로 변경해서 저장한다.&#x20;
* bit 를 다룰 때 사용하는 2진수는 사람이 직관적으로 이해하기가 어렵다.&#x20;

## 2. 컴퓨터와 문자 인코딩1 - ASCII 기반 문자 집합

* 간단한 수학 공식을 사용하면 사람이 사용하는 10진수를 컴퓨터가 사용하는 2진수로 쉽게 변경할 수 있다. 따라서 컴퓨터는 10진수를 2진수로 변경해서 메모리(전구) 에 저장할 수 있다.&#x20;
* 그렇다면 숫자가 아닌 문자는 어떻게 메모리에 저장할 수 있을까? 컴퓨터는 전구를 켜고 끄는 2진수만 알고 있다. 10진수는 정해진 수학 공식을 사용하면 쉽게 2진수로 변환할 수 있지만, 문자 'A' 를 2진수로 변경하는 수학 공식은 세상에 없다..&#x20;
* 때문에, 초창기 컴퓨터 과학자들은 문자 집합을 만들고, 각 문자에 숫자를 연결시키는 방법을 생각해냈다.&#x20;
  * 예를 들어, 우리가 문자 'A' 를 저장하면 컴퓨터는 문자 집합을 통해 'A' 의 숫자 값 65를 찾아내고, 65를 메모리에 저장하게 된다. (물론 65는 2진수로 변경하여 저장한다)&#x20;
  * 메모리에 저장된 문자를 불러올 때는 반대로 작동한다. 메모리에 저장된 숫자 값 65를 불러오고 문자 집합을 통해서 문자 'A' 를 찾아서 화면에 출력한다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-20 14.14.20.png" alt="" width="563"><figcaption></figcaption></figure>

### ASCII

* 각 컴퓨터 회사가 독자적인 문자 집합을 사용한다면, 서로 다른 컴퓨터 간 호환이 되지 않는 문제가 발생한다.&#x20;
* 이러한 문제를 해결하기 위해서 ASCII 라는 표준 문자 집합이 1960년도에 개발되었다.&#x20;
* 초기 컴퓨터에는 주로 영문 알파벳, 숫자, 키보드의 특수문자, 스페이스, 엔터와 같은 기본적인 문자만 표현하면 충분해다. 따라서 7비트를 사용해 총 128가지 문자를 표현할 수 있는 ASCII 공식 문자 집합이 만들어졌다.&#x20;

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-20 14.18.43.png" alt="" width="563"><figcaption></figcaption></figure>

### ISO\_8859\_1

* 서유럽을 중심으로 컴퓨터 사용 인구가 늘어나면서, 서유럽 문자를 표현하는 문자 집합이 필요해졌다.&#x20;
* 1980 년도&#x20;
* 기본 ASCII 에 서유럽 문자의 추가 필요
* 국제 표준화 기구에서 서유럽 문자를 추가한 새로운 문자 규격을 만듬&#x20;
* ISO\_8859\_1, LATIN1, ISO-LATIN-1 등으로 불림&#x20;
  * 1바이트 문자 집합 -> 256 가지의 표현 가능&#x20;
  * 기존 ASCII 유지&#x20;
  * ASCII 에 128가지 문자 추가&#x20;
* 기본 ASCII 문자 집합과 호환 가능&#x20;

## 3. 컴퓨터와 문자 인코딩2 - 한글 문자 집합&#x20;

* 한국에도 컴퓨터 사용 인구가 늘어나면서, 한글을 표현할 수 있는 문자 집합이 필요해졌다.&#x20;

### EUC-KR

* 1980 년도&#x20;
* 초창기 등장한 한글 문자 집합&#x20;
* 모든 한글을 담는 것 보다는 자주 사용하는 한글 2350개만 포함해서 만들었다.&#x20;
* 한글의 글자는 아주 많기 때문에, 256 가지만 표현할 수 있는 1 byte 로 표현하는 것은 불가하다.&#x20;
* &#x20;2 byte 를 사용하면 65536 가지의 표현 가능
* ASCII + 자주 사용하는 한글 2350 개 + 한국에서 자주 사용하는 기타 글자&#x20;
  * 한국에서 자주 사용하는 한자 4888 개&#x20;
  * 일본어 가타카나 등도 함께 포함&#x20;
* ASCII 는 1 byte, 한글은 2 byte 를 사용ㅎ나다.&#x20;
* 기존 ASCII 문자 집합과 호환 가능&#x20;

### MS949

* 1990 년도&#x20;
* MS 가 EUC-KR 을 확장하여 만든 문자 집합
* 한글 초성, 중성, 종성 모두 조합하면 가능한 한글의 수는 총 11,172 자&#x20;
* EUC-KR 은 삡' 과 같은 드물게 사용하는 음절을 표현하지 못함&#x20;
* 기존 EUC-KR 과 호환을 이루면서, 한글 11,172 자를 모두 수용하도록 만든 것이 MS949&#x20;
* EUC-KR 과 마찬가지로 ASCII 는 1 byte, 한글은 2 byte 를 사용함&#x20;
* 기존 ASCII 문자 집합과 호환 가능&#x20;
* 윈도우 시스템에서 계속 사용됨&#x20;

## 4. 컴퓨터와 문자 인코딩3 - 전세계 문자 집합&#x20;

* 전세계적으로 컴퓨터 인구가 늘어나면서, 전세계 문자를 대부분 다 표현할 수 있는 문자 집합이 필요해졌다.&#x20;

#### 문제점&#x20;

* EUC-KR 이나 MS949 같은 한글 문자표를 PC 에 설치하지 않으면 다른 나라 사람들은 한글로 작성된 문서를 열어볼 수 없다.&#x20;
  * 우리도 마찬가지로, 히브리어, 아랍어를 보려만 각 나라의 문자 집합이 필요하다.&#x20;
* 한 문서 안에 영어, 한글, 중국어, 일본어, 히브리어, 아랍어를 함께 저장해야 한다면 문제가 생긴다.&#x20;
* 1980년대 말, 다양한 문자 인코딩 표준이 존재했지만, 이들은 모두 특정 언어 또는 문제 세트를 대상으로 했기 때문에, 국제적으로 호환성 문제가 많았다.&#x20;

### 유니코드&#x20;

* 이를 해결하기 위해서 전 세계의 모든 문자들을 단일 문자 세트로 표현할 수 있는 유니코드(Unicode) 표준이 1990년대에 도입되었다.&#x20;
* 전세계의 모든 문자와 기호를 하나의 표준으로 통합하여 표현할 수있는 문자 집합을 만든 것&#x20;
* UTF-16, UTF-8  의 문자 집합
* 두 표준이 비슷하게 등장했고, 초반에는 UTF-16 이 인기가 있었다.&#x20;

### UTF-16

* 1990 년도&#x20;
* &#x20;16bit(2byte) 기반
* 자주 사용하는 기본 다국어들은 2byte 로 표현, 2byte 는 65536 가지를 표현할 수 있다 .
  * 영어, 유렵 언어, 한국어, 중국어 일본어 등이 2byte 를 사용한다.&#x20;
* 그 외는 4byte 로 표현 4byte 는 42억 가지를 표현할 수 있다.&#x20;
  * 고대 문자, 이모지, 중국어 확장 한자 등 ..&#x20;
* **단점 : ASCII 영문도 2byte 를 사용한다. ASCII 와 호환되지 않음**&#x20;
  * UTF-16 을 사용한다면, 영문의 경우 다른 문자 집합 보다 2배의 메모리를 더 사용한다.&#x20;
  * 웹에 있는 문서의 80% 는 영문 문서이다.&#x20;
  * ASCII 와 호환되지 않는다는 점도 큰 단점 중 하나이다.&#x20;
* 초반에는 UTF-16 이 인기, 이 시기에 등장한 자바도 언어 내부적으로 문자를 표현할 때, UTF-16을 사용함, 그래서 자바의 char 타입이 2byte 를 사용함&#x20;
* 대부분의 문자를 2byte 로 처리하기 때문에, 계산이 편리함.&#x20;

### UTF-8

* 1990 년도&#x20;
* 8bit(1byte) 기반, 가변길이 인코딩&#x20;
* 1byte - 4byte 를 사용해서 문자를 인코딩
  * 1byte : ASCII, 영문, 기본 라틴 문자
  * 2byte : 그리스어, 히브리어, 라틴 확장 문자&#x20;
  * 3byte : 한글, 한자, 일본어
  * 4byte : 이모지, 고대문자 등 ..&#x20;
* 단점 : 상대적으로 사용이 복잡하다.&#x20;
  * UTF-16 은 대부분의 기본 문자들이 2바이트로 표현되기 때문에, 문자열의 특정 문자에 접근하거나 문자 수를 세는 작업이 상대적으로 간단함.&#x20;
  * 반면, UTF-8 에서는 각 문자가 가변 길이로 인코딩되므로 이런 작업이 더 복잡하다.&#x20;
* 단점 : ASCII 를 제외한 일부 언어에서 더 많은 용량 사용&#x20;
  * UTF-8 은 ASCII 문자를 1바이트로, 비 ASCII 문자를 2-4바이트로 인코딩한다.&#x20;
  * 한글, 한자, 아랍어, 히브리어와 같은 문자들은 UTF-8 에서 3바이트 또는 4바이트로 차지한다.&#x20;
  * 반면 UTF-16 에서는 이들 문자가 대부분 2바이트로 인코딩된다.&#x20;
* 장점 : ASCII 문자는 1바이트로 표현, ASCII 호환&#x20;
* 현대의 사실상 표준 인코딩 기술&#x20;
  * 1990년도 후반 \~ 2000년도 초반에 인터넷과 웹이 빠르게 성장하면서 저변 확대&#x20;
  * 2008년 W3C 웹 표준에 UTF-8 채택&#x20;
  * 현재 대부분의 웹사이트와 애플리케이션에서 기본 인코딩으로 사용&#x20;

### 정리&#x20;

#### UTF-8 이 현대의 사실상 표준 인코딩 기술이 된 이유&#x20;

* 저장 공간 절약과 네트워크 효율성&#x20;
  * UTF-8 은 ASCII 문자를 포함한 많은 서양 언어의 문자에 대해 1바이트를 사용한다.&#x20;
  * 반면에 UTF-16 은 최소 2바이트를 사용하므로, 주로 ASCII 문자로 이루어진 영문 텍스트에서는 UTF-8이 2배는 효율적이다. 특히 데이터를 네트워크로 전달할 때는 매우 큰 효율의 차이를 보인다.&#x20;
  * 참고로 웹이 있는 문서의 80% 이상은 영문 문서이다.&#x20;
* ASCII 와의 호환성&#x20;
  * UTF-8 은 ASCII 와 호환된다. UTF-8 로 인코딩 된 텍스트에서 ASCII 범위에 있는 문자는 기존 ASCII 와 동일한 방식으로 처리된다.&#x20;
  * 이로 인해 수 많은 레거시 시스템과도 호환을 유지하면서도 전 세계의 모든 문자를 표현할 수 있다.&#x20;

### 결론&#x20;

#### UTF-8 을 사용하자!&#x20;

#### 참고 : 한글 윈도우의 경우 기존 윈도우와 호환성 때문에, 기본 인코딩을 MS949 로 유지한다. 한글 윈도우 기본 인코딩을 UTF-8 로 변경하려고 노력중이다.&#x20;

## 5. 문자 인코딩 예제1&#x20;

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

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-27 19.31.14.png" alt=""><figcaption></figcaption></figure>

## 6. 문자 인코딩 예제2&#x20;

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

<figure><img src="../../../../.gitbook/assets/스크린샷 2024-10-27 19.33.19.png" alt=""><figcaption></figcaption></figure>
