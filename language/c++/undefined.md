# 파일 입출력

## 1. 파일 입출력 기본

#### 파일 입출력 또한 stream 이다! (stream : 입출력에서 사용하는 것들을 지칭하는 추상적인 개념)

#### 파일 입출력에 관련된 클래스를 알아보자 \<fstream>

* **ifstream**
  * 파일을 입력
* **ofstream**
  * 파일을 출력
* **fstream**
  * 파일을 입력 및 출력

\=> 파일 스트림에 <<, >>, 조정자(manipulate) 등도 쓸 수 있다.

#### 파일 IO 이전에, 파일을 어떻게 열 수 있을까?

```cpp
// 읽기 전용으로 파일을 오픈
ifstream fin;
fin.open("helloWorld.txt");

// 쓰기 전용으로 파일을 오픈(파일이 없다면 만든다)
ofstream fout;
fout.open("helloWorld.txt");

// 읽기 쓰기 법용으로 파일을 오픈
fstream fs;
fs.open("helloWorld.txt");
```

#### 파일을 모두 사용하고 어떻게 닫을 수 있을까?

```cpp
ifstream fin;

// ...

// fin 은 스코프를 벗어나게 되면 기본적으로 fin.close() 를 호출하게된다. 
// 하지만, 의도적으로 일찍 닫거나 코드를 이쁘게 만들기 위해서 사용한다. 
fin.close();
```

#### 스트림 상태 확인하기!

```cpp
fstream fs; 
fs.open("helloWorld.txt");

// fs.is_open() : 스트림이 열려 있는지 확인한다.
if(fs.is_open())
{
    // ...
}
```

#### 파일에서 문자 하나씩 읽기&#x20;

```cpp
ifstream fin;
fin.open("helloWorld.txt");

char character;
while(true)
{
    fin.get(character);
    if(fin.fail())    // fin.fail() : 읽는 것을 실패했다면 EOF 와 다를바가 없다. 
    {
        break;
    }    
    cout << character;
}

fin.close();
```

#### 다른 스트림에서도 사용되는 get(), getline(), >> 추상화!&#x20;

```cpp
fin.get(character);

fin.getline(firstName, 20);     // 파일에서 문자 20개를 읽음

string line;
getline(fin, line);             // 파일에서 한 줄을 읽음    

fin >> word;                    // 파일에서 한 단어를 읽음
```

## 2. 파일에서 읽기

#### 파일에서 한줄씩 읽기 코드

```cpp
ifstream fin;
fin.open("helloWorld.txt");

string line;
while(!fin.eof())
{
    getline(fin, line);
    cou << line << endl;
}

fin.close();
```

#### ex1 한줄씩 읽어보자&#x20;

`Hello,`\
`I love C++`\
`C++ is the best computer language.`

1. H 에서 포인터가 시작하며, 'Hello,' 까지 읽고 'I' 로 포인터가 이동한다.&#x20;
2. 'Hello,' 를 출력한다.
3. 'I love C++' 를 읽고 'C' 로 포인터가 이동한다.&#x20;
4. 'I love C++' 를 출력한다.&#x20;
5. 'C++ is the best computer language.' 를 읽고 마지막에 EOF 로 포인터가 이동한다.&#x20;
6. 'C++ is the best computer language.' 를 출력하고 EOF 읽고 읽기를 종료하게 된다.&#x20;

#### ex2. 아무것도 없는 글자를 읽어보자

&#x20;

1. 포인터가 EOF 로 이동하게 된다.&#x20;
2. 한줄을 출력하고, EOF 를 읽은 후 읽기를 종료하게 된다. \
   \-> 빈 파일을 읽지만, 한줄이 출력된 것을 볼 수 있다.&#x20;

```cpp
ifstream fin;
fin.open("helloWorld.txt");

string name; 
int balance;
while(!fin.eof())
{
    fin >> name >> balance
    cout << name << ": $" << balance << endl;
}

fin.close();
```

