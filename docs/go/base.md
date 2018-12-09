# 图谱

## 学习路线

https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/c702df29da67be3c4083ecce1d0eadb7.png

## 知识图表

https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/add8566dc5431378bda313a32a6ebb85.jpg

# 程序结构

## 关键字

```
break      default       func     interface   select
case       defer         go       map         struct
chan       else          goto     package     switch
const      fallthrough   if       range       type
continue   for           import   return      var
```

## 内置常量、类型、函数

内建常量

```
true false iota nil
```

内建类型

```
int int8 int16 int32 int64
uint uint8 uint16 uint32 uint64 uintptr
float32 float64 complex128 complex64
bool byte rune string error
```

内建函数

```
make len cap new append copy close delete
complex real imag
panic recover
```

## 声明

Go 语言主要有四种类型的声明语句：var、const、type 和 func，分别对应变量、常量、类型和函数实体对象的声明。

## 变量

一般语法

``` go
var 变量名字 类型 = 表达式

// 比如

var name string = "jack"
```

其中“类型”或“= 表达式”两个部分可以省略其中的一个。如果省略的是类型信息，那么将根据初始化表达式来推导变量的类型信息。如果初始化表达式被省略，那么将用零值初始化该变量。

1. **数值**类型变量对应的零值是 `0`
2. **布尔**类型变量对应的零值是 `false`
3. **字符串**类型对应的零值是 `空字符串`
4. **接口或引用**类型（包括 `slice`、`指针`、`map`、`chan` 和`函数`）变量对应的零值是 `nil` 。
4. **数组或结构体**等聚合类型对应的零值是每个元素或字段都是对应该类型的零值。

``` go
var i, j, k int                 // int, int, int
var b, f, s = true, 2.3, "four" // bool, float64, string
```

?> 初始化表达式可以是字面量或任意的表达式。在包级别声明的变量会在 main 入口函数执行前完成初始化，局部变量将在声明语句被执行到的时候完成初始化。

一组变量也可以通过调用一个函数，由函数返回的多个返回值初始化：

``` go
var f, err = os.Open(name) // os.Open returns a file and an error
```

### 简短变量声明

``` go
anim := gif.GIF{LoopCount: nframes}
freq := rand.Float64() * 3.0
t := 0.0
```

``` go
i := 100                  // an int
var boiling float64 = 100 // a float64
var names []string
var err error
var p Point
```

``` go
i, j := 0, 1
```

!> 请记住 `:=` 是一个变量声明语句，而 `=` 是一个变量赋值操作。

``` go
i, j = j, i // 交换 i 和 j 的值
```

### 指针

``` go
x := 1
p := &x         // p, of type *int, points to x
fmt.Println(*p) // "1"
*p = 2          // equivalent to x = 2
fmt.Println(x)  // "2"
```

?> 任何类型的指针的零值都是 `nil` 。如果 `p` 指向某个有效变量，那么 `p != nil` 测试为真。指针之间也是可以进行相等测试的，只有当它们指向同一个变量或全部是 `nil` 时才相等。

``` go
var x, y int
fmt.Println(&x == &x, &x == &y, &x == nil) // "true false false"
```

### new 函数

另一个创建变量的方法是调用内建的 `new` 函数。表达式 `new(T)` 将创建一个 `T` 类型的匿名变量，初始化为 `T` 类型的零值，然后返回 **变量地址**，返回的指针类型为 `*T`。

``` go
p := new(int)   // p, *int 类型, 指向匿名的 int 变量
fmt.Println(*p) // "0"
*p = 2          // 设置 int 匿名变量的值为 2
fmt.Println(*p) // "2"
```

!> `new` 函数类似是一种语法糖，而不是一个新的基础概念。

下面的两个 newInt 函数有着相同的行为：

``` go
func newInt() *int {
    return new(int)
}

func newInt() *int {
    var dummy int
    return &dummy
}
```

## 赋值

``` go
x = 1                       // 命名变量的赋值
*p = true                   // 通过指针间接赋值
person.name = "bob"         // 结构体字段赋值
count[x] = count[x] * scale // 数组、slice或map的元素赋值
// or
count[x] *= scale
```

```
v := 1
v++    // 等价方式 v = v + 1；v 变成 2
v--    // 等价方式 v = v - 1；v 变成 1
```

### 元组赋值

交换两个变量的值

``` go
x, y = y, x

a[i], a[j] = a[j], a[i]
```

### 可赋值性

``` go
medals := []string{"gold", "silver", "bronze"}
```

``` go
medals[0] = "gold"
medals[1] = "silver"
medals[2] = "bronze"
```

## 类型

``` go
type 类型名字 底层类型
```

``` go
type Celsius float64    // 摄氏温度
type Fahrenheit float64 // 华氏温度

const (
    AbsoluteZeroC Celsius = -273.15 // 绝对零度
    FreezingC     Celsius = 0       // 结冰点温度
    BoilingC      Celsius = 100     // 沸水温度
)

func CToF(c Celsius) Fahrenheit { return Fahrenheit(c*9/5 + 32) }

func FToC(f Fahrenheit) Celsius { return Celsius((f - 32) * 5 / 9) }
```

``` go
var c Celsius
var f Fahrenheit
fmt.Println(c == 0)          // "true"
fmt.Println(f >= 0)          // "true"
fmt.Println(c == f)          // compile error: type mismatch
fmt.Println(c == Celsius(f)) // "true"!
```

## 包和文件

在 Go 语言中，一个简单的规则是：如果一个名字是大写字母开头的，那么该名字是导出的。

### 包的初始化

包中内容初始化顺序

``` go
var a = b + c // a 第三个初始化, 为 3
var b = f()   // b 第二个初始化, 为 2, 通过调用 f (依赖c)
var c = 1     // c 第一个初始化, 为 1

func f() int { return c + 1 }
```

可以用一个特殊的 init 初始化函数来简化初始化工作。每个文件都可以包含多个 init 初始化函数

``` go
func init() { /* ... */ }
```

这样的 init 初始化函数除了不能被调用或引用外，其他行为和普通函数类似。在每个文件中的 init 初始化函数，在程序开始执行时按照它们声明的顺序被自动调用(也不一定)。

?> 每个包在解决依赖的前提下，以导入声明的顺序初始化，每个包只会被初始化一次。因此，如果一个p包导入了q包，那么在p包初始化的时候可以认为q包必然已经初始化过了。初始化工作是自下而上进行的，main包最后被初始化。以这种方式，可以确保在main函数执行之前，所有依赖的包都已经完成初始化工作了。