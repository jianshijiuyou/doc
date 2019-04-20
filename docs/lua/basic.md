# 安装

``` bash
sudo apt install lua5.3

sudo ln -s /usr/bin/lua5.3 /usr/bin/lua

$ lua -v
Lua 5.3.1  Copyright (C) 1994-2015 Lua.org, PUC-Rio
```

# Lua 语言入门

Hello World

``` lua
print("Hello World")
```

``` bash
$ lua hello.lua 
Hello World
```

定义函数体：计算一个数的阶乘

``` lua
function fact(n)
  if n==0 then
    return 1
  else
    return n * fact(n-1)
  end
end

print("enter a number:")
a = io.read("*n")
print(fact(a))
```

使用 `-i` 选项在运行完程序后进入交互模式

``` bash
$ lua -i hello.lua
Lua 5.3.1  Copyright (C) 1994-2015 Lua.org, PUC-Rio
enter a number:
3
6
> fact
function: 0x14c5660
> 
```

用 `dofile` 在运行时动态加载 lua 脚本

``` lua
dofile("lib1.lua")
```

注释

``` lua
-- 单行注释

--[[
    多行注释
]]

--[[
print(10)
--]]
```

重新启用注释中的代码，多加个连字符

``` lua
---[[
print(10)
--]]
```

语句之间的分隔符，换行符都不是必须的

``` lua
a = 1
b = a * 2

a = 1;
b = a * 2;

a = 1; b = a * 2

a = 1  b = a * 2
```

全局变量无需声明即可使用

``` lua
> b
nil
> b = 10
> b
10
```

?> 当把 `nil` 赋值给全局变量时，Lua 会最终回收该变量占用的内存

Lua 是动态类型语言

Lua 有 8 种基本类型：`nil`（空）、`boolean`（布尔）、`number`（数值）、`string`（字符串）、`userdata`（用户数据）、`function`（函数）、`thread`（线程）、`table`（表）。

使用 `type` 可以检查一个值的类型

``` lua
> type(nil)
nil
> type(true)
boolean
> type(10)
number
> type("hello")
string
> type(io.stdin)
userdata
> type(print)
function
> type(type) 
function       -- 《Lua 程序设计》第四版上返回的是 thread
> type({})
table
> type(type(X))
string
> type(X)
nil
```

?> `type` 返回的永远都是字符串

Lua 在条件测试的时候将除 `false` 和 `nil` 以外的值都视为真

!> Lua 将零和空字符串都视为真

逻辑运算符： `and`、`or`、`not`

``` lua
> 4 and 5           --> 5
> nil and 13        --> nil
> false and 13      --> false
> 0 or 5            --> 0
> false or "hi"     --> hi
> nil or false      --> false
```

!> 都只会判断第一个操作数，使用 `and` 时，如果第一个操作数为假，则返回第一个操作数，否则返回第二个。使用 `or` 时，如果第一个不为假，则返回第一个，否则返回第二个。

?> `not` 永远返回 `bollean` 类型

解释器参数

``` lua
print(arg[-3])
print(arg[-2])
print(arg[-1])
print(arg[0])
print(arg[1])
print(arg[2])
```

结果

``` bash
$ lua -e "a=1" hello.lua 12 "abc"
lua
-e
a=1
hello.lua
12
abc
```

# 数值

Lua 5.3 开始有整型，在这之前所有数值都是双精度浮点型。

整型和浮点型的类型都是 `number`

``` lua
> type(3)
number
> type(3.5)
number
> 
```

相同算术值结果相同

``` lua
> 1 == 1.0
true
> -3 == -3.0
true
> 0.2e3 == 200
true
> 
```

如果你硬要区分类型

``` lua
> math.type(3)
integer
> math.type(3.0)
float
```

通过 `%a` 参数格式化输出

``` lua
> string.format("%a", 123)
0x1.ecp+6
> string.format("%a", 12.3)
0x1.899999999999ap+3
```

算术运算

``` lua
> 13 + 15
28
> 12.3 + 12.7
25.0
> 12.5 + 3
15.5
> 4 / 2       -- 除法永远返回浮点数
2.0
> 3 // 2
1
> 3.0 // 2    -- floor 除法
1.0
> 9 % 2       -- 取模
1
```

关系运算

``` lua
<   >   <=  >=  ==  ~=
```

`==` 用于相等性测试，`~=` 用于不等性测试

?> 这两个运算符可以应用于任意两个值，当这两个值的类型不同时，Lua 则认为它们不相等

数学库

``` lua
> math.pi
3.1415926535898
> math.huge
inf
> math.max(1,2,3)
3
> math.min(1,2,3)
1
> math.sin(90)
0.89399666360056
> math.deg(0.8)
45.836623610466
> math.random()         -- [0, 1)
0.84018771676347
> math.random()
0.39438292663544
> math.random(10)       -- [0, 10]
7
> math.random(10,100)   -- [10, 100]
32
```

