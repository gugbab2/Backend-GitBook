# 캐스팅(형변환, casting)

#### 암시적 캐스팅

* 컴파일러가 형을 변환해준다.
* 형 변환이 허용될 때만 해준다.

```cpp
int number1 = 3;
long number2 = number1;
```

#### 명시적 캐스팅

* 프로그래머가 형 변환을 위한 코드를 직접 작성
* C++ 의 캐스팅은 다음과 같다.
  * static\_cast
  * const\_cast
  * dynamic\_cast
  * reinterpret\_cast

#### C 스타일 캐스팅

* 아래 코드는 무엇을 할까? (자바에서도 다음과 같은 캐스팅 방법을 사용한다)
* 다음의 캐스팅 방법은 무슨 문제를 가지고 있을까?
  * C++ 에서 제공하는 캐스팅 중(4가지 캐스팅 방법) 하나를 하기 때문에, 명확하지 못하다...
  * 명백한 실수일 때라도, 컴파일러가 캐치 하지 못한다.
  * C++ 캐스팅은 명시적으로 캐스팅 해 다음과 같은 문제를 해결했다.\
    \-> 하지만 하드웨어는 C, C++ 캐스팅을 구분하지 못한다.

```cpp
int score = (int)someVariable;
```

## 정적캐스팅(static\_cast)

#### 1. 값

* 두 숫자형 값을 변환
  * 값을 유지하려고 노력 (올림의 오차는 제외)
  * _**이진수 표기는 달라질 수 있다.**_\
    \-> float 를 int 로 변환했을 때, 소수점 아래는 버려지기 때문에, 이진수 표현이 다를 수 있다.

```cpp
int number1 = 3;
short number2 = static_cast<short>(number1);

// number1 -> 0000 0000 0000 0011
// number2 -> 0000 0000 0000 0011

//---------------------------------------

float number1 = 3.f;
int number2 = static_cast<int>(number1);

// number1 -> 0100 0000 0100 0000
// number2 -> 0000 0000 0000 0011
```

#### 2. 개체 포인터

* **컴파일 시** 상속 관계 확인 후 자식 클래스로 변환
* **런타임 시** 실제 크래시가 날 수 있다.\
  \-> 컴파일 타임에는 상속 관계 만을 확인하는 것이기 때문에, 함수를 호출 가능한 것인지 까지는 확인하지 않는다.

<pre class="language-cpp"><code class="lang-cpp">Animal* myPet = new Cat(2, "COCO");

Cat* myCat = static_cast&#x3C;Cat*>(myPet);    // OK

Dog* myDog = static_cast&#x3C;Dog*>(myPet);    // 상속관계만 형성된다면 컴파일 시 문제가 되지 않지만,
myDog->GetDogHouseName();                 // 런타임에 가상함수를 실행하면 문제가 생길 수 있다.

//---------------------------------------

Animal* myPet = new Cat(2, "COCO");
House* myHouse = static_cast&#x3C;House*>(myPet);    // C 스타일의 캐스팅에서는 컴파일러가 추측하는 형식이기에 문제가 되지 않았지만,
<strong>myHouse->GetAddress();                          // C++ 캐스팅은 컴파일러가 상속관계를 파악하기에 컴파일 에러가 난다.
</strong></code></pre>

## 재해석 캐스팅(reinterpret\_cast)

_**-> 상당히 위험한 캐스팅이다!**_

* 전혀 연관없는 두 포인터의 형변환을 가능하게 해준다.
  * Cat\* ↔️ House\*
  * int\* ↔️ char\*
* 포인터와 포인터가 아닌 변수 사이의 형변환을 가능하게 해준다.
  * Cat\* ↔️ unsigned int
*   하지만, _**캐스팅 되더라도 이진수 표기는 달라지지 않는다!**_\
    \-> A 형의 이진수 표기를 B 형인 것처럼 해석한다는 것이다.(위험해보인다 .. 알아듣기도 힘들어 ..)\
    \-> _**컴퓨터는 이진수의 값을 코드의 시점에서 해석하는 것이다!**_\
    _**-> 개발자의 의도가 있을 것이라고 생각하고, 문법적으로 이상한 캐스팅이라더라도 허용해준다.**_&#x20;

    <figure><img src="../../.gitbook/assets/스크린샷 2023-11-18 15.56.54.png" alt=""><figcaption></figcaption></figure>

