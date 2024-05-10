# std::algorithm

## std::find

```cpp
template <class InputIterator, class T>
InputIterator find(InputIterator first, InputIterator last, const T& val);
```

* 범위 안(`first` 부터 `last` 전 까지) 의 원소들 중 val 과 일치하는 첫 번쨰 원소를 가리키는 반복자를 리턴한다. 만일 일치하는 원소를 찾지 못할 경우 `last` 를 리턴한다.
* 이 함수는 원소를 비교할 때 `operator==` 을 사용한다.\
  \-> T 타입이 `==` 연산자 오버로딩이 안되어 있다면 컴파일 에러가 발생한다.&#x20;
* 참고로 이 함수는 `string` 의 `find` 함수와 다르다.&#x20;

#### 매개변수&#x20;

* `first, last` : 원소들의 시작과 끝을 가리키는 반복자들. 이 때  확인하는 범위에 `first` 가 가리키는 원소는 포함되지만, `last` 가 가리키는 원소는 포함되지 않는다.\
  \-> begin(), end() 를 생각하자.
* `val` : 비교할 값. 이 떄 **`val` 타입 `T` 의 경우 `operator==` 가 정의되어 있어야만 한다.**

#### 리턴값&#x20;

* 첫 번째로 일치하는 원소를 가리키는 반복자를 리턴한다. \
  \-> **일치하는 원소가 없을 경우 last 가 리턴된다.**&#x20;

#### 구현 예시

```cpp
template<class InputIt, class T>
constexpr InputIt find(InputIt first, InputIt last, const T& value) {
    for(; first != last; ++first){
        if(*first == value)
            return first;
    }
    return last;
}
```

#### 실행  예제&#x20;

```cpp
#include <algorithm>
#include <iostream>
#include <vector>

int main() {
    
    // array 예제
    int myInts[] = {10, 20, 30, 40};
    int* p;
    
    p = std::find(myInts, myInts + 4, 30);
    if(p != myInts + 4)
        std::cout << "Element found in myInts: " << *p << endl;
    else 
        std::cout << "Element not found in myInts: " << *p << endl;
    
    // vector 예제 
    std::vecotr<int> myVector(myInts, myInts + 4);
    std::vector<int>::iterator it;
    
    it = find(myVector.begin(), myVector.end(), 30);
    if(it != myVector.end())
        std::cout << "Element found in myInts: " << *it << endl;
    else 
        std::cout << "Element not found in myInts: " << *it << endl;
        
    
    return 0;
}
```

## std::copy

```cpp
// 참고로 C++ 20 부터 모두 constexpr 함수 이다. 
template <class InputIt, class OutputIt>
OutputIt copy(InputIt first, InputIt last, OutputIt d_first);

template <class InputIt, class OutputIt, class UnaryPredicate>
Output copy_if(InputIt first, InputIt last, OutputIt d_first, UnaryPredicate pred);
```

