### 面试题 2：实现 Singleton 模式
> [http://python.jobbole.com/87294/](http://python.jobbole.com/87294/)


1 使用 `__new__` 方法

``` python
class Singleton:
    def __new__(cls, *args, **kw):
        if not hasattr(cls, '_instance'):
            cls._instance = super().__new__(cls)
        return cls._instance
```

加锁

``` python
import threading

class Singleton:
    lock = threading.RLock()
    def __new__(cls, *args, **kw):
        cls.lock.acquire()
        if not hasattr(cls, '_instance'):
            cls._instance = super().__new__(cls, *args, **kw)
        cls.lock.release()
        return cls._instance
```


2 使用装饰器实现

``` python
import functools

def singleton(cls):
    instances = {}
    @functools.wraps(cls)
    def get_instance(*args, **kw):
        if not cls in instances:
            instances[cls] = cls(*args, **kw)
        return instances[cls]
    return get_instance

@singleton
class A:
    pass
```

3 使用 metaclass

``` python
class Singleton(type):
    _instances = {}
    def __call__(cls, *args, **kw):
        if not cls in cls._instances:
            cls._instances[cls] = super().__call__(*args, **kw)
        return cls._instances


class A(metaclass=Singleton):
    pass
```

4 使用模块

5 还是老方法实用

``` python
class Singleton:

    def __init__(self):
        print('__init__')

    @classmethod
    def instance(cls):
        if not hasattr(cls, '_instance'):
            cls._instance = cls()
        return cls._instance
```

### 面试题 3：数组中重复的数字

#### 题目一：找出数组中重复的数字

``` python
def duplicate(alist):
    if not isinstance(alist, list) or not alist:
        return StopIteration
    
    for n in alist:
        if not isinstance(n, int) or n < 0 or n >= len(alist):
            return StopIteration
    
    # 算法开始
    for i, n in enumerate(alist):
        while not alist[i] == i:
            if alist[i] == alist[alist[i]]:
                yield alist[i]
                break
            o = alist[i]
            alist[i], alist[o] = alist[o], alist[i]
```

时间复杂度：`O(n)`  
空间复杂度：`O(1)`


#### 题目二：不修改数组找出重复的数字

##### 方法一

用类似二分查找算法的方式，不需要辅助数组

``` python
def getDuplication(alist):
    
    if not isinstance(alist, list) or not alist:
        return -1
    
    start = 1
    end = len(alist) - 1
    while end >= start:
        middle = ((end - start) >> 1) + start
        print(middle)
        count = countRange(alist, start, middle)
        if end == start:
            if count > 1:
                return start
            else:
                break
        
        if count > (middle - start + 1):
            end = middle
        else:
            start = middle + 1
    
    return -1

def countRange(alist, start, end):
    count = 0
    for i in alist:
        if start <= i <= end:
            count += 1
    return count
```

时间复杂度：`O(nlogn)`  
空间复杂度：`O(1)`

##### 方法二

利用辅助数组

``` python
def getDuplication(alist):
    temp_list = [0 for i in range(len(alist))]
    for n in alist:
        if n == temp_list[n]:
            yield n
        else:
            temp_list[n] = n
```

### 面试题 4：二维数组中的查找

``` python
def find(matrix, number):
    rows = 0
    columns = len(matrix[0]) - 1
    rows_total = len(matrix)-1
    while rows <= rows_total and 0 <= columns:
        a = matrix[rows][columns]

        if a == number:
            return True

        if a > number:
            columns -= 1
        else:
            rows += 1

    return False
```
### 面试题 10：斐波那契数列
青蛙跳台阶问题

矩形覆盖问题


时间复杂度： `O(n)`

``` python
def fibonacci(n):
    if n < 2:
        return n
    fib_one = 0
    fib_two = 1
    fib_n = 0
    for i in range(2,n+1):
        fib_n = fib_one + fib_two
        fib_one = fib_two
        fib_two = fib_n
    return fib_n
```

### 番外：快速排序

``` python
def partition(data, start, end):
    i = start + 1
    j = end
    v = data[start]
    while True:
        while data[i] < v and i != end:
            i += 1
        while v < data[j] and j != start:
            j -= 1
        if i >= j:
            break
        data[i], data[j] = data[j], data[i]
    
    data[start], data[j] = data[j], data[start]

    return j

def quick_sort(data, start, end):
    if start >= end:
        return
    
    j = partition(data, start, end) 
    quick_sort(data, start, j-1)
    quick_sort(data, j+1, end)
```
### 面试题 11：旋转数组中的最小数字

采用二分查找法解决

``` python
def min(data):
    head = 0
    foot = len(data)-1
    mid = head
    while data[head] >= data[foot]:
        if (head + 1) == foot:
            mid = foot
            break
        mid = round((foot + head)/2)
        if data[head] <= data[mid]:
            head = mid
        elif data[mid] <= data[foot]:
            foot = mid
    return data[mid]
```

以上代码如果遇到数组 `[1, 0, 1, 1, 1]` 或者 `[1, 1, 1, 0, 1]` 就无法找出最小值了，此时需要顺序查找：

``` python
def min(data):
    head = 0
    foot = len(data)-1
    mid = head
    while data[head] >= data[foot]:
        if (head + 1) == foot:
            mid = foot
            break
        mid = round((foot + head)/2)
        if data[head] == data[foot] and data[head] == data[mid]:
            # 进行顺序查找
            return min_in_order(data, head, foot)

        if data[head] <= data[mid]:
            head = mid
        elif data[mid] <= data[foot]:
            foot = mid
    return data[mid]

def min_in_order(data, head, foot):
    result = data[head]
    for i in range(head+1, foot+1):
        if result > data[i]:
            result = data[i]
    return result
```

### 面试题 15：二进制中 1 的个数

``` python
def number_of_1(n):
    count = 0
    while n:
        count += 1
        n = (n-1) & n
    return count
```

### 面试题 16：数值的整数次方

``` python
def power_with_unsigned_exponent(base, exponent):
    if exponent == 0.0:
        return 1.0
    if exponent == 1:
        return base
    
    result = power_with_unsigned_exponent(base, exponent >> 1)
    result *= result
    if exponent & 1 == 1:
        result *= base
    
    return result

def power(base, exponent):
    
    if base == 0.0 and exponent < 0:
        raise ValueError("exponent can't is netative when base is 0")
    abs_exponent = exponent
    if exponent < 0:
        abs_exponent = -exponent

    result = power_with_unsigned_exponent(base, abs_exponent)

    if exponent < 0:
        return 1.0 / result
    
    return result
```

### 面试题 21：调整数组顺序使奇数位于偶数前面

#### 方法一

初级方法

``` python
def reorder(data):
    start = 0
    end = len(data) - 1
    while True:
        while data[start] & 1:
            start += 1
        
        while not data[end] & 1:
            end -= 1
        if start > end:
            break
        data[start], data[end] = data[end], data[start]
```