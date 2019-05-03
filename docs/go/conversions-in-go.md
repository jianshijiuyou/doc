> [原文链接](https://medium.com/golangspec/conversions-in-go-4301e8d84067)

# Conversions in Go

有时可能需要将值转换为其他类型。Golang 不允许以任意方式执行此操作。它们是由类型系统强制执行的某些规则。在这个故事中，我们将看到哪种转换是可能的，哪些转换不是，转换实际上有用。

Go 是强类型编程语言。

它在类型上是严格的，在编译期间会报告错误：

``` go
package main
import "fmt"
func main() {
    monster := 1 + "2"
    fmt.Printf("monster: %v\n", monster)
}
> go build
# github.com/mlowicki/lab
./lab.go:6: cannot convert "2" to type int
./lab.go:6: invalid operation: 1 + "2" (mismatched types int and string)
```

JavaScript 是弱（松散）类型语言之一。让我们看看它的实际效果：

``` javascript
var monster = 1 + "foo" + function() {};
console.info("type:", typeof monster)
console.info("value:", monster);
```

将数字，字符串甚至函数加在一起可能看起来很奇怪。不用担心，JavaScript会在没有任何抱怨的情况下为您完成：

```
type: string
value: 1foofunction () {}
```

在某些情况下，可能需要将值转换为其他类型，将它作为参数传递或者将其放入表达式中：

``` go
func f(text string) {
    fmt.Println(text)
}
func main() {
    f(string(65))  // integer constant converted to string
}
```

函数调用中的表达式是转换的一个示例，我们将在下面看到更多。上面的代码可以正常工作，但删除转换后：

``` go
f(65)
```

导致编译时错误：“cannot use 65 (type int) as type string in argument to f”

# 底层类型

底层类型的 string，boolean，numeric 或 literal 类型是相同的类型。否则它是类型声明的基础类型：

``` go
type A string             // string
type B A                  // string
type C map[string]float64 // map[string]float64 (type literal)
type D C                  // map[string]float64
type E *D                 // *D
```

如果基础类型相同，则转换 100％ 有效：

``` go
package main
type A string
type B A
type C map[string]float64
type C2 map[string]float64
type D C
func main() {
    var a A = "a"
    var b B = "b"
    c := make(C)
    c2 := make(C2)
    d := make(D)
    a = A(b)
    b = B(a)
    c = C(c2)
    c = C(d)
    var _ map[string]float64 = map[string]float64(c)
}
```

没有什么可以阻止编译这样的程序。底层类型的定义不是递归的：

``` go
type S string
type T map[S]float64
...
var _ map[string]float64 = make(T)
```

它给出了编译时错误：

```
cannot use make(T) (type T) as type map[string]float64 in assignment
```

这是因为T的底层类型不是 `map[string]float64` 而是 `map[S]float64`。转换既不会起作用：

``` go
var _ map[string]float64 = (map[string]float64)(make(T))
```

它在构建时触发错误：

```
cannot convert make(T) (type T) to type map[string]float64
```

# 可转换性

Go 规范了可转换性。它定义了值 v 何时可赋值给类型为 T 的变量。让我们看一下它的一个规则，当至少有一个类型没有被命名时，它会说明相同的底层类型：

``` go
package main
import "fmt"
func f(n [2]int) {
    fmt.Println(n)
}
type T [2]int
func main() {
    var v T
    f(v)
}
```

该程序打印 “[0 0]”。在可转换性规则允许的所有情况下都可以进行转换。这样程序员可以明确地标记他/她的意识决定：

``` go
f([2]int(v))
```

使用上面的调用给出与以前相同的结果。

第一个转换规则（相同的基础类型）与其中一个可赋值规则重叠 -- 当基本类型是相同的，并且至少一种类型是（在本节第一示例）命名。较弱的规则会影响更严格的规则。所以最后转换只有底层类型需要相同 - 命名/未命名类型无关紧要。

# 常量

当 v 可由类型 T 的值表示时，常量 v 可以转换为类型 T：

``` go
a := uint32(1<<32 – 1)
//b := uint32(1 << 32) // constant 4294967296 overflows uint32
c := float32(3.4e38)
//d := float32(3.4e39) // constant 3.4e+39 overflows float32
e := string("foo")
//f := uint32(1.1) // constant 1.1 truncated to integer
g := bool(true)
//h := bool(1) // convert 1 (type int) to type bool
i := rune('ł')
j := complex128(0.0 + 1.0i)
k := string(65)
```

[官方博客](https://blog.golang.org/constants)上提供了对常量的深入介绍。

# 数字类型

## floating-point number → integer

``` go
var n float64 = 1.1
var m int64 = int64(n)
fmt.Println(m)
```

删除小数部分，以便代码打印“1”。

对于其他转换：

* floating-point number → floating-point number,
* integer → integer,
* integer → floating-point number,
* complex → complex.

value 四舍五入到目标精度：

``` go
var a int64 = 2 << 60
var b int32 = int32(a)
fmt.Println(a)         // 2305843009213693952
fmt.Println(b)         // 0
a = 2 << 30
b = int32(a)
fmt.Println(a)         // 2147483648
fmt.Println(b)         // -2147483648
b = 2 << 10
a = int64(b)
fmt.Println(a)         // 2048
fmt.Println(b)         // 2048
```

## Pointers

可转换性可以像处理其他类型文字一样处理指针类型：

``` go
package main
import "fmt"
type T *int64
func main() {
    var n int64 = 1
    var m int64 = 2
    var p T = &n
    var q *int64 = &m
    p = q
    fmt.Println(*p)
}
```

程序运行正常并依据已经涵盖的可分配性规则打印“2”。`*int64` 和 T 的基础类型是相同的，另外 `*int64` 是未命名的。转换增加了更多选项。对于未命名的指针类型，基本类型的指针具有相同的基础类型就足够了：

``` go
package main
import "fmt"
type T int64
type U W
type W int64
func main() {
    var n T = 1
    var m U = 2
    var p *T = &n
    var q *U = &m
    p = (*T)(q)
    fmt.Println(*p)
}
```

?> `*T` 被括号括起来，否则它将被解释为 `*(T(q))`。

与之前类似，程序打印“2”。U 和 T 的基础类型是相同的，它们在 `*U` 和 `*T` 中用作基类型。可转换性：

``` go
p = q
```

将无法工作，因为它处理两种不同的基础类型： `*T` 和 `*U` ，作为练习，让我们稍微改变声明时会发生什么：

``` go
type T *int64
type U *W
type W int64
func main() {
    var n int64 = 1
    var m W = 2
    var p T = &n
    var q U = &m
    p = T(q)
    fmt.Println(*p)
}
```

U 和 W 的声明已经改变。想一想会发生什么......

编译器在以下位置指出错误 “cannot convert q (type U) to type T”：

``` go
p = T(q)
```

这是因为 p 的类型的基础类型是 `*int64` 但是对于 q 的类型它是 `*W`。q 的类型命名为（U），因此关于获取指针基类型的基础类型的规则不适用于此处。

# Strings

## integer → string

将数字 N 传递给内置字符串将其转换为包含由 N 表示的符号的 UTF-8 编码字符串。

``` go
fmt.Printf("%s\n", string(65))
fmt.Printf("%s\n", string(322))
fmt.Printf("%s\n", string(123456))
fmt.Printf("%s\n", string(-1))
```

output:

``` go
A
ł
�
�
```

两次首次转换使用完全有效的代码点。您可能想知道最后两行显示的奇怪符号是什么。它是一个替换字符，它是 Unicode 块的成员，称为 specials。它的代码是 `\uFFFD`

## Short introduction to strings

字符串基本上是字节的片段：

``` go
text := "abł"
for i := 0; i < len(text); i++ {
    fmt.Println(text[i])
}
```

output:

``` go
97
98
197
130
```

97 和 98 是 UTF-8 编码的 “a” 和 “b” 字符。第 3 行和第 4 行呈现字母 “ł” 的编码，其在 UTF-8 中占用两个字节。

范围循环有助于迭代 Unicode 定义的代码点（Go 中的代码点称为 rune）：

``` go
text := "abł"
for _, s := range text {
    fmt.Printf("%q %#v\n", s, s)
}
```

output:

``` go
'a' 97
'b' 98
'ł' 322
```

?> 要了解有关 `％q` 或 `％#v` 等格式动词的更多信息，请参阅 [fmt](https://golang.org/pkg/fmt/) 软件包的文档。

[“Go中的字符串，字节，符文和字符”](https://blog.golang.org/strings)中的更多信息。在快速解释之后，字符串和字节或符文切片之间的转换不再那么奇怪。

## string ↔ slice of bytes

``` go
bytes := []byte("abł")
text := string(bytes)
fmt.Printf("%#v\n", bytes) // []byte{0x61, 0x62, 0xc5, 0x82}
fmt.Printf("%#v\n", text)  // "abł"
```

Slice 包含 UTF-8 转换字符串的字节。

## string ↔ slice of runes

``` go
runes := []rune("abł")
fmt.Printf("%#v\n", runes)         // []int32{97, 98, 322}
fmt.Printf("%+q\n", runes)         // ['a' 'b' '\u0142']
fmt.Printf("%#v\n", string(runes)) // "abł"
```

从转换后的字符串创建的切片包含 Unicode 代码点（runes）