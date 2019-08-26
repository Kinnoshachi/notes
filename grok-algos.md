#### Log base 10
![i1](https://learning.oreilly.com/library/view/grokking-algorithms-an/9781617292231/007fig01.jpg)

#### Most common Big O

* O(log n), also known as log time. Example: Binary search.           
* O(n),also known as linear time. Example: Simple search.          
* O(n * log n). Example: A fast sorting algorithm, like quicksort (coming up in chapter 4).         
* O(n2). Example: A slow sorting algorithm, like selection sort (coming up in chapter 2).         
* O(n!). Example: A really slow algorithm, like the traveling salesperson (coming up next!).


```python
def binary_search(list, item):
    low = 0
    high = len(list)-1

    while low <= high:
        mid = (low + high) /2
        guess = list[mid]
        if guess == item:
            return mid
        if guess > item:
            high = mid -1
        else:
            low = mid + 1
    return None
```

#### Binary search(must be sorted list) O(Log n)
#### Simple search O(n)

## _______________________________
