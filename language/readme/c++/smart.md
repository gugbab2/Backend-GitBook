# 스마트(smart) 포인터

## RAII(Resource Acquisition Is Initialization)

* C++ 의 중요한 프로그래밍 원칙 중 하나로 **자원의 획득과 해제를 개체의 생명주기와 연결하는 방법이다.** → 이 원칙은 자원 누수를 방지하고, 예외 안정성을 향상시키기 위해서 사용된다.

### 기본 개념

* **자원의 획득은 개체의 초기화와 함께 이루어진다.**
  * 자원을 필요로 하는 개체는 생성자에서 자원을 획득한다. → 파일 열기, 메모리 할당
* **자원의 해제는 개체의 소멸과 함께 이루어진다.**
  * 개체가 더 이상 필요하지 않다면, 소멸자에서 자원을 자동으로 해제한다. → 파일 닫기, 메모리 해제

```cpp
#include <iostream>
#include <fstream>

class FileRAII {
public:
    // 생성자에서 파일 열기
    FileRAII(const std::string& filename)
        : file(filename) {
        if (!file.is_open()) {
            throw std::runtime_error("Unable to open file");
        }
    }

    // 소멸자에서 파일 닫기
    ~FileRAII() {
        if (file.is_open()) {
            file.close();
        }
    }

    // 파일에 쓰기
    void write(const std::string& content) {
        if (file.is_open()) {
            file << content;
        }
    }

private:
    std::ofstream file;
};

int main() {
    try {
        FileRAII file("example.txt");
        file.write("Hello, RAII!");
    } catch (const std::exception& e) {
        std::cerr << "Exception: " << e.what() << std::endl;
    }

    // 파일이 자동으로 닫힘
    return 0;
}
```

### 소멸자가 호출되는 시점

* **스택의 할당된 개체** : 해당 개체를 포함하는 스코프를 벗어날 때
* **힙에 할당된 개체** : `delete` 연산자가 호출될 때
* **클래스 멤버 개체** : 클래스 개체 소멸 시
* **전역 개체** : 프로그램 종료 시
* **스마트 포인터를 사용한 개체 관리** : 스마트 포인터 개체가 소멸될 때 내부에서 관리하는 개체의 소멸자가 호출된다.

## 기존 포인터 문제점

* 기존 포인터는 무조건적으로 메모리를 해제해야 한다.

```cpp
#include "Vector.h"

int main()
{
	Vector* myVector = new Vector(10.f, 30.f);
	
	// ...
	
	delete myVector; 
	
	return 0;
}
```

* 스마트 포인터를 사용하면 `delete` 를 직접 호출할 수 필요가 없다.
* **그리고 다른 언어의 가비지 컬렉터보다 빠르다!** → 기존 가비지 컬렉터는 언제 메모리를 해제해주는지 알 수가 없었지만, → **스마트 포인터는 메모리가 쓰이지 않는 순간에 바로 메모리가 해제된다.**

## unique\_ptr (C++11)

* **나 말고 누구도 사용할 수 없는 포인터이다. → 그 누구와도 공유되지 않는다.**
* **때문에, 복사와 대입 연산이 이루어진다면 컴파일 에러가 발생한다.**
* `unique_ptr` 가 범위를 벗어날 때, 포인터는 지워진다. → 범위랑 생명주기를 같이 하겠다는 의미이다. (RAII)

```cpp
#include <memory> 
#include "Vector.h"

int main() 
{
	std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));   // 포인터(*) 부호가 없다.
	
	myVector -> print();   // 포인터처럼 사용
	
	// delete 가 없다. 
	
	return 0;
}

// ...

// 복사와 대입 연산은 컴파일에러를 발생시킨다. 
std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));

std::unique_ptr<Vector> copiedVector1 = myVector;   // compile error
std::unique_ptr<Vector> copiedVector2(myVector);    // compile error 
```

### 유니크 포인터의 사용이 적합한 경우

#### 클래스에서 생성자, 소멸자

* **소멸자에서 개체의 메모리를 해제하지 않아도 된다.** → 해당 클래스 개체가 사용되지 않을 때 멤버 개체의 메모리가 해제된다.

```cpp
// 기존 방법 
// player.h
class player
{
private : 
	Vector* mLocation; 
};

// player.cpp 
player::player (std::string name) 
	:mLocation(new Vector(); 
{
}

player::~player()
{
	delete mLocation;
}

// 유니크 포인터 사용 방식 
// player.h
class player
{
private : 
	std::unique_ptr<Vector> mLocation; 
};

// player.cpp 
player::player (std::string name) 
	:mLocation(new Vector(); 
{
}

// 소멸자가 필요없다. 
```