```cpp
int* signedNumber = new int(-10);

// 컴파일 에러, 컴파일 타임에 에러를 잡아주려 static_cast 가 나온것이다.
unsigned int* unsignedNumber1 = static_cast<unsugned int*>(signedNumber);

// 정상, unsugned int 의 시점에서 해석하도록 한다.
// 대부분 이렇게까지 복잡한 생각을 요구하지 않는다!(대부분 실수이다)        
unsigned int* unsignedNumber2 = reinterpret_cast<unsugned int*>(signedNumber);
```

## const\_cast

* _**const\_cast 로는 형을 바꾸는 것이 아닌, const 를 제거할 때 사용한다.**_

<pre class="language-cpp"><code class="lang-cpp">Animal* myPet = new Cat(2, "COCO");
const Animal* petptr = myPet;

// C Style
<strong>Animal* myAnimal1 = (Animal*) petptr;   // OK
</strong>Cat* myCat1 = (Cat*) petptr;            // OK

// C++ Style
Animal* myAnimal2 = const_cast&#x3C;Animal*>(petptr);   // OK
Cat* myCat2 = const_cast&#x3C;Cat*>(petptr);            // 컴파일 에러! 형변환을 허용하지 않는다. 
</code></pre>

* 포인터 형에서만 사용하는 것이 말이 된다.\
  \-> 값 복사는 원본의 영향을 주지 않기에 문제가 안된다!
* const\_cast 를 하고 있다면 무언가 단단히 잘못되는 것이다!\
  \-> 아래 함수를 살펴보면 함수 인터페이스에서 매개변수를 바꾸지 않겠다고 선언했는데!\
  \-> const 를 없애 매개변수의 값을 바꾼다? 말이 안된다!
* _**외부 라이브러리를 사용할 때만 제한적으로 const\_cast 를 사용하자**_\
  \-> 내가 변경 권한이 없는 라이브러리를 사용할 때만!\
  \-> 외부 라이브러리의 인터페이스가 잘못 설계되어 있을 때!

```cpp
void Foo(const char* name)
{
    char* str = const_cast<char*>(name);
    *str = 'p';
}
```

## 동적캐스팅(dynamic\_cast)

* 런타임에 캐스팅이 가능한지 판단을 하고 캐스팅이 가능할 때는 캐스팅, 아니라면 null 을 반환한다.\
  **-> 때문에, dynamic\_cast 가 static\_cast 보다 더 안전하다.**

```cpp
// static_cast
Animal* myPet = new Cat(2, "COCO");

Cat* myCat = static_cast<Cat*>(myPet);    // OK

Dog* myDog = static_cast<Dog*>(myPet);    // 상속관계만 형성된다면 컴파일 시 문제가 되지 않지만,
myDog->GetDogHouseName();                 // 런타임에 가상함수를 실행하면 문제가 생길 수 있다.

//-------------------------------------------

// dynamic_cast
Animal* myPet = new Cat(2, "COCO");

Cat* myCat = dynamic_cast<Cat*>(myPet);    // OK

Dog* myDog = dynamic_cast<Dog*>(myPet);    // 런타임에 캐스팅 할 수 없기에 null 을 반환한다.

if(myDog != null)
{
    myDog->GetDogHouseName();              
}
```

* **하지만, dynamic\_cast 를 사용하기 위해서는 컴파일 중에 RTTI(실시간 타입정보, Real Time Type Infomation) 설정을 켜야한다!**\
  **-> 이 기능을 키지 않는다면 static\_cast 와 동일하게 작동한다.**
* 대부분의 프로젝트에서는 RTTI 가 꺼져있다.\
  \-> 왜? C++ 을 쓰는 업계는 성능이 중요하다고 했는데, RTTI 를 버틸수 있을 만큼의 성능을 가지고 있지 않는다.\
  \-> 정말 좋은 개념이지만 실제로는 거의 쓰이지 않는다..

## 왜 이리, C++ 에는 문제가 많아..? 그럼 Best Practice 는 무엇이야?

1. 기본적으로 static\_cast 를 사용하자
2. 코드에서 정말 필요할 때만 reinterpret\_cast 를 사용하자
3. 외부 라이브러리를 사용할 때만 제한적으로 const\_cast 를 사용하자\
   \-> 내가 변경 권한이 없는 라이브러리를 사용할 때만!\
   \-> 외부 라이브러리의 인터페이스가 잘못 설계되어 있을 때!
