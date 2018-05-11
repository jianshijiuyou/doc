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

`itertools.accumulate(iterable[, func])`

创建一个迭代器，它返回累计的和或其他二元函数的累计结果（通过可选的 func 参数指定）。如果提供了 func，它应该是两个参数的函数。iterable 的元素可以是任何能够被接受为 func 参数的类型。（例如，如果是默认的相加操作，元素可以是任何可相加类型，包括 `Decimal` 或 `Fraction`。）如果传入的迭代器为空，则输出迭代器也将为空。

说再多不如看个例子

``` python
In : list(itertools.accumulate([1, 2, 3, 4, 5]))
Out: [1, 3, 6, 10, 15]
```

传入序列为 `1, 2, 3, 4, 5`

输出应该为 `1, 1+2, 1+2+3, 1+2+3+4, 1+2+3+4+5` 结果就是 `1, 3, 6, 10, 15`


再看个例子

``` python
In : list(itertools.accumulate([1, 2, 3, 4, 5], func=lambda x, y:x*y))
Out: [1, 2, 6, 24, 120]
```

传入序列为 `1, 2, 3, 4, 5`

输出应该为 `1, 1*2, 1*2*3, 1*2*3*4, 1*2*3*4*5` 结果就是 `1, 2, 6, 24, 120`


大致相当于：

``` python
def accumulate(iterable, func=operator.add):
    'Return running totals'
    # accumulate([1,2,3,4,5]) --> 1 3 6 10 15
    # accumulate([1,2,3,4,5], operator.mul) --> 1 2 6 24 120
    it = iter(iterable)
    try:
        total = next(it)
    except StopIteration:
        return
    yield total
    for element in it:
        total = func(total, element)
        yield total
```

!> `func` 参数在 Python 3.3 以上可用

### chain

`itertools.chain(*iterables)`

创建一个迭代器，它从第一个迭代器中返回元素，直到它耗尽，然后继续下一个迭代器，直到所有迭代器都耗尽。用于将连续序列作为单个序列进行处理。

看个例子

``` python
In : a1 = ["a", "b", "c"]

In : a2 = ("d", "e")

In : a3 = (it for it in range(3))

In : list(itertools.chain(a1, a2, a3))
Out: ['a', 'b', 'c', 'd', 'e', 0, 1, 2]
```


大致相当于：

``` python
def chain(*iterables):
    # chain('ABC', 'DEF') --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```

### chain.from_iterable

`classmethod chain.from_iterable(iterable)`

`chain.from_iterable` 和 `chain` 差不多，只不过参数只能是一个可迭代序列，且序列中的每一项必须可迭代。

看个例子

``` python
In : a1 = ["a", "b", "c"]

In : a2 = ("d", "e")

In : a3 = (it for it in range(3))

In : list(itertools.chain.from_iterable([a1, a2, a3]))
Out: ['a', 'b', 'c', 'd', 'e', 0, 1, 2]
```

大致相当于：

``` python
def from_iterable(iterables):
    # chain.from_iterable(['ABC', 'DEF']) --> A B C D E F
    for it in iterables:
        for element in it:
            yield element
```


### compress

`itertools.compress(data, selectors)`

创建一个迭代器，用于过滤来自 `data` 的元素，仅返回那些在 `selectors` 中对应位置上有元素并且计算结果为 `True` 的元素。`data` 或 `selectors` 迭代器耗尽时停止。

大致相当于：

``` python
def compress(data, selectors):
    # compress('ABCDEF', [1,0,1,0,1,1]) --> A C E F
    return (d for d, s in zip(data, selectors) if s)
```


### dropwhile

`itertools.dropwhile(predicate, iterable)`

将 `iterable` 中的元素依次交给 `predicate` 处理，直到 `predicate(elem)` 的值为 `False` 时，返回从 `elem` 开始的所有元素。

大致相当于：

``` python
def dropwhile(predicate, iterable):
    # dropwhile(lambda x: x<5, [1,4,6,4,1]) --> 6 4 1
    iterable = iter(iterable)
    for x in iterable:
        if not predicate(x):
            yield x
            break
    for x in iterable:
        yield x
```


### filterfalse

`itertools.filterfalse(predicate, iterable)`


返回 `predicate(elem)` 为 `False` 的结果集， 与 `filter` 的相反：

举个栗子

``` python
In : list(itertools.filterfalse(lambda x: x%2, range(10)))
Out: [0, 2, 4, 6, 8]

In : list(filter(lambda x: x%2, range(10)))
Out: [1, 3, 5, 7, 9]
```

