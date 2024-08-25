# 문자열(string)

#### 기존 C 의 코드

* 기존 C 의 코드에는 다음과 같은 상황일 때 동작하지 않는다.
  * 아무것도 읽지 못했을 때
  * 한 줄의 문자가 256자 이상일 때(즉, 버퍼가 충분하지 않을 때)

```cpp
char line[256];
cin.getline(line, 256);
```

#### C++ 의 std::string 클래스

* 기존의 문자열을 읽는 방식과 다르게,&#x20;
* **string 자료형은 연산 시, 크기가 변하기 때문에 입력 받는 크기의 제한이 없다.**

<pre class="language-cpp"><code class="lang-cpp"><strong>#include &#x3C;string>
</strong>std::string firstName;
std::cin >> firstName;

string firstName = "POPE";
string fullName = "John Doe";

// 대입
fullName = firstName;
// 덧붙이기
fullName += " KIM";

// 비교연산 가능
if(firstName1 == firstName2)
{
}

// 사전 상의 순서를 비교 
if(firstName1 > firstName2)
{
}

// 문자열의 길이를 반환
cout &#x3C;&#x3C; firstName.size() &#x3C;&#x3C; endl;
cout &#x3C;&#x3C; fitstName.length() &#x3C;&#x3C; endl;

// const char* : 해당 문자열을 변경할 수 없다. 
// .c_str() : 해당 string 이 가지고 있는 문자 배열의 시작 주소를 가리키는 포인를 반환
string line;
cin >> line;
const char* cLine = line.c_str();    // const 를 사용하는 이유 : 
                                     // 내가 해당 문자열의 메모리를 관리하고 있는데, 다른 곳에서 해당 메모리에 관여하는 것은 말이 안된다.
// C 와 같이 인덱스에 접근할 수 있다. 
string firstName = "POKE";
char letter = firstName[1];

firstName[2] = 'P';

// string 속의 문자에 접근하기
char letter = firstName.at(1);
fistName.at(2) = 'P';

// 한 줄 읽기
// -> 다음 조건을 만족할 때까지 스트림에서 문자들을 읽는다. 
//    1. eof 를 만날때 
//    2. 구분 문자를 만날때까지 (구분문자는 버려진다.)
string mailHeader;
getline(cin, mailHeader);
getline(cin, mailHeader, '@');
</code></pre>

#### 그래서 왜 string 이 더 낫다는 거죠?

* 문자 배열 길이에 대해서 고민할 필요가 없다.
*   그런데 어떻게?

    * string 을 선언하게 되면 기본적으로 16바이트의 용량을 할당받는다.\
      (실제 값은 heap 에 저장)

    <figure><img src="../../../../.gitbook/assets/스크린샷 2023-10-25 21.08.59.png" alt="" width="375"><figcaption></figcaption></figure>

    * 처음 제공되는 16바이트 미만의 값으로 초기화를 하게 되면 해당 메모리에 값을 넣어준다.

    <figure><img src="../../../../.gitbook/assets/스크린샷 2023-10-25 21.10.16 (1).png" alt="" width="375"><figcaption></figcaption></figure>

    * 이전에 넣어주었던 값에 추가로 남는 메모리보다 많은 값을 넣어주게 되면, 새로운 용량을 할당 받고 포인터를 변경해준다. 그 후 값을 추가해준다. (heap)
    * 그리고 이전에 사용했던 메모리는 반환한다.

    <figure><img src="../../../../.gitbook/assets/스크린샷 2023-10-25 21.12.22.png" alt="" width="375"><figcaption></figcaption></figure>

#### string 의 메모리 관리 방법은 상당히 복잡하다..

*   힙 메모리 할당은 원래 느리다..

    \-> 스택은 컴파일 할 때 .exe 파일에 들어간 공간이기 때문에, 스택 포인터만 바꾸어도 되지만, 힙은 운영체제가 가지고 있는 모든 메모리를 확인한 후 할당을 하기 때문에, 느릴 수밖에 없다..