#### 지역 변수

* 기존에 지역변수로 포인터를 사용했을 때 delete 를 통해서 메모리를 해제해주어야 했지만,
* **unique\_ptr 을 사용하면 자동적으로 포인터의 메모리를 해제한다.**

```cpp
#include <memory> 
#include "Vector.h"

int main() 
{
	std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));   // 포인터(*) 부호가 없다.
	
	myVector -> print();   // 포인터처럼 사용
	
	// delete 가 없다. 
	
	return 0;
}

```

#### STL 벡터에 포인터 저장하기

* STL 벡터에 포인터를 저장하는 경우가 있는데, 이 때 모든 데이터 사용 후 포인터를 해제해주어야 했다.
* **unique\_ptr 를 사용하면 벡터를 사용한 개체 소멸 시 자동적으로 포인터의 메모리를 해제한다.**

![스크린샷 2024-06-11 06.40.25.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/44fb0459-8fc2-4df3-8b4b-adbfce107ce8/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_06.40.25.png)

### unique\_ptr 만들기(C++14)

#### unique\_ptr 이 공유 될 때 문제

* `unique_ptr` 이 공유될 수 없다고 말했지만, 아래의 형태로 사용하게 된다면, 공유가 가능하다.
  * 아래 예제를 살펴보면 `vectorPtr`, `vector`, `anotherVector` 모두 같은 포인터를 갖는 것을 알 수 있다.
  * 마지막에 `anotherVector` 를 `nullptr` 처리 해주는 것을 알 수 있는데, 이 때 `anotherVector` 를 초기화 할 때 사용했던 `vectorPtr` 의 소멸자가 호출되며 메모리를 해제한다.
  * 또 문제점은, `vector` 입장에서는 `vectorPtr` 의 메모리가 해제된 것을 모르기 때문에, 한번 더 메모리를 해제하려 한다.. (난리가 난다)

![스크린샷 2024-06-11 06.48.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/2d4431fb-b442-4430-a946-b595247c9595/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_06.48.56.png)

![스크린샷 2024-06-11 06.51.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/9d0297f6-cd1e-483a-87b9-f09e5039e5c3/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_06.51.56.png)

#### C++14 이후 해결책

* `std::unique_ptr` 을 초기화 시 `std::make_unique` 를 사용해서, 원시 포인터 공유를 막자는 게 포인트이다.
* 둘 이상의 `std::unique_ptr` 이 원시 포인터를 공유하지 못하게 하는게 전부이다.

![스크린샷 2024-06-11 06.55.10.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/b0e9b993-8fdf-401c-872c-91b4abaaa42e/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_06.55.10.png)

![스크린샷 2024-06-11 06.57.56.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/07b5d8f1-e5e2-4d81-8737-cceb2c77ceba/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_06.57.56.png)

#### make\_ptr 을 통해 unique\_ptr 만들기

![스크린샷 2024-06-11 07.00.20.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/52e2d99b-00a6-413b-a3d4-a447548b248f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.00.20.png)

### 유니크 포인터 재설정, 원시 포인터 가져오기, 원시 포인터 소유권 박탈하기

#### 유니크 포인터 재설정

* **유니크 포인터를 재설정하는 방법이다.**
* 매개변수가 있는 `reset` 함수를 호출하는 경우, 이전 메모리를 해제하고, **매개변수 개체로 새로운 메모리를 할당한다.**
* 매개변수가 없는 `reset` 함수를 호출하는 경우, 이전 메모리를 해제하고, **`nullptr` 을 할당한다.**\
  **→ `myVector.reset()` 과 `myVector = nullptr` 은 같은 의미이기 때문에, 사용자의 취향에 따라 사용법이 갈린다.**

```cpp
int main() 
{
	std::unique_ptr<Vector> myVector(new Vector(10.f, 30.f));   // 포인터(*) 부호가 없다.
	
	myVector.reset(new Vector(20.f, 40.f);
	
	myVector.reset(); 
	
	return 0;
}
```

#### 원시 포인터 가져오기

* 예를 들어 함수에서 원시 포인터를 매개 변수로 요청하는 경우 유니크 포인터에서 가지고 있는 원시 포인터가 필요할 수도 있다.
* 이럴 때, **`get` 함수를 통해서 해당 유니크 포인터의 원시 포인터를 추출 할 수 있다. → 꺼내 온 원시 포인터를 delete 해주면 안된다!!**

![스크린샷 2024-06-11 07.10.46.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/a223e392-5262-45d4-bef3-2e304ab1472f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.10.46.png)