大致相当于：

``` python
def filterfalse(predicate, iterable):
    # filterfalse(lambda x: x%2, range(10)) --> 0 2 4 6 8
    if predicate is None:
        predicate = bool
    for x in iterable:
        if not predicate(x):
            yield x
```


### groupby

`itertools.groupby(iterable, key=None)`

根据 `key` 给 `iterable` 中的元素分组，如果 `key` 为 `None`，则依据元素自身分组。

注意，注意，注意：必须先排序后才能分组，因为 `groupby` 是通过比较相邻元素来分组的。

看个例子

``` python
In : {k:list(g) for k, g in itertools.groupby('aaaabbbcccccc')}
Out:
{'a': ['a', 'a', 'a', 'a'],
 'b': ['b', 'b', 'b'],
 'c': ['c', 'c', 'c', 'c', 'c', 'c']}

# 打乱顺序后结果就不正确了

In : {k:list(g) for k, g in itertools.groupby('aaccaabbbccc')}
Out: {'a': ['a', 'a'], 'c': ['c', 'c', 'c'], 'b': ['b', 'b', 'b']}
```
> 因为分组后每组数据是生成器，所以用 `list` 转换下，看起来更直观。


`groupby()` 大致等价于：

``` python
class groupby:
    # [k for k, g in groupby('AAAABBBCCDAABBB')] --> A B C D A B
    # [list(g) for k, g in groupby('AAAABBBCCD')] --> AAAA BBB CC D
    def __init__(self, iterable, key=None):
        if key is None:
            key = lambda x: x
        self.keyfunc = key
        self.it = iter(iterable)
        self.tgtkey = self.currkey = self.currvalue = object()
    def __iter__(self):
        return self
    def __next__(self):
        while self.currkey == self.tgtkey:
            self.currvalue = next(self.it)    # Exit on StopIteration
            self.currkey = self.keyfunc(self.currvalue)
        self.tgtkey = self.currkey
        return (self.currkey, self._grouper(self.tgtkey))
    def _grouper(self, tgtkey):
        while self.currkey == tgtkey:
            yield self.currvalue
            try:
                self.currvalue = next(self.it)
            except StopIteration:
                return
            self.currkey = self.keyfunc(self.currvalue)
```


### islice

`itertools.islice(iterable, stop)`  
`itertools.islice(iterable, start, stop[, step])`

创建一个迭代器， 从可迭代的序列 `iterable` 中返回数据，从 `start` 位置开始，到 `stop`， 步长为 `step`

大致相当于：

``` python
def islice(iterable, *args):
    # islice('ABCDEFG', 2) --> A B
    # islice('ABCDEFG', 2, 4) --> C D
    # islice('ABCDEFG', 2, None) --> C D E F G
    # islice('ABCDEFG', 0, None, 2) --> A C E G
    s = slice(*args)
    start, stop, step = s.start or 0, s.stop or sys.maxsize, s.step or 1
    it = iter(range(start, stop, step))
    try:
        nexti = next(it)
    except StopIteration:
        # Consume *iterable* up to the *start* position.
        for i, element in zip(range(start), iterable):
            pass
        return
    try:
        for i, element in enumerate(iterable):
            if i == nexti:
                yield element
                nexti = next(it)
    except StopIteration:
        # Consume to *stop*.
        for i, element in zip(range(i + 1, stop), iterable):
            pass
```


### starmap

`itertools.starmap(function, iterable)`

把 `iterable` 中的元素传递给 `function(*elem)` 处理，然后返回结果。 `iterable` 中的元素 `elem` 可以是任意值，这取决于 `function` 函数。

大致相当于：

``` python
def starmap(function, iterable):
    # starmap(pow, [(2,5), (3,2), (10,3)]) --> 32 9 1000
    for args in iterable:
        yield function(*args)
```

再看个例子：找出一组序列中每个序列的最小值。

``` python
In : a1 = [(1, 2,), (5,1), (32, 22, 11)]

In : list(itertools.starmap(min,a1))
Out: [1, 1, 11]
```


### takewhile

`itertools.takewhile(predicate, iterable)`

将 `iterable` 中的元素依次交给 `predicate` 处理，返回 `predicate(elem)` 的值为 `True` 的元素，当 `predicate(elem)` 的值为 `False` 时立即结束（如果处理第一个元素就返回 False ，那么返回空列表），这和 `dropwhile()` 相反。

