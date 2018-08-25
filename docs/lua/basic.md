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