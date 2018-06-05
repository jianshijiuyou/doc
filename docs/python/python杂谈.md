# 简单 http 服务器

python2

```
python -m SimpleHTTPServer [port]
```

python3

```
python -m http.server [port]
```
默认 `8000` 端口

# 命令行格式化 JSON

``` python
echo '{"json":"obj"}' | python -m json.tool

# {
#     "json": "obj"
# }
```

# 计时

``` python
>>> import timeit
#执行命令
>>> t2 = timeit.Timer('x=range(1000)')
#显示时间
>>> t2.timeit()
10.620039563513103

#执行命令
>>> t1 = timeit.Timer('sum(x)', 'x = (i for i in range(1000))')
#显示时间
>>> t1.timeit()
0.1881566039438201
```

或者

``` python
In [1]: from timeit import timeit
  
In [2]: timeit('x=1')  
Out[2]: 0.03820111778328037  
  
In [3]: timeit('x=map(lambda x:x*10,range(32))')  
Out[3]: 8.05639690328919  
```

在 ipython 中可以直接使用

``` python
In [1]: timeit y=map(lambda x:x**10,range(32))
372 ns ± 4.94 ns per loop (mean ± std. dev. of 7 runs, 1000000 loops each)
```


# 时间格式

`date`, `datetime`, 和 `time` 都支持 `strftime(format)` 方法。

| 符号 | 含义 | 例子 |
|:-------------------
| `%a` | 星期几的缩写 | Sun, Mon, …, Sat
| `%A` | 星期几的全称 | Sunday, Monday, …, Saturday
| `%w` | 用数字表示星期几，`0` 表示星期天，`6` 表示星期六 | 0, 1, …, 6
| `%d`	| 某月的几号	| 01, 02, …, 31	 
| `%b`	|月份的缩写|	Jan, Feb, …, Dec 
| `%B`	| 月份的全称| January, February, …, December 
| `%m`	|用数字表示月份|	01, 02, …, 12	 
| `%y`	|两位数年份|	00, 01, …, 99	 
| `%Y`	|四位数年份|	0001, 0002, …, 2013, 2014, …, 9998, 9999	
| `%H`	|24 小时表示|	00, 01, …, 23	 
| `%I`	|12 小时表示|	01, 02, …, 12	 
| `%p`	|上下午表示|	AM, PM 
| `%M`	|分钟数|	00, 01, …, 59	 
| `%S`	|秒数	|00, 01, …, 59	(4)
| `%f`	|微秒数|	000000, 000001, …, 999999	(5)
| `%z`	|UTC 偏移量格式为 +HHMM 或 -HHMM |	(empty), +0000, -0400, +1030	
| `%Z`	|时区名称|	(empty), UTC, EST, CST	 
| `%j`	|一年中的第几天|	001, 002, …, 366	 
| `%U`	|一年中的第几周（星期天为一周的第一天）|	00, 01, …, 53
| `%W`	|一年中的第几周（星期一为一周的第一天）|	00, 01, …, 53
| `%c`	|按照本地设置表示日期时间|	`Tue Aug 16 21:30:00 1988` 
| `%x`	|按照本地设置表示日期	| 08/16/88 
| `%X`	|按照本地设置表示时间	| `21:30:00`
| `%%`	| 表示 `%` | `%`

# 字符串格式

格式字符串包含由花括号 `{}` 包围的 “替换字段”。任何不包含在大括号中的内容都将被视为文字文本，并将其原样复制到输出中。如果您需要在字面文本中包含大括号字符，则可以通过加倍 `{{` 和 `}}` 来将其转义。

替换字段的语法如下所示：

```
replacement_field ::=  "{" [field_name] ["!" conversion] [":" format_spec] "}"
field_name        ::=  arg_name ("." attribute_name | "[" element_index "]")*
arg_name          ::=  [identifier | digit+]
attribute_name    ::=  identifier
element_index     ::=  digit+ | index_string
index_string      ::=  <源字符除外 "]"> +
conversion        ::=  "r" | "s" | "a"
format_spec       ::=  <后面介绍>
```

!> python 3.1 后: 位置参数说明符可以省略，因此 `'{} {}'` 等同于 `'{0} {1}'`。

一些简单的格式字符串示例：

``` python
"First, thou shalt count to {0}"  # 引用第一个位置参数
"Bring me a {}"                   # 隐式引用第一个位置参数
"From {} to {}"                   # 与 "From {0} to {1}" 相同
"My quest is {name}"              # 引用关键字参数 'name'
"Weight in tons {0.weight}"       # 第一位置参数的 'weight' 属性
"Units destroyed: {players[0]}"   # 关键字参数 'players' 的第一个元素
```

目前支持三种转换标志：`'!s'` 是对值调用 `str()`，`'!r'` 是对值调用 `repr()`，`'!a'` 是对值调用 `ascii()`

