# std::vector

## vector&#x20;

* vector 는 헤더파일\<vector> 에 정의되어 있는 순차 컨테이너의 종류로, 각각의 원소들이 선형적으로 배열되어 있다.&#x20;
* vector 는 동적 배열로 구현되는데, 보통의 배열처럼 vecor 도 각각의 원소들이 메모리상에 연속적으로 존재하게 된다. \
  \-> 이 때문에, vector 의 원소를 참조할 때 반복자(iterator) 를 이용해 순차적으로 참조가 가능하다.&#x20;
* **하지만 보통의 배열과는 달리 vector 는 스스로 공간을 할당하고, 크기를 확장할 수 있고, 또 줄일 수 있다.** \
  **-> 가장 큰 장점**\
  **-> vector 의 크기가 증가될 떄는 2배씩 증가된다.**&#x20;
* vecotr 의 장점&#x20;
  * 각각의 원소를 index 값으로 참조 가능하다. (포인터와 비슷하게 사용 가능) \
    **-> 조회 시 O(1)**&#x20;
  * 원소들을 임의의 순서로 접근할 수 있다. (메모리 연속적)
  * 벡터 끝에 새로운 원소를 추가하거나 제거할 때 속도적인 이점을 갖는다. \
    \-**> 배열 끝에 삽입, 삭제, 수정 시 O(1)**

## vector 생성자

```cpp
// 디폴트 생성자로 빈 vector 를 생성하는 생성자
explict vector(const Allocator& = Allocator());
// 값이 T 인 원소 n 개를 가지는 vector 를 생성하는 생성자
explict vector(size_type n, const T& value = T(), const Allocator& = Allocator());

// first 부터 last 번째 원소까지 순회하며, 각각의 원소들을 복사하는 생성자
template <class InputIterator>
vector(InputIterator first, InputIterator last, const Allocator& = Allocator());
// x 와 동일한 원소를 가지는 복사생성자
vector(const vector<T, Allocator>& x);
```

> #### 암시적 형변환이 생성자에서 일어나는  경우
>
> * 아래의 경우 개체를 생성자 호출의 형태로 생성하지는 않았지만, 컴파일러에 의해서 암시적 형변환이 일어난다. (컴파일러에 의해서 `Number num(42);` 의 형태로 변경된다)\
>   \-> 매개변수를 받는 생성자가 없다면, 형변환이 일어나지 않는다.&#x20;
>
> ```cpp
> #include <iostream>
>
> class Number {
> private:
>     int value;
> public:
>     // 정수형을 받는 생성자
>     Number(int val) : value(val) {}
>
>     // 객체의 값을 출력하는 멤버 함수
>     void print() const {
>         std::cout << "Value: " << value << std::endl;
>     }
> };
>
> int main() {
>     // 정수형 값으로 Number 객체를 생성
>     Number num = 42;
>
>     // 생성된 객체의 값을 출력
>     num.print();
>
>     return 0;
> }
> ```
>
> ####
>
> #### explict  를 통한 암시적 형변환 불가 (안정적인 코드 사용)
>
> *   #### explict  키워드를 사용하게 되면, 생성자에서 암시적 형변환을 허용하지 않아, `Number num = 42;` 코드는 컴파일 에러가 발생한다.&#x20;
>
>     \-> 반드시 생성자 호출의 형태로 사용해야 한다.&#x20;
>
> ```cpp
> #include <iostream>
>
> class Number {
> private:
>     int value;
> public:
>     // explicit 키워드 추가
>     explicit Number(int val) : value(val) {}
>
>     // 객체의 값을 출력하는 멤버 함수
>     void print() const {
>         std::cout << "Value: " << value << std::endl;
>     }
> };
>
> int main() {
>     // 정수형 값으로 Number 객체를 생성
>     // Number num = 42; // 컴파일 오류 발생
>     Number num(42); // 명시적으로 생성자 호출
>
>     // 생성된 객체의 값을 출력
>     num.print();
>
>     return 0;
> }
> ```

## vector::assign

```cpp
template <class InputIterator>
void assign(InputIterator first, InputIterator last);
void assign(size_type n, const T& u);
```