#### 원시 포인터 소유권 박탈하기

* 기존 유니크 포인터의 소유권을 `release` 함수를 통해서 전달할 수 있다. → 하지만 이런 식으로 원시 포인터의 소유권을 전달하기 시작하면 코드 복잡도가 상당히 높아진다.. → 권장하는 방식은 아니다…

![스크린샷 2024-06-11 07.13.39.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/488a2198-6571-450e-8960-546672d8d14d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.13.39.png)

* `release` 호출 후 `get` 을 호출하면, `nullptr` 이 반환된다. → **소유권이 박탈되었다!**

![스크린샷 2024-06-11 07.16.10.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/90a15d96-1236-4749-8ce4-a04800106f3d/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.16.10.png)

### 유니크 포인터 소유권 이전하기

* **유니크 포인터의 복사는 불가하고, 소유권 이전만 가능하다! → `std::unique_ptr` 이 소유한 원시 포인터를 그 누구와도 공유하지 않는다. (주소를 복사하지 않는다는 뜻) → 대신 소유권 이전은 가능하다.**
* `std::move` 함수를 통해서 이전 유니크 포인터의 소유권이 이전된다. → **기존 유니크 포인터 내 원시 포인터는 `nullptr` 처리된다. → 메모리 할당과 해제는 이루어지지 않는다.**

![스크린샷 2024-06-11 07.19.55.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/575eae8a-6639-45da-a75c-b4ca3fbde01e/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.19.55.png)

* **`const std::unique_ptr` 은 당연히 소유권 이전이 안된다! → 컴파일 에러 발생!!**

![스크린샷 2024-06-11 07.23.15.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/6fc07148-886c-4a44-8949-1340209c249b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.23.15.png)

* STL 벡터에 유니크 포인터 요소를 추가할 때 `std::move` 를 통해 소유권을 이전해주어야 한다. → **만약 하지 않을 경우 컴파일 에러가 발생한다. → 당연히 원시 포인터에 접근하는 변수는 유일해야 한다!**

![스크린샷 2024-06-11 07.29.02.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/1c69e0a4-44ce-4448-b1ee-68716d4cb4fd/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.29.02.png)

![스크린샷 2024-06-11 07.31.30.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/5c542286-c5dc-4d30-b69e-492dfe951a3b/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_07.31.30.png)

### 유니티 포인터 베스트 프렉티스

* 이제 다들 이걸 쓴다. → **직접 메모리 관리하는 것 만큼 빠르다!**
* RAII(Resource Acquisition Is Initialization) 원칙에 들어맞는다.
  * 개체의 수명과 할당은 연관이 있다.
  * 생성자에서 `new`, 소멸자에서 `delete`
  * std::unique\_ptr 이 이걸 해준다.
* 실수를 방지해주기에, 모든 곳에 사용하자!

## shared\_ptr(C++11)

* 말 그대로 포인터를 공유하기 때문에, 포인터를 누가 해제해주는가에 대한 정의가 필요하다.
* 그 전에, 자동 메모리 관리에서 사용 되는 대표적인 방식 2가지가 있다.
  * 가비지 콜렉션 : Java, C# 에서 지원
  * 참조 카운팅 : Swift, Objective-C 에서 지원

### 가비지 콜렉터

* 보통 트레이싱 가비지 컬렉션을 의미한다.
* 메모리 누수를 막으려는 시도
* 주기적으로 컬렉션을 실행한다.
* 충분한 메모리 여유가 없을 때 컬렉션을 실행한다. → 스케줄에 따라 또는 수동으로도 실행이 가능하다.
* 매 주기마다, GC 는 루트를 확인한다. → 전역변수 → 스택 → 레지스터
* **힙에 있는 개체를 루트를 통해서 접근할 수 있는지 판단한다. → 접근할 수 없다면 가비지로 판단해서 해제한다.**

#### 가비지 컬렉션의 문제점

* 사용 되지 않는 메모리를 즉시 해제하지 않는다.
* GC 가 메모리를 해제해야 하는지 판단하는 동안 애플리케이션이 멈추거나 버벅거릴 수 있다. (stop the world) → **때문에, 리얼타임 서비스에서 GC 기반 언어는 불리할 수 있다!**

### 참조 카운터

* GC 처럼, 개체에 대한 참조가 없을 때 개체가 해제된다.
* **GC 와 차이점은 개체를 사용하지 않는 시점에 바로 해제한다! → 리얼타임 서비스에서 유리하다!**
* 참조 횟수를 사용해서, 특정 개체가 몇번이나 참조되고 있는지 판단이 가능하다. → 특정 개체를 참조하는 개체 A 가 범위를 벗어날 때 참조 횟수가 줄어든다.
* `shared_ptr` 은 참조 카운팅을 자동으로 해준다.