一些例子：

``` python
"Harold's a clever {0!s}"        # 在参数上调用 str()
"Bring out the holy {name!r}"    # 在参数上调用 repr()
"More {!a}"                      # 在参数上调用 ascii()
```

## 格式规范迷你语言

标准格式说明符的一般形式是：

```
format_spec     ::=  [[fill]align][sign][#][0][width][grouping_option][.precision][type]
fill            ::=  <any character>
align           ::=  "<" | ">" | "=" | "^"
sign            ::=  "+" | "-" | " "
width           ::=  digit+
grouping_option ::=  "_" | ","
precision       ::=  digit+
type            ::=  "b" | "c" | "d" | "e" | "E" | "f" | "F" | "g" | "G" | "n" | "o" | "s" | "x" | "X" | "%"
```

如果指定了有效的 `align` 值，则可以在前面加上一个 `fill` 字符，该字符可以是任何字符，如果省略，则默认为空格。

各种 `align` 选项的含义如下：

| 选项 | 含义
|:------------
| `'<'`	| 强制字段在可用空间内左对齐（这是大多数对象的默认设置）。
| `'>'`	| 强制字段在可用空间内右对齐（这是数字的默认值）。
| `'='`	| 强制将填充放置在符号（如果有）之后但位于数字之前。这用于以 `'+000000120'` 格式打印字段。此对齐选项仅适用于数字类型。当 `'0'` 紧接在字段宽度之前时，它成为默认值。
| `'^'`	| 强制该字段在可用空间内居中。

`sign` 选项仅适用于数字类型，可以是以下之一：

| 选项 | 含义
|:------------
| `'+'`	| 表示 sign 应该用于正数和负数。
| `'-'`	| 表示 sign 只能用于负数（这是默认行为）。
| space	| 表示应在正数上使用前导空格，在负数上使用负号。

`','` 选项表示使用千位分隔符的逗号。对于可识别语言环境的分隔符，请改为使用 `'n'` 整数表示类型。

`'_'` 选项表示对于浮点表示类型和整数表示类型 `'d'` 的千位分隔符使用下划线。

`width` 是定义最小字段宽度的十进制整数。如果未指定，则字段宽度将由内容决定。

如果未给出明确的对齐方式，则在宽度字段前加上一个零（`'0'`）字符可为数字类型启用符号感知零填充。这相当于 `full` 字符 `'0'`，`align` 类型为 `'='`。

`precision` 是一个十进制数，表示在小数点后的浮点值中应该显示多少位数字，用 `'f'` 和 `'F'` 来表示，或者在小数点之前和之后，用 `'g'` 或 `'G'` 来表示浮点值。对于非数字类型，该字段指示最大字段大小 - 换言之，字段内容将使用多少个字符。对于整数值，不允许 `precision`。

最后，`type` 决定了数据应该如何呈现。

可用的字符串表示类型有：

| Type | 含义
|:------------
|`'s'`	| 字符串格式。这是字符串的默认类型，可以省略。
|None	| 与 `'s'` 一样。

可用的整数表示类型有：

| Type | 含义
|:------------
|`'b'`	| 二进制格式。输出基数为 2 的数字。
|`'c'`	| 字符。打印前将整数转换为相应的 unicode 字符。
|`'d'`	| 十进制整数。以 10 为基数输出数字。
|`'o'`	| 八进制格式。输出基数为 8 的数字。
|`'x'`	| 十六进制格式。输出基数为 16 的数字，对于 9 以上的数字使用小写字母。
|`'X'`	| 十六进制格式。输出基数为 16 的数字，对于 9 以上的数字使用大写字母。
|`'n'`	| 整数。这与 `'d'` 相同，除了它使用当前区域设置插入适当的数字分隔符。
|None	| 同 `'d'`。

浮点和小数值的可用表示类型有：

| Type | 含义
|:------------
|`'e'`	|指数表示法。使用字母 `'e'` 以科学记数法打印数字以指示指数。默认精度为 `6`。
|`'E'`	|指数表示法。与 `'e'` 相同，只是它使用大写字母 `'E'` 作为分隔符。
|`'f'`	|浮点数。将该数字显示为一个浮点数。默认精度为 `6`。
|`'F'`	|浮点数。与 `'f'` 相同，但将 `nan` 转换为 `NAN` 并将 `inf` 转换为 `INF`。
|`'g'`	|略
|`'G'`	|略
|`'n'`	|略
|`'%'`	|百分比。将数字乘以 100，并以浮点 (`'f'`) 格式显示，然后显示百分号。
|None	|类似 `'g'`

## 格式化例子

本节包含 `str.format()` 语法的示例以及与旧 `%` 格式的比较。

在大多数情况下，语法与旧的 `%` 格式类似，但添加了 `{}` 和 `:` 而不是 `%`。例如，`'%03.2f'` 可以翻译为 `'{:03.2f}'`。