* [copy](https://modoocode.com/305) 함수는 `first` 부터 `last` 전 까지의 모든 원소들을 `d_first` 부터 시작하는 곳에 복사한다. [copy\_if](https://modoocode.com/305) 의 경우 `pred` 가 `true` 를 리턴하는 원소만 복사하게 된다. 이 때 원소들의 상대적인 순서는 유지된다.

#### 매개변수&#x20;

* `first, last` : 원소들의 범위를 나타내는 반복자. (각각 첫 번째 원소와 마지막 원소 바로 다음을 가리킴)
* `d_first` : 복사한 원소들을 저장할 곳의 시작점을 나타내는 반복자
* `pred` : 조건을 확인하는 함수. 이 때 반복자들이 가리키는 원소 `v` 에 대해서 `pred(v)` 는 반드시 `bool` 로 변환될 수 있어야만 한다. 에를 들어서 `bool Pred(const Elem& v)` 와 같이 말이다.

#### 리턴값

* 복사된 곳의 마지막 원소 바로 다음을 가리키는 반복자&#x20;

#### 구현예시&#x20;

```cpp
template <class InputIt, class OutputIt>
OutputIt copy(InputIt first, InputIt last, OutputIt d_first) {
  
  // first 와 last 의 반복자가 같지 않다면 
  // d_first 의 반복자부터 복사를 시작하고 마지막 반복자를 리턴
  while (first != last) {
    *d_first++ = *first++;
  }
  return d_first;
}

template <class InputIt, class OutputIt, class UnaryPredicate>
OutputIt copy_if(InputIt first, InputIt last, OutputIt d_first,
                 UnaryPredicate pred) {
  // first 와 last 의 반복자가 같지 않고
  // pred 함수에서 참을 반환하는 것들만 
  // d_first 의 반복자부터 복사를 시작하고 마지막 반복자를 리턴
  while (first != last) {
    if (pred(*first)) *d_first++ = *first;
    first++;
  }
  return d_first;
}
```

#### 실행 예제&#x20;

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <iterator>
#include <numeric>
 
int main()
{
    // 크기가 10 인 from_vector 생성 
    std::vector<int> from_vector(10);
    // 0 부터 9 까지 원소들을 from_vector 에 집어넣는다.
    std::iota(from_vector.begin(), from_vector.end(), 0);   // from_vector : 0 1 2 3 4 5 6 7 8 9
 
    std::vector<int> to_vector;
    // from_vector 의 원소들을 to_vector 에 복사한다.
    std::copy(from_vector.begin(), from_vector.end(),
              std::back_inserter(to_vector);        // to_vector : 0 1 2 3 4 5 6 7 8 9
    std::cout << "to_vector contains: ";
 
    // to_vector 의 원소들을 표준 출력 객체(std::cout) 에 전달한다. 
    std::copy(to_vector.begin(), to_vector.end(),
              std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';
}
```

## std::transform

```cpp
template <class InputIt, class OutputIt, class UnaryOperation>
OutputIt transform(InputIt first1, InputIt last1, OutputIt d_first,
                   UnaryOperation unary_op);  // (1)
                   
template <class InputIt1, class InputIt2, class OutputIt, class BinaryOperation>
OutputIt transform(InputIt1 first1, InputIt1 last1, InputIt2 first2,
                   OutputIt d_first, BinaryOperation binary_op);  // (2)
                   
template <class ExecutionPolicy, class ForwardIt1, class ForwardIt2,
          class ForwardIt3, class BinaryOperation>
ForwardIt3 transform(ExecutionPolicy&& policy, ForwardIt1 first1,
                     ForwardIt1 last1, ForwardIt2 first2, ForwardIt3 d_first,
                     BinaryOperation binary_op);  // (3)
```

* (1) 단항함수 `unary_op` 이 `first1` 부터 `last1` 전까지 원소들을 인자로 전달하여 실행한다.&#x20;
* (2) 이항 함수 `binary_op` 이 `first1` 부터 `last1` 전까지 원소들과 `first2` 부터의 원소들을 쌍으로 전달해서 실행합니다. 예를 들어서 `first1` 부터 `last1` 이 {1,2,3} 이고 `first2` 부터의 원소가 {4,5,6} 이면 `binary_op` 에는 `bianry_op(1,4), binary_op(2,5), binary_op(3,6)` 이 실행되는 셈입니다.
* (3) 어떠한 방식으로 실행 시킬지. 지정할 수 있습니다. `ExecutionPolicy` 참조.

#### 매개변수&#x20;

* `first1, last1` `: transform` 을 적용할 원소들을 가리키는 범위
* `first2` : 두 번째 인자들의 시작
* `d_first` : 결과를 저장할 범위. `first1` 이나 `first2` 와 동일해도 된다.
* `policy` : 어떠한 방식으로 실행 시킬지. `ExecutionPolicy` 참조.
* `unary_op` : 원소들을 전달할 단항 함수. 해당 함수는 다음과 같이 생겨야 합니다.

```cpp
Ret unary(const Type &a);
```

참고로 함수의 인자로 굳이 `const&` 일 필요는 없다. `Type` 는 `InputIt` 의 역참조된 타입과 같거나 `Type` 로 변환될 수 있어야 하며, [Ret](https://modoocode.com/ret) 의 경우 `OutputIt` 의 역참조 된 타입이거나, [Ret](https://modoocode.com/ret) 에 변환되서 대입할 수 있어야 한다.

* `binary_op` : 원소를 두 개 받는 이항 함수로 해당 함수는 아래와 같이 생겼다.&#x20;

```cpp
Ret binary(const Type1 &a, const Type2 &b);
```

함수의 인자로 굳이 `const&` 가 아니여도 된다. 위의 경우와 비슷하게 `InputIt1` 과 `InputIt2` 의 역참조 타입이 `Type1` 과 `Type2` 와 각각 같거나 해당으로 변환될 수 있어야만 한다.&#x20;

#### 구현 예시&#x20;

```cpp
template <class InputIt, class OutputIt, class UnaryOperation>
OutputIt transform(InputIt first1, InputIt last1, OutputIt d_first,
                   UnaryOperation unary_op) {
  while (first1 != last1) {
    *d_first++ = unary_op(*first1++);
  }
  return d_first;
}

template <class InputIt1, class InputIt2, class OutputIt, class BinaryOperation>
OutputIt transform(InputIt1 first1, InputIt1 last1, InputIt2 first2,
                   OutputIt d_first, BinaryOperation binary_op) {
  while (first1 != last1) {
    *d_first++ = binary_op(*first1++, *first2++);
  }
  return d_first;
}
```

#### 실행 예제

```cpp
// 아래 예제는 문자열을 대문자로 바꾸거나, 해당 아스키 값으로 바꿉니다.
#include <algorithm>
#include <cctype>
#include <iostream>
#include <string>
#include <vector>

int main() {
  std::string s("hello");
  std::transform(
    s.begin(), s.end(), s.begin(),
    [](unsigned char c) -> unsigned char { return std::toupper(c); });  // HELLO

  std::vector<std::size_t> ordinals;
  std::transform(s.begin(), s.end(), std::back_inserter(ordinals),
                 [](unsigned char c) -> std::size_t { return c; }); // 72 69 76 76 79

  std::cout << s << ':';
  for (auto ord : ordinals) {
    std::cout << ' ' << ord;
  }
}
```