* 메모리 파편화의 문제..\
  \-> 위와 같은 방법으로 메모리를 할당 받게 되면, 메모리의 구멍이 생기게 되는데 메모리를 할당 받을 때는 연속된 메모리를 받아야 하기 때문에 실제 메모리를 할당 받지 못하는 메모리 파편화가 생긴다
* 때문에, c++ 를 쓰는 업계가 어디인지 생각해야 한다.\
  \-> 이러한 문제 때문에, char\[] 과 같은 방식을 아직까지 많이 사용한다

#### c\_str() 을 다시한번 생각해보자

```cpp
string line = "POPE";
const char* cLine = line.c_str();    // 포인터는 변경이 가능하지만, 해당 string 값은 변경이 불가하다.
// char* const cLine          : 포인터가 변경이 불가하다.                  
// const char* const cLine    : 포인터와 string 값 모두 변경 불가하다. 
```

#### String 클래스를 직접 구현해보자

```cpp
#include "MyString.hpp"

using namespace std;

MyString::MyString()
{
    str_ = nullptr;
    size_ = 0;
}

MyString::MyString(const char* init)
{
    // 크기(size_) 결정
    while(init[size_] != '\0')
        size_++;

    // 메모리 할당
    str_ = new char[size_];
    
    // 데이터 복사
    memcpy(str_, init, size_);
}

MyString::MyString(const MyString& str)
{
    size_ = str.size_;
    str_ = new char[size_];
    memcpy(str_, str.str_, size_);
}

MyString::~MyString()
{
    // 메모리 해제
    if(str_ != nullptr)
    {
        delete[] str_;
        str_ = nullptr;
        size_ = 0;
    }
}

bool MyString::IsEmpty()
{
    return Length() == 0;
}

bool MyString::IsEqual(const MyString& str)
{
    for(int i=0; i<str.size_; i++)
    {
        if(str_[i] != str.str_[i])
            return false;
    }

    return true;
}

int MyString::Length()
{
    return size_;
}

void MyString::Resize(int new_size)
{
    char* new_str = new char[new_size];
    
    for(int i=0; i<(size_ > new_size ? new_size:size_); i++)
        new_str[i] = str_[i];
    
    delete str_;
    str_ = new_str;
    size_ = new_size;
}

MyString MyString::Substr(int start, int num)
{
    MyString temp;
    temp.size_ = num;
    temp.str_ = new char[temp.size_];
    
    for(int i=start; i<start + num; i++)
    {
        temp.str_[i-start] = str_[i];
    }

    return temp;
}

MyString MyString::Concat(MyString app_str)
{
    MyString temp;

    temp.size_ = size_ + app_str.size_;
    temp.str_ = new char[temp.size_];
    
    memcpy(temp.str_, str_, size_);
    memcpy(temp.str_ + size_, app_str.str_, app_str.size_);

    return temp;
}

MyString MyString::Insert(MyString t, int start)
{
    assert(start >= 0);
    assert(start <= this->size_);

    MyString temp;

    temp.size_ = size_ + t.size_;
    temp.str_ = new char[temp.Length()];

    memcpy(temp.str_, str_, start);
    memcpy(temp.str_ + start, t.str_, t.size_);
    memcpy(temp.str_ + start + t.size_, str_ + start, size_ - start);

    return temp;
}

int MyString::Find(MyString pat)
{
    //TODO:
    for(int i=0; i<Length() - pat.Length(); i++)
    {
        for(int j=0; j<pat.Length(); j++)
        {
            if(str_[i+j] != pat.str_[j])
                break;
            
            if(j == pat.Length()-1)
                return i;
        }
    }

    return -1;
}

void MyString::Print()
{
    for (int i = 0; i < size_; i++)
        cout << str_[i];
    cout << endl;
}


```
