# 정렬(선택정렬 / 버블정렬 / 삽입정렬)

## 1. 선택정렬

#### 선택정렬이란?

* 해당 순서에 원소를 넣을 위치는 이미 정해져 있고, 최솟값을 순서대로 해당 위치에 넣는 알고리즘

#### 선택정렬 순서

1. 주어진 배열 중에 최소값을 찾는다.&#x20;
2. 그 값을 맨 앞에 위치한 값과 교체한다.&#x20;
3. 맨 처음 위치를 뺀 나머지 배열에서 같은 방법으로 교체한다.&#x20;
4. 하나의 원소만 남을 때까지 위 과정을 반복한다.&#x20;

<figure><img src="../.gitbook/assets/image.png" alt="" width="391"><figcaption></figcaption></figure>

#### 선택정렬 코드

<pre class="language-cpp"><code class="lang-cpp"><strong>int main() 
</strong><strong>{
</strong>    int number[] = {15,3,23,64,77,46,42,174,68,78,91,5,76,310,84,342,176,120,33,41};
    int size = sizeof(number) / sizeof(int);
    
    // swap 할 공간
    int temp;

    for (int i = 0; i &#x3C; size-1; i++) 
    {			
        // 가장 처음값을 최솟값으로 가정한다.
        int indexMin = i;
        for (int j = i+1; j &#x3C; size; j++) 
        {		
            if(number[j] &#x3C; number[indexMin]) 
            {			
                indexMin = j;
                swap(number[j], number[indexMin])
            }
        }
        
        for (int e = 0; e &#x3C; size; e++) 
        {
	    cout &#x3C;&#x3C; number[e] &#x3C;&#x3C; " " &#x3C;&#x3C; flush;
	}
	cout &#x3C;&#x3C; endl;
        
    }

    cout &#x3C;&#x3C; "selection sorting DONE." &#x3C;&#x3C; endl;
    
    return 0;
}
</code></pre>

#### 선택정렬 장점&#x20;

* 정렬은 위한 비교 횟수는 많지만, 버블정렬에 비해서 비교적 교환하는 횟수가 적기 때문에, 버블정렬과 비교해 효율적이다.&#x20;

#### 선택정렬 단점

* 시간 복잡도가 O(n^2) 으로 비효율적이다.&#x20;
* 불안정 정렬(Unstable Sort)이다.

## 2. 버블정렬

#### 버블정렬이란?

* 버블정렬은 배열내의 두개의 인접한 index 를 비교하여 더 큰 숫자를 뒤로 보내 차곡차곡 쌓아 정렬하는 알고리즘.
* 배열의 뒤쪽부터 정렬하는 방법.

#### 버블정렬 순서&#x20;

1. 배열의 앞쪽부터 다음 인덱스와 비교해 더 큰 숫자를 뒤로 보내준다.&#x20;
2.  위 방법을 정렬이 완료될 때까지 반복한다.&#x20;

    <figure><img src="../.gitbook/assets/image (1).png" alt="" width="563"><figcaption></figcaption></figure>

#### 버블정렬 코드



<pre class="language-cpp"><code class="lang-cpp"><strong>int main() 
</strong>{ 
    int number[] = {11,234,23,4,1,5,6,2,65,764,825,46,72,47,26,69,793,25,498,245};
    int size = sizeof(number) / sizeof(int);
    
    for (int i = 0; i &#x3C; size -1; i++) 
    {				
        for (int j = 0; j &#x3C; size-i-1; j++) 
        {		
            if(number[j] > number[j+1]) 
            {					
                swap(number[j], number[j+1);
            }
        }
        
        for (int e = 0; e &#x3C; size; e++) 
        {
	    cout &#x3C;&#x3C; number[e] &#x3C;&#x3C; " " &#x3C;&#x3C; flush;
	}
	cout &#x3C;&#x3C; endl;
    }

    cout &#x3C;&#x3C; "bubble sorting DONE." &#x3C;&#x3C; endl;
}
</code></pre>

#### 버블정렬 장점

* 안정 정렬(Stable Sort) 입니다.

#### 버블정렬 단점

* 시간복잡도가 최악, 최선, 평균 모두 O(n^2)으로, 굉장히 비효율적입니다.
* 정렬 돼있지 않은 원소가 정렬 됐을때의 자리로 가기 위해서, 교환 연산(swap)이 많이 일어나게 됩니다.

## 3. 삽입정렬

#### 삽입정렬이란?

* 기준이 되는 숫자와 그 앞에있는 숫자를 비교하여 조건에 맞게 정렬하는 방법이다.&#x20;
* 0번째 인덱스는 앞쪽에 있는 숫자가 없기 때문에, 정렬의 시작은 1번째 인덱스로 시작한다.&#x20;

#### 삽입정렬 순서

<figure><img src="../.gitbook/assets/image (2).png" alt="" width="563"><figcaption></figcaption></figure>

#### 삽입정렬 코드&#x20;

<pre class="language-cpp"><code class="lang-cpp"><strong>int main() 
</strong>{ 
    int number[] = {11,234,23,4,1,5,6,2,65,764,825,46,72,47,26,69,793,25,498,245};
    int size = sizeof(number) / sizeof(int);
    
    int key;
    for (int i = 1; i &#x3C; size; i++) 
    {		
        int key = arr[i];		
        for (int j = i; j > 0; j--) 
        {		
            if(number[j-1] > number[j])
            {
                swap(number[j-1], number[j]);
            }
            else
            {
                break;
            }
        }
        
        for (int e = 0; e &#x3C; size; e++) 
        {
	    cout &#x3C;&#x3C; number[e] &#x3C;&#x3C; " " &#x3C;&#x3C; flush;
	}
	cout &#x3C;&#x3C; endl;
    }

    cout &#x3C;&#x3C; "bubble sorting DONE." &#x3C;&#x3C; endl;
}
</code></pre>

#### 삽입정렬 장점

* 대부분의 원소가 이미 정렬되어 있는 경우, 매우 효율적일 수 있습니다.
* 안정 정렬(Stable Sort) 입니다.

#### 삽입정렬 단점

* 평균과 최악의 시간복잡도가 O(n^2)으로 비효율적입니다.

<figure><img src="../.gitbook/assets/image (17).png" alt=""><figcaption></figcaption></figure>
