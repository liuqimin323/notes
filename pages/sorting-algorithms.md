# Sorting Algorithms

## Selection Sort

Find the minimum number is an array, move it to frontl

```
arr[] = 64 25 12 22 11
// Find the minimum element in arr[0...4]
// and place it at beginning
11 25 12 22 64
// Find the minimum element in arr[1...4]
// and place it at beginning of arr[1...4]
11 12 25 22 64
// Find the minimum element in arr[2...4]
// and place it at beginning of arr[2...4]
11 12 22 25 64
// Find the minimum element in arr[3...4]
// and place it at beginning of arr[3...4]
11 12 22 25 64 
```

```cpp
int* selectionSort(int* arr) {
    int i = 0;
    while (i < arr.size()) {
        int min = arr[i];
        for (int j = i+1; j < arr.size(); j++) {
            if (arr[j] < min) {
                arr[i] = arr[j];
                arr[j] = min;
                min = arr[i];
            }
        }
        i++;
    }
    return arr;
}

```

Time Complexity: O(n^2)

## Bubble Sort

Repeatly swap position if arr[i] < arr[i-1] until the array is sorted.

```
First Pass:
( 5 1 4 2 8 ) –> ( 1 5 4 2 8 ), Here, algorithm compares the first two elements, and swaps since 5 > 1.
( 1 5 4 2 8 ) –>  ( 1 4 5 2 8 ), Swap since 5 > 4
( 1 4 5 2 8 ) –>  ( 1 4 2 5 8 ), Swap since 5 > 2
( 1 4 2 5 8 ) –> ( 1 4 2 5 8 ), Now, since these elements are already in order (8 > 5), algorithm does not swap them.
Second Pass:
( 1 4 2 5 8 ) –> ( 1 4 2 5 8 )
( 1 4 2 5 8 ) –> ( 1 2 4 5 8 ), Swap since 4 > 2
( 1 2 4 5 8 ) –> ( 1 2 4 5 8 )
( 1 2 4 5 8 ) –>  ( 1 2 4 5 8 )
Now, the array is already sorted, but our algorithm does not know if it is completed. The algorithm needs one whole pass without any swap to know it is sorted.
Third Pass:
( 1 2 4 5 8 ) –> ( 1 2 4 5 8 )
( 1 2 4 5 8 ) –> ( 1 2 4 5 8 )
( 1 2 4 5 8 ) –> ( 1 2 4 5 8 )
( 1 2 4 5 8 ) –> ( 1 2 4 5 8 )
```
**Time complexity** O(n^2)


```cpp
int* bubbleSort(int* arr) {
    bool isSorted = false;
    while(!isSorted) {
        isSorted = true;
        for(int i = 0; i < arr.size()-1; i++) {
            if (arr[i+1] < arr[i]) {
                int tmp = arr[i];
                arr[i] = arr[i+1];
                arr[i+1] = tmp;
                isSorted = false;
            }
        }
    }
    return arr;
}
```

Time Complexity: O(n^2)

## Insertion Sort

For each element, compare it with the previous elememt until it finds correct positin, insert the element to the position.