取整

``` lua
> math.floor(3.3)
3
> math.ceil(3.3)
4
> math.modf(3.3)
3	0.3
```

无偏取整函数

``` lua
function round(x)
  local f = math.floor(x)
  if (x == f) or (x % 2.0 == 0.5) then
    return f
  else
    return math.floor(x + 0.5)
  end
end

> round(2.5)
2
> round(3.5)
4
> round(-2.5)
-2
> round(-1.5)
-2
```

整数范围

``` lua
> math.maxinteger
9223372036854775807
> math.mininteger
-9223372036854775808
> math.maxinteger + 1
-9223372036854775808
> math.maxinteger + 1 == math.mininteger
true
```

整数和浮点数转换

``` lua
> 123123213 + 0.0
123123213.0
> 2^10
1024.0
> 2^10 | 0
1024
> math.tointeger(2.0)
2
> math.tointeger(2.1)   -- 无法转化小数部分不是零的
nil
```
# 字符串

Lua 中字符串时不可变的

改变字符串的某些部分会产生新的字符串

``` lua
> a = "one string"
> b = string.gsub(a, "one", "another")
> a
one string
> b
another string
```

通过 `#` 获取字符串的长度 

``` lua
> a = "hello"
> #a
5
> #"good bye"
8
> #"哈哈"   --> 一个中文占三个字节
6
```

字符串拼接

``` lua
> "hello" .. "world"
helloworld
> "result is " .. 3
result is 3
```

多行字符串

``` lua
page = [[
  <html>
    <head>
    </head>
    <body>
    </body>
  </html>
]]
```

?> 第一个换行符会被忽略

类型转换，`string` to `number`

``` lua
> tonumber("   -3  ")
-3
> tonumber("   10e4  ")
100000.0
> tonumber("20e")
nil
> tonumber("0x1.3p-4")
0.07421875
> tonumber("10010101", 2)
149
> tonumber("ffff", 16)
65535
```

类型转换，`number` to `string`

``` lua
> tostring(10) == "10"
true
```

字符串标准库

``` lua
> string.len("abcde")
5
> string.rep("a", 10)
aaaaaaaaaa
> string.reverse("abcde")
edcba
> string.upper("abcde")
ABCDE
> string.lower("ABCDE")
abcde
```

子串

``` lua
> s = "[in brackets]"
> string.sub(s, 2, -2)
in brackets
> string.sub(s, 1, 1)
[
> string.sub(s, -1, -1)
]
```

`char` <==> `byte`

``` lua
> string.char(97, 98, 99)
abc
> string.byte("abc")
97
> string.byte("abc", 1)
97
> string.byte("abc", 2)
98
> string.byte("abc", 3)
99
> string.byte("abc", 1, 3)
97	98	99
> a, b, c = string.byte("abc", 1, 3)
> a
97
> b
98
> c
99
```

字符串格式化

``` lua
> string.format("x = %d y = %d", 10, 20)
x = 10 y = 20
> string.format("x = %x", 10)
x = a
> string.format("x = 0x%X", 100)
x = 0x64
> string.format("x = %f", 100)
x = 100.000000
> tag, title = "h1", "a title"
> string.format("<%s>%s</%s>", tag, title, tag)
<h1>a title</h1>
```

``` lua
> string.format("%.2f", math.pi)
3.14
> string.format("%02d", 5)
05
> string.format("%03d", 5)
005
> string.format("%10d", 5)
         5
```

调用标准库的快捷方式

``` lua
> s = "abc"
> s:len()
3
> s:upper()
ABC
```

字符串查找搜索

``` lua
> string.find("hello world", "wor")
7	9
> string.find("hello world", "war")
nil
```

字符串替换

``` lua
> string.gsub("hello world", "l", ".")
he..o wor.d	3
> string.gsub("hello world", "ll", "..")
he..o world	1
> string.gsub("hello world", "a", ".")
hello world	0
```

Unicode 编码

``` lua
> utf8.len("哈哈")
2
> utf8.len("哈哈asd")
5
> utf8.codepoint("哈")
21704
> utf8.char(21704)
哈

```

# 表

``` lua
> a = {}
> a
table: 0x11822d0
> k = "x"
> a[k] = 10
> a
table: 0x11822d0
> a[20] = "great"
> a["x"]
10
> k = 20
> a[k]
great
> a["x"] = a["x"] + 1
> a["x"]
11

```

``` lua
> a = {}
> a.x = 10
> a["x"]
10
> a.y
nil
> a.x
10
```

表构造器

当数组使用，不过下标从 1 开始

``` lua
> let = {"a", "b", "c"}
> let[1]
a
> let[2]
b
> let[3]
```

当字典使用

``` lua
> a = {x=10, y=20}
> a.x
10
> a['y']
20
```

