# 입력(Input)

## 1. 입력의 기본

#### c++ 은 다음과 같은 형식으로 입력을 받게 된다.&#x20;

```cpp
char firstName[20];
cin >> firstName;
```

* c 의 입력 메서드인 scanf 는 경계검사를 하지 않기 때문에, 위험하다.\
  (문자열은 마지막을 확인하기 위해서 \o을 넣어준다)&#x20;
* 마찬가지로 c++ 에서도 같은 문제를 가지고 있다.&#x20;
  * char\[] == char\*&#x20;
  * cin 은 char 배열의 길이를 모른다..
  * 메모리 할당 이슈 = 내가 할당한 메모리보다 더 많이 사용하게 된다. \
    \-> 나는 4만큼의 메모리를 할당했는데, 실제로는 \o까지 해서 5개의 메모리를 사용하게 된다.
* 때문에, 메모리를 더 사용하는 문제를 막기 위해서 다음과 같은 예방이 필요하다.&#x20;

```cpp
char firstName[4];
cin >> setw(3) >> firstName;
```

#### 입력스트림에서 무엇인가를 읽어왔는데, 성공/실패의 상태를 어떻게 알 수 있을까?

* c++ 에서는 기본적으로 스트림의 상태를 저장한다.&#x20;
* 다음 코드를 보자.

```cpp
string line;
cin >> line;
if(!cin.eof())    // eof : end of file(eof 를 만나면 더이상 읽을 것이 없기에 포기한다)
{
    
}

if(!cin.fail())
{
    
}

if(cin.good())
{

}

if(cin.bad())
{

}
```

* 다음 코드에서 입력값에 따라 어떠한 결과가 나오는지 확인해보자.

```cpp
int number;
cin >> number;

// 입력값 : 456abc     -> number 의 자료형이 int 형이기 때문에, 456까지 읽고 포인터는 멈추어져 있는다. 
// eofbit : unset
// failbit : unset

// 입력값 : 456        -> 콘솔상에서 입력을 받는다면 마지막에 \n 가 있기 때문에, eof 가 unset 이지만, 
// eofbit : (un)set   -> 텍스트 파일을 통해서 입력을 받는다면, \n 가 없기 때문에, eof 가 set 이다.
// failbit : unset

// 입력값 : abc
// eofbit : unset
// failbit : set

// 입력값 : eof
// eofbit : set
// failbit : set
```

#### 스트림에 담겨있는 입력 버리기

* clear() : 스트림을 좋은 상태(good state) 로 돌려 줌 \
  \-> 좋은 상태로 돌려주는 것이지, 현재 꼬여버린 스트림 자체를 날리는 것은 아니다.&#x20;
* ignore() : 스트림의 지정한 수만큼 입력을 버림 \
  \-> 아래 예제들은 파일 끝(eof)에 도달하거나, 지정한 수만큼 문자를 버리면 멈춤]
* get()&#x20;
  * 뉴라인 문자를 만나기 직전까지의 모든 문자를 가져옴
  * 뉴라인 문자는 입력 스트림에 남아있음
* getline()
  * 뉴라인 문자를 만나기 직전까지의 모든 문자를 가져옴
  * 뉴라인 문자는 입력 스트림에서 버림

<pre class="language-cpp"><code class="lang-cpp">cin.clear();

cin.ignore();      // 문자 1개를 버림
cin.ignore(10);    // 문자 10개를 버림

cin.ignore(10, '\n');        // 문자 10개를 버림. 단, 그 전에 \n 문자를 만나면 곧바로 멈춤
cin.ignore(LLONG_MAX, '\n')  // 최대 문자 수를 버림. 단, 그 전에 \n 문자를 만나면 곧바로 멈춤

char firstName[100];
<strong>cin.get(firstName, 100);      // 99개 문자를 가져오거나 뉴라인 문자가 나올 때까지의 문자를 firstName 으로 가져온다.
</strong>cin.get(firstName, 100, '#'); // 99개 문자를 가져오거나 # 문자가 나올 때까지의 문자를 firstName 으로 가져온다.  

cin.getline(firstName, 100);      // 99개 문자를 가져오거나 뉴라인 문자가 나올 때까지의 문자를 firstName 으로 가져온다.
cin.getline(firstName, 100, '#'); // 99개 문자를 가져오거나 # 문자가 나올 때까지의 문자를 firstName 으로 가져온다.  
</code></pre>

#### 다음 코드를 통해서 입력에 대한 내용을 정리해보자

```cpp
#include "ReverseInputString.h"
#include <iostream>

using namespace std;

namespace samples
{
	void ReverseInputStringExample()
	{
		cout << "+------------------------------+" << endl;
		cout << "|    Reverse String Example    |" << endl;
		cout << "+------------------------------+" << endl;

		const int LINE_SIZE = 512;
		char line[LINE_SIZE];

		cout << "Please enter a string to reverse" << endl
			<< "or EOF to quit: ";

		cin.getline(line, LINE_SIZE);
		if (cin.fail())
		{
			cin.clear();
			return;
		}

		char* p = line;				// 문자열 첫번째 포인터 
		char* q = line + strlen(line) - 1;	// 문자열 마지막 포인터(0부터 시작하기에 -1)
		while (p < q)
		{
			char temp = *p;
			*p = *q;
			*q = temp;

			++p;
			--q;
		}
		
		cout << "Reversed string: " << line << endl;
	}
}
```