按位置访问参数：

``` python
>>> '{0}, {1}, {2}'.format('a', 'b', 'c')
'a, b, c'
>>> '{}, {}, {}'.format('a', 'b', 'c')  # 3.1+ only
'a, b, c'
>>> '{2}, {1}, {0}'.format('a', 'b', 'c')
'c, b, a'
>>> '{2}, {1}, {0}'.format(*'abc')      # unpacking argument sequence
'c, b, a'
>>> '{0}{1}{0}'.format('abra', 'cad')   # arguments' indices can be repeated
'abracadabra'
```

按名称访问参数：

``` python
>>> 'Coordinates: {latitude}, {longitude}'.format(latitude='37.24N', longitude='-115.81W')
'Coordinates: 37.24N, -115.81W'
>>> coord = {'latitude': '37.24N', 'longitude': '-115.81W'}
>>> 'Coordinates: {latitude}, {longitude}'.format(**coord)
'Coordinates: 37.24N, -115.81W'
```

访问参数的属性：

``` python
>>> c = 3-5j
>>> ('The complex number {0} is formed from the real part {0.real} '
...  'and the imaginary part {0.imag}.').format(c)
'The complex number (3-5j) is formed from the real part 3.0 and the imaginary part -5.0.'
>>> class Point:
...     def __init__(self, x, y):
...         self.x, self.y = x, y
...     def __str__(self):
...         return 'Point({self.x}, {self.y})'.format(self=self)
...
>>> str(Point(4, 2))
'Point(4, 2)'
```

访问参数的 item：

``` python
>>> coord = (3, 5)
>>> 'X: {0[0]};  Y: {0[1]}'.format(coord)
'X: 3;  Y: 5'
```

替换 `%s` 和 `%r`：

``` python
>>> "repr() shows quotes: {!r}; str() doesn't: {!s}".format('test1', 'test2')
"repr() shows quotes: 'test1'; str() doesn't: test2"
```

对齐文本并指定宽度：

``` python
>>> '{:<30}'.format('left aligned')
'left aligned                  '
>>> '{:>30}'.format('right aligned')
'                 right aligned'
>>> '{:^30}'.format('centered')
'           centered           '
>>> '{:*^30}'.format('centered')  # use '*' as a fill char
'***********centered***********'
```

替换 `%+f`，`%-f`，`% f` 并指定一个 sign：

``` python
>>> '{:+f}; {:+f}'.format(3.14, -3.14)  # show it always
'+3.140000; -3.140000'
>>> '{: f}; {: f}'.format(3.14, -3.14)  # show a space for positive numbers
' 3.140000; -3.140000'
>>> '{:-f}; {:-f}'.format(3.14, -3.14)  # show only the minus -- same as '{:f}; {:f}'
'3.140000; -3.140000'
```

替换 `%x` 和 `%o` 并将值转换为不同的进制：

``` python
>>> # format also supports binary numbers
>>> "int: {0:d};  hex: {0:x};  oct: {0:o};  bin: {0:b}".format(42)
'int: 42;  hex: 2a;  oct: 52;  bin: 101010'
>>> # with 0x, 0o, or 0b as prefix:
>>> "int: {0:d};  hex: {0:#x};  oct: {0:#o};  bin: {0:#b}".format(42)
'int: 42;  hex: 0x2a;  oct: 0o52;  bin: 0b101010'
```

使用逗号作为千位分隔符：

``` python
>>> '{:,}'.format(1234567890)
'1,234,567,890'
```

表示一个百分比：

``` python
>>> points = 19
>>> total = 22
>>> 'Correct answers: {:.2%}'.format(points/total)
'Correct answers: 86.36%'
```

使用特定于类型的格式：

``` python
>>> import datetime
>>> d = datetime.datetime(2010, 7, 4, 12, 15, 58)
>>> '{:%Y-%m-%d %H:%M:%S}'.format(d)
'2010-07-04 12:15:58'
```

嵌套参数和更复杂的例子：

``` python
>>> for align, text in zip('<^>', ['left', 'center', 'right']):
...     '{0:{fill}{align}16}'.format(text, fill=align, align=align)
...
'left<<<<<<<<<<<<'
'^^^^^center^^^^^'
'>>>>>>>>>>>right'
>>>
>>> octets = [192, 168, 0, 1]
>>> '{:02X}{:02X}{:02X}{:02X}'.format(*octets)
'C0A80001'
>>> int(_, 16)
3232235521
>>>
>>> width = 5
>>> for num in range(5,12): 
...     for base in 'dXob':
...         print('{0:{width}{base}}'.format(num, base=base, width=width), end=' ')
...     print()
...
    5     5     5   101
    6     6     6   110
    7     7     7   111
    8     8    10  1000
    9     9    11  1001
   10     A    12  1010
   11     B    13  1011
```