大致相当于：

``` python
def takewhile(predicate, iterable):
    # takewhile(lambda x: x<5, [1,4,6,4,1]) --> 1 4
    for x in iterable:
        if predicate(x):
            yield x
        else:
            break
```


### tee

`itertools.tee(iterable, n=2)`

从 `iterable` 中返回 n 个独立的迭代器。

下面的 Python 代码有助于解释什么是 tee（尽管实际实现更复杂，仅使用了一个底层的 FIFO 队列）。

大致相当于：

``` python
def tee(iterable, n=2):
    it = iter(iterable)
    deques = [collections.deque() for i in range(n)]
    def gen(mydeque):
        while True:
            if not mydeque:             # when the local deque is empty
                try:
                    newval = next(it)   # fetch a new value and
                except StopIteration:
                    return
                for d in deques:        # load it to all the deques
                    d.append(newval)
            yield mydeque.popleft()
    return tuple(gen(d) for d in deques)
```

看个栗子

``` python
In : a1 = [1, 2, 3, 4, 5]

In : [list(it) for it in itertools.tee(a1)]
Out: [[1, 2, 3, 4, 5], [1, 2, 3, 4, 5]]

In : [list(it) for it in itertools.tee(a1, 3)]
Out: [[1, 2, 3, 4, 5], [1, 2, 3, 4, 5], [1, 2, 3, 4, 5]]
```

### zip_longest

`itertools.zip_longest(*iterables, fillvalue=None)`

制作一个迭代器，用于聚合来自每个迭代器的元素。如果迭代的长度不均匀，缺少的值将用 `fillvalue` 填充。继续迭代下去，直到最长的迭代耗尽。

大致相当于：

``` python
class ZipExhausted(Exception):
    pass

def zip_longest(*args, **kwds):
    # zip_longest('ABCD', 'xy', fillvalue='-') --> Ax By C- D-
    fillvalue = kwds.get('fillvalue')
    counter = len(args) - 1
    def sentinel():
        nonlocal counter
        if not counter:
            raise ZipExhausted
        counter -= 1
        yield fillvalue
    fillers = repeat(fillvalue)
    iterators = [chain(it, sentinel(), fillers) for it in args]
    try:
        while iterators:
            yield tuple(map(next, iterators))
    except ZipExhausted:
        pass
```

如果其中一个 `iterables` 可能是无限的，那么 `zip_longest()` 函数应该包含限制调用次数的方式（例如 `islice()` 或 `takewhile()`）。

`fillvalue` 如果未指定，默认为 `None`。

看个例子

``` python
In : a1 = [23, 18, 56]

In : a2 = ('zhang', 'li', 'wang', 'zhao', 'qiao')

In : list(itertools.zip_longest(a1, a2, fillvalue=20))
Out: [(23, 'zhang'), (18, 'li'), (56, 'wang'), (20, 'zhao'), (20, 'qiao')]
```


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

将 iterable 中的元素按照 r 个 r 个组合。（比如 r=2，那就是两个两个组合）

排列顺序可变，元素不重复（同一个组合内不重复）。

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

将 iterable 中的元素按照 r 个 r 个组合。（比如 r=2，那就是两个两个组合）

排列顺序不可变，元素不重复（同一个组合内不重复）。

``` python
In : list(itertools.combinations('123', 2))
Out: [('1', '2'), ('1', '3'), ('2', '3')]
```
combinations 必须要指定 `r` 了，因为排列顺序不可变，所有 `123` 长度为 3 的话只有一种情况。

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

将 iterable 中的元素按照 r 个 r 个组合。（比如 r=2，那就是两个两个组合）

排列顺序不可变，元素可重复。

``` python
In : list(itertools.combinations_with_replacement('123', 2))
Out: [('1', '1'), ('1', '2'), ('1', '3'), ('2', '2'), ('2', '3'), ('3', '3')]
```

大致相当于：

``` python
def combinations_with_replacement(iterable, r):
    # combinations_with_replacement('ABC', 2) --> AA AB AC BB BC CC
    pool = tuple(iterable)
    n = len(pool)
    if not n and r:
        return
    indices = [0] * r
    yield tuple(pool[i] for i in indices)
    while True:
        for i in reversed(range(r)):
            if indices[i] != n - 1:
                break
        else:
            return
        indices[i:] = [indices[i] + 1] * (r - i)
        yield tuple(pool[i] for i in indices)
```