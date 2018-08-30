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