#### ex3. 한글자 씩 읽어보자

`Pope 12000`

1. 'P' 에서 포인터가 시작하면 'Pope ' 까지 읽고 포인터가 1로 이동한다. \
   \-> **'Pope' 하고서 ' ' 까지는 읽는것이 포인뜨!**
2. 'Pope' 를 출력한다.
3. 12000 을 읽고 마지막에 EOF 로 포인터가 이동한다.&#x20;
4. 12000 을 출력하고 EOF 읽고 읽기를 종료하게 된다.&#x20;

```cpp
ifstream fin;
fin.open("helloWorld.txt");
 
int number;
while(!fin.eof())
{
    fin >> number
    cout << number << endl;
}

fin.close();
```

#### ex4. 숫자들과 뉴라인이 있을 때!

`100 200 300`\
&#x20;

1. 1에서 포인터가 시작하고 100까지 읽고 포인터가 2로 이동한다.&#x20;
2. 100을 출력한다.
3. 200까지 읽고 포인터가 3으로 이동한다.&#x20;
4. 200을 출력한다.&#x20;
5. 300까지 읽고 \n 로 포인터가 이동한다.&#x20;
6. 300을 출력한다.
7. EOF 로 포인터가 이동한다.&#x20;
8. 300을 출력한다. \
   \-> failbit가 true 로 변하고, number 에 들어있던 값을 다시 호출하게 된다.&#x20;
9. EOF 읽고 읽기를 종료하게 된다. &#x20;

* **다음과 같은 코드로 변경할 때 ex4. 문제를 해결할 수 있다.**&#x20;

```cpp
ifstream fin;
fin.open("helloWorld.txt");
 
int number;
while(!fin.eof())
{
    fin >> number
    
    if(!fin.fail())
        cout << number << endl;
}

fin.close();
```

#### ex05. 숫자들과 문자가 있을 때!

`100 C++ 300`

1. 1에서 포인터가 시작하고 100까지 읽고 포인터가 C로 이동한다.&#x20;
2. 100을 출력한다.
3. C 를 읽으려하지만 정수가 아니기에 읽을 수 없다.(failbit)
4. 100을 출력한다.&#x20;
5. C 를 읽으려하지만 정수가 아니기에 읽을 수 없다.
6. 무한반복 ..

* **다음과 같은 코드로 변경할 때 ex5. 문제를 해결할 수 있다.**&#x20;

```cpp
ifstream fin;
fin.open("helloWorld.txt");
 
int number;
while(!fin.eof())
{
    fin >> number
    
    if(fin.fail())
    {
       fin.clear();
       fin.ignore(LLONG_MAX, ' '); 
    }
    else
    {
       cout << number << endl;
    }   
}

fin.close();
```

* **만약 `100   C++ 300` 과 같이 ' ' 가 아닌 tap 을 사용할 때는 아래와 같이 코드를 작성하자!**

```cpp
ifstream fin;
fin.open("helloWorld.txt");
 
int number;
string trash;
while(!fin.eof())
{
    fin >> number
    
    if(!fin.fail())
    {
       cout << number << endl;
       coutinue;
    }
    
    if(fin.eof())
    {
       break;
    }   
    fin.clear();
    fin >> trash;
}

fin.close();
```

* 만약 다음 코드에서 `fin.clear()` 와 `fin >> trash` 코드의 순서가 바뀐다면 어떻게 될까?\
  \-> **fin 의 상태가 failbit 이기 때문에, 문자를 읽지 못하고 무한루프가 돌게 된다..**

```cpp
ifstream fin;
fin.open("helloWorld.txt");
 
int number;
string trash;
while(!fin.eof())
{
    fin >> number
    
    if(!fin.fail())
    {
       cout << number << endl;
       coutinue;
    }
    
    if(fin.eof())
    {
       break;
    }   
    
    // 무한루프 지점 ..
    fin >> trash;
    fin.clear();
}

fin.close();
```

## 3. 파일에서 쓰기

