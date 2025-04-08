# 선택 정렬(Selection Sort)

## 선택 정렬 개념 정리&#x20;

* 제자리 정렬(in-place sorting) 알고리즘의 하나
  * 입력 배열(정렬되지 않은 값들) 이외에 다른 추가 메모리를 요구하지 않는 정렬 방법
* 해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 어떤 원소를 넣을지 선택하는 알고리즘
  * 첫 번째 순서에는 첫 번째 위치에 가장 최솟값을 넣는다.
  * 두 번째 순서에는 두 번째 위치에 남은 값 중에서의 최솟값을 넣는다.
  * …
* 과정 설명
  * 주어진 배열 중에서 최솟값을 찾는다. **(최솟값의 인덱스를 찾는다)**&#x20;
  * 그 값을 맨 앞에 위치한 값과 교체한다(패스(pass)).
  * 맨 처음 위치를 뺀 나머지 리스트를 같은 방법으로 교체한다.
  * 하나의 원소만 남을 때까지 위의 1\~3 과정을 반복한다.

## 코드 및 결과

* 로그 파일에 size 와 최솟값을 찾는 갯수에 대해서 작성하는 코드이다.&#x20;
* 해당 코드에 대한 결과로 그래프를 만들면 다음과 같다.&#x20;
  * **시간복잡도 : `O(N^2)`**&#x20;

```cpp
ofstream ofile("log.txt");
for(int size=1; size<1000; size++){
    
    int count = 0;
    int* arr = new int[size];
    for(int s = 0; s<size; s++){
        arr[s] = size-s;
    }
    
    int min_index;
    for(int i=0; i<size-1; i++){
        
        min_index = i;
        for(int j=i+1; j<size; j++){
            count++;
            if(arr[j] < arr[min_index]){
                min_index = j;
            }
        }
        swap(arr[i], arr[min_index]);
    }
    
    ofile << size << ", " << count << endl;
    
    delete[] arr;
}
ofile.close();
```

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-08 14.10.53.png" alt="" width="563"><figcaption></figcaption></figure>

## 안정성 확인&#x20;

* 하지만 선택 정렬은 단순한 자료형이 아닌 클래스나, 구조체를 사용하는 경우 안정성이 깨진다..&#x20;

```java
Element arr[] = {{2, 'a'}, {2, 'b'}, {1, 'c'}};
int size = sizeof(arr) / sizeof(arr[0]);

Print(arr, size);

int min_index;
for(int i=0; i<size-1; i++){
    min_index = i;
    for(int j=i+1; j<size; j++){
        if(arr[j].key < arr[min_index].key){
            min_index = j;
        }
    }
    swap(arr[i], arr[min_index]);
    
    Print(arr, size);
}
```

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-08 14.14.18.png" alt=""><figcaption></figcaption></figure>