* **vector 에 새로운 내용을 집어 넣는다. vector 원본 개체의 원소들을 모두 제거하고, 인자로 받은 새로운 내용을 집어 넣는다.**&#x20;
* 첫번째 형태의 함수는 first 부터 last 까지의 있는 원소들의 내용이 백터에 들어가게 된다.&#x20;
* 두번째 형태의 함수는 원래 내용을 다 지우고, u 를 n 개 가진 vector 로 만든다.&#x20;

#### 실행 예제

```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    vector<int> first;
    vector<int> second;
    vector<int> third;
    
    first.assign(7, 100);    // 100 을 7번 반복해서 집어넣는다. 
    
    vector<int>::iterator it;
    it = first.begin() + 1;
    
    second.assign(it, first.end() - 1);    //first 의 처음와 끝을 제외한 원소들 

    int myInts[] = {1776, 7, 4};
    third.assign(myInts, myInts + 3);    // 배열로 부터 받는다. 
    
    cout << "Size of first: " << int(first.size()) << endl;
    cout << "Size of second: " << int(second.size()) << endl;
    cout << "Size of third: " << int(third.size()) << endl;
    
    return 0;
}
```

## vector::insert

```cpp
iterator insert(iterator position, const T& x);
void insert(iterator position, size_type n, const T& x);
template<class InputIterator>
void insert(iterator position, InputIterator first, InputIterator last);
```

* vector 에 원소를 추가한다. 특정 위치에 원소를 추가함으로써 vector 가 확장된다.&#x20;
* 이 함수는 vector 의 크기를 효과적으로 증가시키는데, 만일  새로운 vector 의 size 가 capacity 를 초과하게 된다면, 재할당을 하게 된다. \
  \-> 이 때 이전에 사용하고 있던 반복자(iterator) 와 참조(reference) 는 유효하지 않다. (주의하자..)
* vector 는 배열과 같이 연속된 메모리의 형태로 저장되기 때문에, vector 의 끝이 아닌 임의의 위치에 원소를 삽입하게 되면, 메모리 복사가 일어난다. (메모리 복사는 효율적이지 못하다..)

#### 매개변수&#x20;

* `position` : 새로운 원소가 추가될 위치로, 반복자 타입이다.&#x20;
* `x` : 삽입할 원소의 값으로 T 타입이다.&#x20;
* `n` : 삽입할 원소의 수&#x20;
* `first, last` : 특정 범위의 원소들을 지칭하는 위치로, 반복자 타입이다.&#x20;

#### 리턴값

* iterator : 가장 처음 원소를 가리키는 반복자 타입을 리턴한다.&#x20;

#### 실행 예제&#x20;

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
  vector<int> myvector(3, 100); // 100 100 100
  vector<int>::iterator it;

  it = myvector.begin();
  it = myvector.insert(it, 200);    // 200 100 100 100

  myvector.insert(it, 2, 300);  // 300 300 200 100 100 100 

  // "it" no longer valid, get a new one:
  it = myvector.begin();

  vector<int> anothervector(2, 400);
  myvector.insert(it + 2, anothervector.begin(), anothervector.end());  // 300 300 400 400 200 100 100 100

  int myarray[] = {501, 502, 503};
  myvector.insert(myvector.begin(), myarray, myarray + 3);  // 501 502 503 300 300 400 400 200 100 100 100

  cout << "myvector contains:";
  for (it = myvector.begin(); it < myvector.end(); it++) cout << " " << *it;
  cout << endl;

  return 0;
}
```

## vector::push\_back

```cpp
void push_back(const T& x);
```

* vector 끝에 원소를 추가한다.&#x20;
* 이 함수는 vector 의 크기를 효과적으로 증가시키는데, 만일  새로운 vector 의 size 가 capacity 를 초과하게 된다면, 재할당을 하게 된다. \
  \-> 이 때 이전에 사용하고 있던 반복자(iterator) 와 참조(reference) 는 유효하지 않다. (주의하자..)

#### 실행 예제&#x20;

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
  vector<int> myvector;
  int myint;

  cout << "Please enter some integers (enter 0 to end):\n";

  cin >> myint;
  
  while(myint){
    myvector.push_back(myint);
    if(myvector[0]) break; 
  }
  
  cout << "myvector stores " << (int)myvector.size() << " numbers.\n";

  return 0;
}
```
