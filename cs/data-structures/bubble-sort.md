# 버블 정렬(Bubble Sort)

## 버블 정렬 개념 정리&#x20;

* 서로 인접한 두 원소를 검사하여 정렬하는 알고리즘
* 인접한 2개의 레코드를 비교하여 크기가 순서대로 되어 있지 않으면 서로 교환한다.
* 이미 정렬된 상태이면 위 과정을 종료한다.

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-09 11.12.58.png" alt="" width="563"><figcaption></figcaption></figure>

## 코드 및 결과&#x20;

```cpp
int arr[] {1,2,3,4,5};
int n = sizeof(arr) / sizeof(arr[0]);

Print(arr, n);
cout << endl;

// TODO : Bubble Sort
{
    for(int i=0; i<n-1; i++){
        
        bool swapped = false;
        
        for(int j=0; j<n-i-1; j++){
            //            for(int j=0; j<n-1; j++){
            if(arr[j] > arr[j+1]){
                swap(arr[j], arr[j+1]);
                swapped = true;
            }
            Print(arr, n);
        }
        cout << endl;
        
        if(swapped == false)
            break;
    }
}
```

## 안정성 확인&#x20;

* 근접해 있는 경우에만 swap 을 진행하기 때문에, 안정성이 있다.&#x20;
* 선택정렬은 인덱스가 근접하지 않더라도 swap 을 진행하기 때문에, 안정성이 없다.&#x20;

```cpp
Element arr[] = {{2, 'a'},{2, 'b'},{1, 'c'}};
int size = sizeof(arr) / sizeof(arr[0]);

Print(arr, size);

cout << endl;

{
    for(int i=0; i<size; i++){
        
        bool swapped = false;
        
        for(int j=0; j<size-i-1; j++){
            if(arr[j].key > arr[j+1].key){
                swap(arr[j], arr[j+1]);
                swapped = true;
            }
                
            Print(arr, size);
        }
        
        cout << endl;
        
        if(swapped == false)
            break;
    }
}

Print(arr, size);
```

<figure><img src="../../.gitbook/assets/스크린샷 2025-04-09 11.16.01.png" alt="" width="98"><figcaption></figcaption></figure>