#### 강한 참조

* 강한 참조란 특정 개체를 누군가 참조한다면 특정 개체는 소멸되지 않는 것을 의미한다.
* 강한 참조 횟수가 0이 될 때 해당 개체는 소멸된다.

#### 참조 카운팅의 문제점

* **참조 횟수가 너무 자주 바뀐다.**
  * 멀티 쓰레딩 환경에서 안전하려면, lock 이나 원자적(atomic) 연산이 필요하다. → **멀티 쓰레딩 환경에서 순수한 포인터보다 훨씬 느리다.**
* **순환 참조의 문제가 있다. → 순환 참조를 통해서 메모리 누수가 발생할 수 있다..**
  * 개체 A 가 개체 B 를 참조
  * 개체 B 가 개체 A 를 참조
  * 절대 해제되지 않는다..

### shared\_ptr

#### shared\_ptr 만들기

* 아래 코드와 같이 `shared_ptr` 을 사용할 수 있고, **포인터 선언과 동시에 참조 포인터가 1 이 되었다는 것**을 생각해 볼 수 있다.

```cpp
#include <memory>
#include "Vector.h"

int main()
{
	std::shared_ptr<Vector> vector = std::make_shared<Vector>(10.f, 30.f); 
	// ...
}
```

#### shared\_ptr 특징

*   두 개의 포인터를 소유

    * 데이터(원시 포인터)를 가리키는 포인터
    * 제어 블록을 가리키는 포인터 → 제어 블록 : 참조 카운트를 위한 데이터 공간

    ![스크린샷 2024-06-11 11.26.48.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/1476b1ab-5b72-47be-9105-7fcdd30a3fd6/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.26.48.png)
* 참조 카운트 기반
* 원시 포인터는 어떠한 `std::shared_ptr` 에게도 참조되지 않을 때 소멸된다. → 참조 카운트가 0 일 때

#### shared\_ptr 공유하기

* 대입 연산자를 통해서 `std::shared_ptr<T>` 타입 변수에 대입을 해줄 수 있다.
  * 대입 시 아래와 같이 원시 포인터의 값과
  * 참조 카운트의 값이 공유되는 것을 알 수 있다. → 포인터의 소유권이 공유된다.

![스크린샷 2024-06-11 11.30.39.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/33a434ec-7166-4fa2-9626-ab4b19407c22/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.30.39.png)

#### shared\_ptr 재설정하기

* 포인터를 공유하고, `reset` 함수 호출 시 해당 개체의 원시 포인터와 참조카운트 값은 빈값(empty) 이 되고,
* 공유 되었던 개체의 참조 카운트는 1 로 줄어들게 된다.
* `vector.reset()` 과 `vector = nullptr;` 은 동일한 코드이다.

![스크린샷 2024-06-11 11.33.14.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/946e7fde-ce60-42ac-81ec-fcbfa8d4c809/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.33.14.png)

#### shared\_ptr 를 사용할 때 발생하는 순환 참조 문제

* 아래와 같이 `Person`, `Pet` 개체를 생성하고 참조 카운트가 각각 1인 것을 확인할 수 있다.

![스크린샷 2024-06-11 11.41.10.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/c5e34113-0af8-4e68-be4f-55b5d521a40a/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.41.10.png)

* 만약 `Person`, `Pet` 이 각각 `Pet`, `Person` 을 지정할 때 매개변수를 `shared_ptr` 을 통해 받게 되면 어떻게 될까?

![스크린샷 2024-06-11 11.42.29.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/827a8fbf-845c-48c3-abd4-5bd8bdb330e1/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.42.29.png)

* 다음과 같이 참조 카운트가 1씩 증가해 2가 된 것을 확인할 수 있다.
* 하지만, 해당 소멸자가 호출되지 않을것을 알 수 있다. 왜그럴까?
  * 이 경우 `owner`, `pet` 지역변수는 메모리 해제 되었지만, 내부적으로 서로를 참조하고 있는 순환 참조가 일어났다. → 지역변수 메모리가 해제되었기에 접근할 수 있는 방법도 없다.
* `weak_ptr`(약한 포인터) 를 통해서 순환 참조를 해결할 수 있다.

![스크린샷 2024-06-11 11.44.04.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/15bf01b1-d293-4b22-81a5-0c7f804f5608/c5440979-b18b-4884-bbe4-4a99588cb499/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA\_2024-06-11\_11.44.04.png)
