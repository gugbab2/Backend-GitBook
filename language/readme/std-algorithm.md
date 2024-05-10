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