![img](https://media.geeksforgeeks.org/wp-content/uploads/insertionsort.png)

```cpp
void insertionSort(int arr[], int n)  
{
    for (int i = 1; i < n; i++) {
        int tmp = arr[i];
        for (int j = i-1; j >=0 && arr[j] > tmp; j-- ) {
            arr[j+1] = arr[j];
        }
        arr[j+1] = tmp;
    }
}
```

Time Complexity: O(n^2)

## Merge Sort

Divide to two parts, sort the divided parts, then merge them back.

```cpp
void mergeSort(int arr[]) {
    divide(arr, 0, arr.size());
}

void merge(int arr[], int l, int m, int r) {
    if (r  == l || arr[m] >= arr[m-1]) return;
    for (int i = m; i < r; i++) {
        int num = arr[i];
        int j = i-1;
        while (j >= l && arr[j] > num) {
            arr[j+1] = arr[j];
            j--;
        }
        arr[j+1] = num;
    }
}

void divide(int arr[], int l,  int r) {
    if (r-l <= 1) {
        return;
    }

    if (r - l == 2) {
        if (arr[l+1] < arr[l]) {
            int tmp = arr[l+1];
            arr[l+1]= arr[l];
            arr[l] = tmp;
        }
        return;
    }
    divide(arr, l, (r-l)/2 + l);
    divide(arr, (r-l)/2 + l, r);
    merge(arr, l,(r-l)/2 + l, r);
}

```

Time Complexity: O(nlogn)

## Quick Sort
Like Merge Sort, QuickSort is a Divide and Conquer algorithm. It picks an element as pivot and partitions the given array around the picked pivot. There are many different versions of quickSort that pick pivot in different ways.


1. Always pick first element as pivot.
2. Always pick last element as pivot (implemented below)
3. Pick a random element as pivot.
4. Pick median as pivot.

The key process in quickSort is partition(). Target of partitions is, given an array and an element x of array as pivot, put x at its correct position in sorted array and put all smaller elements (smaller than x) before x, and put all greater elements (greater than x) after x. All this should be done in linear time.

![image](https://www.geeksforgeeks.org/wp-content/uploads/gq/2014/01/QuickSort2.png)

```cpp
int partition(int arr[], int start, int end) {
    if (end - start <= 1) return start;

    int index = end - 1;
    int num = arr[index];
    for (int i = index - 1; i >= start; i--) {
        if (arr[i] > num) {
            int prev = arr[index-1];
            arr[index] = arr[i];
            arr[index-1] = num;
            if (i != index - 1) {
                arr[i] = prev;
            }
            index--;
        }
    }
    return index;
}
void quickSort(int arr[], int start, int end) {
    if (end - start <= 1) return;
    int pivot = partition(arr, start, end);
    quickSort(arr, start, pivot);
    quickSort(arr, pivot+1, end);
}

int main() {
    int N;
    cout << "Enter value:" << endl;
    cin >> N;
    int arr[N];
    for (int i = 0; i < N; i++) {
        cin >> arr[i];
    }
    quickSort(arr, 0, N);
    cout << "result: ";
    Helper::printArray(arr, N);
    return 0;
}
```

###  Time Complexity:
` T(n) = T(k) + T(n-k-1) + Θ(n)`

The first two terms are for two recursive calls, the last term is for the partition process. k is the number of elements which are smaller than pivot.

The time taken by QuickSort depends upon the input array and partition strategy. Following are three cases.

**Worst Case:** The worst case occurs when the partition process always picks greatest or smallest element as pivot. If we consider above partition strategy where last element is always picked as pivot, the worst case would occur when the array is already sorted in increasing or decreasing order. Following is recurrence for worst case.

```
 T(n) = T(0) + T(n-1) + θ(n)
which is equivalent to  
 T(n) = T(n-1) + θ(n)
 ```
The solution of above recurrence is θ(n2).

**Best Case:** The best case occurs when the partition process always picks the middle element as pivot. Following is recurrence for best case.

 T(n) = 2T(n/2) + θ(n)
The solution of above recurrence is θ(nLogn). It can be solved using case 2 of Master Theorem.

**Average Case:**
To do average case analysis, we need to consider all possible permutation of array and calculate time taken by every permutation which doesn’t look easy.

We can get an idea of average case by considering the case when partition puts O(n/9) elements in one set and O(9n/10) elements in other set. Following is recurrence for this case.

``` T(n) = T(n/9) + T(9n/10) + θ(n)```

Solution of above recurrence is also O(nLogn)

Although the worst case time complexity of QuickSort is `O(n2)` which is more than many other sorting algorithms like Merge Sort and Heap Sort, QuickSort is faster in practice, because its inner loop can be efficiently implemented on most architectures, and in most real-world data. QuickSort can be implemented in different ways by changing the choice of pivot, so that the worst case rarely occurs for a given type of data. However, merge sort is generally considered better when data is huge and stored in external storage.

## Heap Sort

**Heap Sort Algorithm for sorting in increasing order:**
1. Build a max heap from the input data.
2. At this point, the largest item is stored at the root of the heap. Replace it with the last item of the heap followed by reducing the size of heap by 1. Finally, heapify the root of tree.
3. Repeat above steps while size of heap is greater than 1.

### Step:

1. Form a heap map:
```
Input data: 4, 10, 3, 5, 1
         4(0)
        /   \
     10(1)   3(2)
    /   \
 5(3)    1(4)
```

2. As a parent node must be greater than the child node, and 10 is greater then 4. swap them:
```
         10(0)
        /   \
     4(1)   3(2)
    /   \
 5(3)    1(4)

 ==> data: 10 4 3 5 1

```

3. Repeat step 2 until the it is a max heap map where every parent node is greater than child node. e.g. swap 4 and 5:
```
         10(0)
        /   \
     5(1)   3(2)
    /   \
 4(3)    1(4)

data: 10 5 3 4 1
```

4. Swap first and last node and delete the last node:
```
         1(0)               1(0)
        /   \              /  \
     5(1)   3(2)  ===>  5(1)   3(2)
    /   \               /    
 4(3)    10(4)        4(3)

data: 1 5 3 4 10
```

5. Repeat step 2 until it forms a max heap map:
```
         5(0)
        /   \
     4(1)   3(2)
    / 
 1(3) 
 data: 5 4 3 1 10
```

6. Repeat step 4: Swap first and last node and delete the last node:
```
         1(0)
        /   \
     4(1)   3(2)

 data: 1 4 3 5 10
```

7. Repeat step 2
```
         4(0)
        /   \
     1(1)   3(2)

 data: 4 1 3 5 10
```

8. Repeat step 4
```
         3(0)
        / 
     1(1)

 data: 3 1 4 5 10
```
9 As 3 is greater than 1, it is already a max heap, Repeat Step 4:
```
         1(0)

 data: 1 3 4 5 10
```

10. Only one node left, End.

**Tip:** For a binary tree with array input. 
```
left_child(i)  = 2 * i + 1
right_child(i) = 2 * i + 2
parent(i)      = (i - 1) / 2
```

**Time Complexity:** Time complexity of heapify is O(Logn). Time complexity of createAndBuildHeap() is O(n) and overall time complexity of Heap Sort is O(nLogn).

## Shell Sort
Similar to [Insertion Sort](#Insertion-Sort). Instead of finding appropriate position and swap number one by one, only find position in gap.

### Steps:
Example: [3 4 5 1 2]
1. Gap: n/2 = 2. compare arr[2] and arr[0]

    [`3` 4 `5` 1 2]
2. compare arr[3] and arr[1]:
    
    [3 `4` 5 `1` 2] = 1 < 4 => [3 `1` 5 `4` 2]

3. Compare arr[4] and arr[2]
    
    [3 1 `5` 4 `2`] = 2 < 5 => [3 1 `2` 4 `5`]
4. Compare arr[2] and arr[0].

    [`3` 1 `2` 4 5]
    => [`2` 1 `3` 4 5]
5. Gap now becomes n/4 = 1. Compare arr[1] and arr[0]:

    [`3` `1` 2 4 5] ==> [`1` `3` 2 4 5]
6. Compare arr[2] and arr[1]

    [`1` `3` `2` 4 5] ==> [1 `2` `3` 4 5]

```cpp
/* function to sort arr using shellSort */
int shellSort(int arr[], int n) 
{ 
    // Start with a big gap, then reduce the gap 
    for (int gap = n/2; gap > 0; gap /= 2) 
    { 
        // Do a gapped insertion sort for this gap size. 
        // The first gap elements a[0..gap-1] are already in gapped order 
        // keep adding one more element until the entire array is 
        // gap sorted  
        for (int i = gap; i < n; i += 1) 
        { 
            // add a[i] to the elements that have been gap sorted 
            // save a[i] in temp and make a hole at position i 
            int temp = arr[i]; 
  
            // shift earlier gap-sorted elements up until the correct  
            // location for a[i] is found 
            int j;             
            for (j = i; j >= gap && arr[j - gap] > temp; j -= gap) 
                arr[j] = arr[j - gap]; 
              
            //  put temp (the original a[i]) in its correct location 
            arr[j] = temp; 
        } 
    } 
    return 0; 
} 
```

**Time Complexity:** Time complexity of above implementation of shellsort is O(n2). In the above implementation gap is reduce by half in every iteration. There are many other ways to reduce gap which lead to better time complexity. See this for more details.