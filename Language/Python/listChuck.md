## Python 배열 n 개로 분할하기
```
def list_chuck(arr, n):
    return [arr[i: i + n] for i in range(0, len(arr), n)]


array = [1, 2, 3, 4, 5, 6, 7, 8]
result_array = list_chuck(array, 3)
print(result_array)        # [[1, 2, 3], [4, 5, 6], [7, 8]]
```

-   `for i in range(0, len(arr), n)` : 0 부터 배열의 최대길이까지 n 개 씩 증가
-   `arr[i:i+n]` : i 부터 n 개의 배열, i 가 0 일 경우, 1, 2, 3
