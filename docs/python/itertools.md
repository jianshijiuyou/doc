> 官方文档地址: [itertools — Functions creating iterators for efficient looping](https://docs.python.org/3.6/library/itertools.html)

itertools 中包含三类迭代器:

* 可以无限产出的迭代器

| 迭代器 | 参数 | 结果 | 举例 |
|:---|:---|:---|:---|
| count() | start, [step] | start, start+step, start+2*step, ...| `count(10) --> 10 11 12 13 14 ...`
| cycle() | p | p0, p1, ... plast, p0, p1, ... | `cycle('ABCD') --> A B C D A B C D ...`
| repeat() | elem [,n] | elem, elem, elem, ...无限次或 n 次 | `repeat(10, 3) --> 10 10 10`

* 在最短输入序列上终止的迭代器

| 迭代器 | 参数 | 结果 | 举例 |
|:---|:---|:---|:---|
| accumulate()	|p [,func]	|p0, p0+p1, p0+p1+p2, …	| `accumulate([1,2,3,4,5]) --> 1 3 6 10 15`
| chain()|	p, q, …|	p0, p1, … plast, q0, q1, …	| `chain('ABC', 'DEF') --> A B C D E F`
| chain.from_iterable()	|iterable	|p0, p1, … plast, q0, q1, …	| `chain.from_iterable(['ABC', 'DEF']) --> A B C D E F`
| compress()	|data, selectors	|(d[0] if s[0]), (d[1] if s[1]), …	| `compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F`
| dropwhile()	|pred, seq	|seq[n], seq[n+1], 从 pred 返回 False 开始	| `dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1`
| filterfalse()|	pred, seq|	pred(elem) 为 False 的 seq 元素	| `filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8`
| groupby()	|iterable[, key]	| 按 key(v) 的值分组的子迭代器	| 
| islice()|	seq, [start,] stop [, step]	| `elements from seq[start:stop:step]`	| `islice('ABCDEFG', 2, None) --> C D E F G`
| starmap()|	func, seq	| func(\*seq[0]), func(\*seq[1]), …	| `starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000`
| takewhile()|	pred, seq	|seq[0], seq[1], 直到 pred 返回 False	| `takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4`
| tee()	|it, n	|it1, it2, … itn 将一个迭代器分成 n 个	 |
| zip_longest()	|p, q, …|	(p[0], q[0]), (p[1], q[1]), …	| `zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D-`


* 组合迭代器

| 迭代器 | 参数 | 结果 | 
|:---|:---|:---|:---|
| product()	|p, q, … [repeat=1]	|笛卡尔积，相当于一个嵌套的 for 循环
| permutations()|	p[, r]|排列顺序可变，元素不重复。（每一项用长度为 r 的元组表示。
| combinations()	|p, r	|排列顺序不可变，元素不重复。（每一项用长度为 r 的元组表示。
| combinations_with_replacement()|	p, r|排列顺序不可变，元素可重复。（每一项用长度为 r 的元组表示。
| `product('ABCD', repeat=2)`	 ||	`AA AB AC AD BA BB BC BD CA CB CC CD DA DB DC DD`
| `permutations('ABCD', 2)`	| |	`AB AC AD BA BC BD CA CB CD DA DB DC`
| `combinations('ABCD', 2)`	| |	`AB AC AD BC BD CD`
| `combinations_with_replacement('ABCD', 2)`| | `AA AB AC AD BB BC BD CC CD DD`



## 可以无限产出的迭代器


### count

`itertools.count(start=0, step=1)`

创建一个迭代器，返回从数字 `start` 开始的均匀间隔的值。通常用作 map() 的参数来生成连续的数据点。另外，和 zip() 一起使用可以添加序列号。大致相当于：
``` python
def count(start=0, step=1):
    # count(10) --> 10 11 12 13 14 ...
    # count(2.5, 0.5) -> 2.5 3.0 3.5 ...
    n = start
    while True:
        yield n
        n += step
```

使用浮点数进行计数时，有时可以通过替换乘法代码(multiplicative code)来实现更高的精度，例如：`(start + step * i for i in count())`。

!> 在 Python 3.1 中进行了更改：添加了 `step` 参数并允许使用非整数参数。

### cycle

`itertools.cycle(iterable)`

从迭代器 iterable 中返回元素并保存每个元素的副本。当迭代器 iterable 耗尽时，从保存的副本中返回元素。无限重复。大致相当于：

``` python
def cycle(iterable):
    # cycle('ABCD') --> A B C D A B C D A B C D ...
    saved = []
    for element in iterable:
        yield element
        saved.append(element)
    while saved:
        for element in saved:
              yield element
```

!> 请注意，cycle() 可能会占用大量的辅助存储（取决于 `iterable` 的长度）。

### repeat

`itertools.repeat(object[, times])`

创建一个一直返回 `object` 的迭代器。除非指定了 `times` 参数，否则无限期地运行。用作 map() 的参数时，当作调用函数的不变参数。与 zip() 一起使用时，用来创建元组记录的不变部分。  

大致相当于：

``` python
def repeat(object, times=None):
    # repeat(10, 3) --> 10 10 10
    if times is None:
        while True:
            yield object
    else:
        for i in range(times):
            yield object
```

repeat() 的一个常见用途是为 map 或 zip 提供一个常量值流：

``` python
>>> list(map(pow, range(10), repeat(2)))
[0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```


## 在最短输入序列上终止的迭代器



### accumulate
### chain
### chain.from_iterable
### compress
### dropwhile
### filterfalse
### groupby
### islice
### starmap
### takewhile
### tee
### zip_longest


















## 组合迭代器

### product

`itertools.product(*iterables, repeat=1)`

大致相当于生成器表达式中的嵌套 for 循环。例如，`product(A, B)` 与 `((x,y) for x in A for y in B)` 返回的结果相同。

要计算迭代器与自身的乘积，请使用可选的 `repeat` 关键字参数指定重复次数。例如，`product(A, repeat=4)` 意味着与 `product(A, A, A, A)` 相同。

这个函数大致等价于下面的代码，只是实际的实现不会在内存中建立中间结果：

``` python
def product(*args, repeat=1):
    # product('ABCD', 'xy') --> Ax Ay Bx By Cx Cy Dx Dy
    # product(range(2), repeat=3) --> 000 001 010 011 100 101 110 111
    pools = [tuple(pool) for pool in args] * repeat
    result = [[]]
    for pool in pools:
        result = [x+[y] for x in result for y in pool]
    for prod in result:
        yield tuple(prod)
```


### permutations

`itertools.permutations(iterable, r=None)`

排列顺序可变，元素不重复。
``` python
In : list(itertools.permutations('123', 2))
Out: [('1', '2'), ('1', '3'), ('2', '1'), ('2', '3'), ('3', '1'), ('3', '2')]
```

如果未指定 `r` 或者是 `None`，那么 `r` 默认为 `iterable` 的长度。

``` python
In : list(itertools.permutations('123'))
Out:
[('1', '2', '3'),
 ('1', '3', '2'),
 ('2', '1', '3'),
 ('2', '3', '1'),
 ('3', '1', '2'),
 ('3', '2', '1')]
```

大致相当于：

``` python
def permutations(iterable, r=None):
    # permutations('ABCD', 2) --> AB AC AD BA BC BD CA CB CD DA DB DC
    # permutations(range(3)) --> 012 021 102 120 201 210
    pool = tuple(iterable)
    n = len(pool)
    r = n if r is None else r
    if r > n:
        return
    indices = list(range(n))
    cycles = list(range(n, n-r, -1))
    yield tuple(pool[i] for i in indices[:r])
    while n:
        for i in reversed(range(r)):
            cycles[i] -= 1
            if cycles[i] == 0:
                indices[i:] = indices[i+1:] + indices[i:i+1]
                cycles[i] = n - i
            else:
                j = cycles[i]
                indices[i], indices[-j] = indices[-j], indices[i]
                yield tuple(pool[i] for i in indices[:r])
                break
        else:
            return
```

### combinations

`itertools.combinations(iterable, r)`

排列顺序不可变，元素不重复。

``` python
In [6]: list(itertools.combinations('123', 2))
Out[6]: [('1', '2'), ('1', '3'), ('2', '3')]
```
combinations 必须要制定 `r` 了，因为排列顺序不可变，所有 `123` 长度为 3 的话只有一种情况。

大致相当于：

``` python
def combinations(iterable, r):
    # combinations('ABCD', 2) --> AB AC AD BC BD CD
    # combinations(range(4), 3) --> 012 013 023 123
    pool = tuple(iterable)
    n = len(pool)
    if r > n:
        return
    indices = list(range(r))
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != i + n - r:
                break
        else:
            return
        indices[i] += 1
        for j in range(i+1, r):
            indices[j] = indices[j-1] + 1
        yield tuple(pool[i] for i in indices)
```








### combinations_with_replacement

`itertools.combinations_with_replacement(iterable, r)`

排列顺序不可变，元素可重复。