混合

``` lua
> a = {x=10, y=20, "a", "b"}
> a[1]
a
> a[2]
b
> a.x
10
> a.y
20
```

``` lua
> a = {"+"=10}
stdin:1: '}' expected near '='
> a = {["+"]=10}
> a["+"]
10
```

获取表的长度(不可靠)

``` lua
> let = {"a", "b", "c"}
> #let
3
> let[#let]
c
> let[#let + 1] = "d"
> let[#let]
d

> a = {}
> a[1] = 1
> a[10000] = 10000
> #a      
1       --> 很尴尬

> a = {1, 2, 3, nil, nil}
> #a
3       --> 很尴尬
> a = {1, 2, 3, nil, nil, 4}
> #a
6       --> 很尴尬
```

遍历表

``` lua
> a = {"a", "b", "c", nil, nil, "d"}
> for k, v in pairs(a) do
    print(k, v)
  end
1	a
2	b
3	c
6	d
```

当表是字典时可以用 `pairs` 遍历，当表是数组时可以用 `ipairs` 遍历。

?> 如果时列表，会按照顺序遍历，如果时字典，则是随机的

表标准库

``` lua
> a = {10, 20, 30}
> table.insert(a, 5)        --> 10, 20, 30, 5
> table.insert(a, 1, 50)    --> 50, 10, 20, 30, 5

> table.remove(a, 1)        --> 删除并返回指定元素
50
> table.remove(a)           --> 删除最后一个并返回
5
```

# 函数

定义个求和函数

``` lua
function add(a)
  local sum = 0
  for i = 1, #a do
    sum = sum + a[i]
  end
  return sum
end

a = {10, 20, 30}
result = add(a)
print(result)
```

Lua 会通过抛弃多余的参数和将不足的参数设置为 `nil` 来调整参数的个数

``` lua
> function f(a, b) print(a, b) end
> f()
nil	nil
> f(3)
3	nil
> f(3, 4)
3	4
> f(3, 4, 5)
3	4
```

函数可以返回多个结果

``` lua
> function f() return "one", "two" end
> a, b = f()
> a
one
> b
two
```

`()` 会限制返回结果

``` lua
> f()
one	two
> (f())
one
```

可变长参数

``` lua
> function f(...) return ... end
> f(1, 2, 3)
1	2	3
> function f(...) return {...} end
> f(1, 2, 3)
table: 0x1c0e2d0
> function f(...) local a, b, c = ...; print(a, b, c) end
> f(1, 2, 3)
1	2	3
```

判断可变参数中是否有 `nil`

``` lua
function nonils(...)
  local arg = table.pack(...)
  for i=1, arg.n do
    if arg[i] == nil then
      return false
    end
  end
  return true
end

print(nonils(2, 3, nil))    --> false
print(nonils(2, 3))         --> true
print(nonils())             --> true
print(nonils(nil))          --> false
```

使用 `select` 函数遍历

``` lua
> select(1, "a", "b", "b")
a	b	b
> select(2, "a", "b", "b")
b	b
> select(3, "a", "b", "b")
b
> select(1, "a", nil, "b")
a	nil	b
> select("#", "a", nil, "b")
3
> select("#", "a", nil, "b", nil)
4
```

`table.unpack`

``` lua
> type(table.pack(1,2))
table
> type(table.unpack({1}))
number
> type(table.unpack({1, 2}))
number
> table.unpack({1, 2})
1	2
> table.unpack{1, 2, 3}
1	2	3

> table.unpack({10,20,30}, 2, 3)
20	30
```

# 输入输出

``` lua
f = io.open("test.txt", "r")
txt = f:read("a") --> 全部读出
print(txt)
f:close()
```

# 补充知识

局部变量

Lua 中的变量默认都是全局变量，局部变量必须单独声明

``` lua
while i <= x do
  local x = i * 2     --> 单独声明，否则为全局变量
  print(x)
  i = i + 1
end
```

`if` 语句

``` lua
if op == "+" then
  r = a + b
elseif op == "-" then
  r = a - b
else
  error("invalid operation")
end
```

`while` 语句

``` lua
local i = 1
while a[i] do
  print(a[i])
  i = i + 1
end
```

`repeat` 语句

``` lua
local line
repeat
  line = io.read()
until line ~= ""
print(line)
```

数值型 `for`

```
for var = start, end, step do
  something
end
```

`step`（步长）可以省略，默认为 1

``` lua
for i = 1, 10, 2 do
  print(i)
end
```

`goto` 语句

``` lua
while xxxx do
  ::redo::
  if yyyy then
    goto continue
  elseif zzzz then
    goto redo
  some code
  ::continue::
end
```

Lua 在条件测试的时候将除 `false` 和 `nil` 以外的值都视为真

?> Lua 将 `0` 和 “空字符串” 都视为真