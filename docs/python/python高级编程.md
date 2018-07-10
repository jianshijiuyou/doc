> [原文地址](http://dongweiming.github.io/Expert-Python/#1)

# 设置全局变量

``` python
>>> a
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
NameError: name 'a' is not defined
>>> d = {'a': 1, 'b':2}
>>> # 粗暴的写法
>>> for k, v in d.iteritems():
...     exec "{}={}".format(k, v)
...
>>> # 文艺的写法
>>> globals().update(d)
>>> a, b
(1, 2)
>>> 'a', 'b'
('a', 'b')
>>> globals()['a'] = 'b'
>>> a
'b'
```

# 字符串格式化

``` python
>>> "{key}={value}".format(key="a", value=10) # 使⽤命名参数
'a=10'
>>> "[{0:<10}], [{0:^10}], [{0:*>10}]".format("a") # 左中右对⻬
'[a         ], [    a     ], [*********a]'
>>> "{0.platform}".format(sys) # 成员
'darwin'
>>> "{0[a]}".format(dict(a=10, b=20)) # 字典
'10'
>>> "{0[5]}".format(range(10)) # 列表
'5'
>>> "My name is {0} :-{{}}".format('Fred') # 真得想显示{},需要双{}
'My name is Fred :-{}'
>>> "{0!r:20}".format("Hello")
"'Hello'             "
>>> "{0!s:20}".format("Hello")
'Hello               '
>>> "Today is: {0:%a %b %d %H:%M:%S %Y}".format(datetime.now())
'Today is: Mon Mar 31 23:59:34 2014'
```

# 列表去重

``` python
>>> l = [1, 2, 2, 3, 3, 3]
>>> {}.fromkeys(l).keys()
[1, 2, 3] # 列表去重(1)
>>> list(set(l)) # 列表去重(2)
[1, 2, 3]
In [2]: %timeit list(set(l))
1000000 loops, best of 3: 956 ns per loop
In [3]: %timeit {}.fromkeys(l).keys()
1000000 loops, best of 3: 1.1 µs per loop
In [4]: l = [random.randint(1, 50) for i in range(10000)]
In [5]: %timeit list(set(l))
1000 loops, best of 3: 271 µs per loop
In [6]: %timeit {}.fromkeys(l).keys()
1000 loops, best of 3: 310 µs per loop 
```
!> 在字典较大的情况下, 列表去重(1)略慢了

# 操作字典

``` python
>>> dict((["a", 1], ["b", 2])) # ⽤两个序列类型构造字典
{'a': 1, 'b': 2}
>>> dict(zip("ab", range(2)))
{'a': 0, 'b': 1}
>>> dict.fromkeys("abc", 1) # ⽤序列做 key,并提供默认 value
{'a': 1, 'c': 1, 'b': 1}
>>> {k:v for k, v in zip("abc", range(3))} # 字典解析
{'a': 0, 'c': 2, 'b': 1}
>>> d = {"a":1, "b":2}
>>> d.setdefault("a", 100) # key 存在,直接返回 value
1
>>> d.setdefault("c", 200) # key 不存在,先设置,后返回
200
>>> d
{'a': 1, 'c': 200, 'b': 2}
```

# 字典视图

``` python
>>> d1 = dict(a = 1, b = 2)
>>> d2 = dict(b = 2, c = 3)
>>> d1 & d2 # 字典不支持该操作
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unsupported operand type(s) for &: 'dict' and 'dict'
>>> v1 = d1.viewitems()
>>> v2 = d2.viewitems()
>>> v1 & v2 # 交集
set([('b', 2)])
>>> dict(v1 & v2) # 可以转化为字典
{'b': 2}
>>> v1 | v2 # 并集
set([('a', 1), ('b', 2), ('c', 3)])
>>> v1 - v2 #差集(仅v1有,v2没有的)
set([('a', 1)])
>>> v1 ^ v2 # 对称差集 (不会同时出现在 v1 和 v2 中)
set([('a', 1), ('c', 3)])
>>> ('a', 1) in v1 #判断
True
```

# vars

``` python
>>> vars() is locals()
True
>>> vars(sys) is sys.__dict__
True
```

# @contextmanager

上下文管理器协议包含 `__enter__` 和 `__exit__` 两个方法。`with` 语句开始运行时，会在
上下文管理器对象上调用 `__enter__` 方法。`with` 语句运行结束后，会在上下文管理器对
象上调用 `__exit__` 方法，以此扮演 `finally` 子句的角色。

在使用 `@contextmanager` 装饰的生成器中，`yield` 语句的作用是把函数的定义体分成两
部分：`yield` 语句前面的所有代码在 `with` 块开始时（即解释器调用 `__enter__` 方法
时）执行， `yield` 语句后面的代码在 `with` 块结束时（即调用 `__exit__` 方法时）执行。

``` python
import contextlib

@contextlib.contextmanager
def test():
    print("open...")
    yield 'arg...'
    print("close...")


with test() as arg:
    print(arg)

# open...
# arg...
# close...
```

# 静态方法和类方法的区别

``` python
>>> class User(object):
...     def a(self):
...         print 'a'
...     @staticmethod
...     def b():
...         print 'b'
...     @classmethod
...     def c(cls):
...         print 'c'
>>> u = User()
>>> u.a(), u.b(), u.c()
(a, b, c)
>>> User.a()
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unbound method a() must be called with User instance as first argument (got nothing instead)
>>> User.b()
b
>>> User.c()
c
```