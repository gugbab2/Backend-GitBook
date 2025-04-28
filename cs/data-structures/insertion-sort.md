# 삽입 정렬(Insertion Sort)

## 삽입 정렬 개념 정리&#x20;

* 손안의 카드를 정렬하는 방법과 유사하다.
  * 새로운 카드를 기존의 정렬된 카드 사이의 올바른 자리를 찾아 삽입한다.
  * 새로 삽입될 카드의 수만큼 반복하게 되면 전체 카드가 정렬된다.
* 자료 배열의 모든 요소를 앞에서부터 차례대로 이미 정렬된 배열 부분과 비교 하여, 자신의 위치를 찾아 삽입함으로써 정렬을 완성하는 알고리즘
* 매 순서마다 해당 원소를 삽입할 수 있는 위치를 찾아 해당 위치에 넣는다.

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-09 14.03.34.png" alt="" width="563"><figcaption></figcaption></figure>

## 코드 및 결과&#x20;

```cpp
int arr[] = {1,2,4,5,3,6,7};
//    int arr[] = {7,6,5,4,3,2,1};
//    int arr[] = {2,4,1,5,3,6,7};
int size = sizeof(arr) / sizeof(arr[0]);

int i,j,temp;
for(i=1; i<size ;i++) {
    
    temp = arr[i];
    
    for(j=i; j>0; j--){
        if(arr[j-1] <= temp)
            break;
    
        arr[j] = arr[j-1];
    }
    arr[j] = temp;
    
    Print(arr, size);
} 
```

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-09 14.05.21.png" alt="" width="191"><figcaption></figcaption></figure>

## 안정성 확인&#x20;

## 정렬의 복잡도 분석&#x20;

버블정렬, 삽입정렬은 정렬이 된 경우에는 안쪽 반복문을 수행하지 않기 때문의 최선의 경우 시간복잡도가 O(n) 이다.&#x20;

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-28 10.45.41.png" alt=""><figcaption></figcaption></figure>
