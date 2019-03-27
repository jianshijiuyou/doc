> [原文地址](https://www.ibm.com/developerworks/aix/library/au-unixtext/index.html)

# 基于UNIX系统的文本操作简介

UNIX 哲学的基本原则是创建做一件事的程序（或进程），并做好一件事。要求仔细考虑接口和将这些较小（希望更简单）的过程结合在一起以创建有用结果的方法是一种哲学。通常，文本数据在这些接口之间流动。随着时间的推移，已经开发出越来越多的高级文本处理工具和语言。对于语言，早期有 perl，后来是 python，还有 ruby。虽然这些和其他语言是非常强大的文本处理器，但这些工具并不总是可用，尤其是在生产环境中。在本文中，将演示许多基本的 UNIX 文本处理命令，这些命令可以单独使用，也可以相互结合使用，以解决可能使用较新语言解决的问题。对于许多人来说，一个例子提供的信息比冗长的解释更多。请注意，由于可用的 UNIX 和 UNIX-like 系统种类繁多，命令标志，程序行为和输出在实现之间有所不同。

## cat

`cat` 命令是最基本的命令之一。它用于创建，追加，显示和连接文件。

我们可以使用 `>` 创建一个带有 cat 的文件，将标准输入（stdin）重定向到文件。使用 `>` 运算符会截断指定的输出文件的内容。之后输入的文本将重定向到 `>` 运算符右侧指定的文件。control-d 发出文件结束信号，将控制权返回给 shell。


`cat` 创建文件的示例：

``` bash
$ cat > grocery.list
apples
bananas
plums
<ctrl-d>
$
```

使用 `>>` 运算符将标准输入附加到现有文件中。

``` bash
$ cat >> grocery.list
carrots
<ctrl-d>
```

使用不带标志的 `cat` 检查 `grocery.list` 文件的内容。请注意文件内容如何包含重定向和追加运算符示例的输入。

没有标志的 `cat` 的例子：

``` bash
$ cat grocery.list
apples
bananas
plums
carrots
```

`cat` 命令可用于对文件的行进行编号。

`cat` 计数行的示例：

``` bash
$ cat -n grocery.list
     1  apples
     2  bananas
     3  plums
     4  carrots
```

## nl

`nl` 过滤器从 stdin 或指定文件中读取行。输出写入 stdout ，可以重定向到文件或通过管道传输到另一个进程。 `nl` 的行为通过各种命令行选项来控制。

默认情况下， `nl` 计算与 `cat -n` 类似的行。

`nl` 的默认用法示例：

``` bash
$nl grocery.list
     1  apples
     2  bananas
     3  plums
     4  carrots
```

使用 `-b` 标志指定要编号的行。该标志将其参数作为“类型”。该类型告诉 `nl` 哪些行需要编号 -- 使用 `a` 对所有行进行编号，`t` 告诉 `nl` 不对空行或只有空格的行进行编号，`n` 指定不对行进行编号。在该示例中，示出了用于模式的 `p` 类型。 `nl` 为正则表达式模式指定的行编号，在本例中为以字母 “a” 或 “b” 开头的行。

符合正则表达式的 `nl` 到数字行的示例：

``` bash
$ nl -b p^[ba] grocery.list
     1  apples
     2  bananas
       plums
       carrots
```

默认情况下， `nl` 使用 tab 将行号与文本分开。使用 `-s` 指定不同的分隔符，例如 “=” 符号。

用于指定分隔符的 `nl` 示例：

``` bash
$nl –s= grocery.list
     1=apples
     2=bananas
     3=plums
     4=carrots
```

## wc

wc（wordcount）命令计算行数，单词（由空格分隔）和指定文件中的字符，或来自 stdin 。

wc 用法示例：

``` bash
$wc grocery.list
       4       4      29 grocery.list
$wc -l grocery.list
       4 grocery.list
$wc -w grocery.list
       4 grocery.list
$wc -c grocery.list
      29 grocery.list
```

## grep

`grep` 命令在指定的文件或 stdin 中搜索与给定表达式匹配的模式。

`grep` 的输出由各种选项标志控制。

为了演示，创建了一个与 `grocery.list` 一起使用的新文件。

``` bash
$cat grocery.list2
Apple Sauce
wild rice
black beans
kidney beans
dry apples
```

基本 `grep` 用法示例：

``` bash
$ grep apple grocery.list grocery.list2
grocery.list:apples
grocery.list2:dry apples
```

`grep` 有相当多的选项标志。以下是一些示例，演示了一些选项的用法。

要显示具有找到模式的行数的文件名（如果有多个文件） - 在这种情况下，请计算每个文件中出现 “apple” 一词的行数。

`grep` 示例 - 计算文件中的匹配数：

``` bash
$ grep -c apple grocery.list grocery.list2
grocery.list:1
grocery.list2:1
```

搜索多个文件时，使用 `-h` 选项会禁止将文件名作为输出的一部分打印。

`grep` 的示例 - 在输出中抑制文件名：

``` bash
$ grep -h apple grocery.list grocery.list2
apples
dry apples
```

在许多情况下，需要不区分大小写的搜索。 `grep` 命令具有 `-i` 选项，可在执行搜索时忽略区分大小写。

`grep` 的示例 - 不区分大小写：

``` bash
$ grep -i apple grocery.list grocery.list2
 
grocery.list:apples
grocery.list2:Apple Sauce
grocery.list2:dry apples
```

有时，只需要打印文件名，而不是模式匹配行。 `grep` 提供 `-l` 选项，仅打印包含具有匹配模式的行的文件名。

`grep` 的示例 - 仅文件名：

``` bash
$ grep -l carrot grocery.list grocery.list2
grocery.list
```

行号可以作为输出的一部分提供。使用 `-n` 选项包含行号。

`grep` 的示例 - 包括行号：

``` bash
$ grep -n carrot grocery.list grocery.list2
grocery.list:4:carrots
```

有时，与模式不匹配的行是所需的输出。在这种情况下使用 `-v` 选项。

`grep` 示例 - 反向匹配：

``` bash
$ grep -v beans grocery.list2
Apple Sauce
wild rice
dry apples
```

有时，所需的模式形成一个由空格或其他字符（如破折号或圆括号）包围的“单词”。大多数版本的 `grep` 都提供了-w选项，以便于编写对这些模式的搜索。

`grep` 示例 - 单词匹配：

``` bash
$ grep -w apples grocery.list grocery.list2
grocery.list:apples
grocery.list2:dry apples
```

## Streams, pipes, redirects, tee, here docs

在 UNIX 中，默认情况下，终端包含三个流，一个用于输入，两个基于输出的流。输入流被称为 stdin ，并且通常被映射到键盘（可以使用其他输入设备，或者可以从另一个进程管道输入）。标准输出流称为 stdout ，通常打印到终端，或输出可能被另一个进程（如 stdin ）消耗。主要用于状态报告的另一个输出流 stderr 通常像 stdout 一样打印到终端。由于这些流中的每一个都有自己的文件描述符，即使它们都连接到终端，每个流也可以与另一个分开管道或重定向。每个流的文件描述符是：

* stdin = 0
* stdout = 1
* stderr = 2

这些流可以通过管道传输并重定向到文件或其他进程。此构造通常称为构建管道。例如，程序员可能想要合并 stdout 和 stderr 流，然后在终端上显示它们，但也将结果保存到文件中以检查构建问题。使用 `2>＆1`，带有文件描述符 `2` 的 stderr 流被重定向到 `＆1` ，一个 '指针' 到 stdout 流。这有效地将 stderr 合并到 stdout 中。使用 `|` 符号表示管道。管道将 stdout 从左侧进程（make）链接到右侧进程（tee）的 stdin。`tee` 命令复制（合并的）stdout 流，将数据发送到终端和文件，在本例中称为 build.log。

合并和拆分标准流的示例：

``` bash
$ make –f build_example.mk 2>&1 | tee build.log
```

另一个重定向示例是使用 `cat` 命令和一些流重定向来创建文本文件的副本。

``` bash
$ cat < grocery.list > grocery.list.bak
```

之前使用 nl 命令将行号添加到 stdout 上显示的文件中。

管道可用于将 stdout 流（从`cat grocery.list`）发送到另一个进程，在本例中为 `nl` 命令。

``` bash
$ cat grocery.list | nl
     1  apples
     2  bananas
     3  plums
     4  carrots
```

前面显示的另一个示例是对模式的文件进行不区分大小写的搜索。这可以使用重定向来完成 - 在本例中是从 stdin 或使用管道，类似于上面的简单管道示例。

`grep` - stdin 重定向和管道示例：

``` bash
$ grep -i apple < grocery.list2
Apple Sauce
dry apples
$ cat grocery.list2 | grep -i apple
Apple Sauce
dry apples
```

在某些情况下，文本块将作为脚本的一部分重定向到命令或文件中。实现此目的的机制是使用 “here document” 或 “here-doc”。要将 here-doc 嵌入到脚本中，`<<` 运算符用于重定向以下文本，直到到达文件结束分隔符。分隔符在 `<<` 运算符后指定。

``` bash
$ cat << EOF
> oranges
> mangos
> pinapples
> EOF
oranges
mangos
pinapples
```

此输出可以重定向到文件，在此示例中，分隔符从 “EOF” 更改为 “!”。然后使用 tr 命令（稍后解释）用 here-doc 大写字母。

示例基本 here-doc 重定向到文件：

``` bash
cat << ! > grocery.list3
oranges
mangos
pinapples
!
$ cat grocery.list3
oranges
mangos
pinapples
$ tr [:lower:] [:upper:] << !
> onions
> !
ONIONS
```

## head 和 tail

`head` 和 `tail` 命令用于检查文件的顶部（头部）或底部（尾部）部分。要显示文件的前两行和后两行，请分别使用 `-n` 选项标志和这些命令。同样，`-c` 选项显示文件中的第一个或最后一个字符。

``` bash
$ head -n2 grocery.list
apples
bananas
$ tail -n2 grocery.list
plums
carrots
$ head -c12 grocery.list # 显示文件前 12 个字符
apples
banan
$ tail -c12 grocery.list
ums
carrots
```

`tail` 命令的一个常见用途是查看日志文件或正在运行的进程的输出以查看是否存在问题，或者注意进程何时完成。`-f`（`tail -f`）选项使得 tail 继续观察流，即使在到达文件结束标记之后，并且当流包含更多数据时继续显示输出。

## tr

`tr` 命令用于从 stdin 转换字符，在 stdout 上显示它们。在其一般形式中， `tr` 采用两组字符，并将第一组中的字符替换为第二组中的字符。 `tr` 和其他一些命令可以使用许多预定义的字符类（集）。

* alnum - 字母数字字符
* alpha - 字母字符
* blank - 空白字符
* cntrl - 控制字符
* digit - 数字字符
* graph - 图形字符
* lower - 小写字母字符
* print - 可打印的字符
* punct - 标点字符
* space - 空格字符
* upper - 大写字符
* xdigit - 十六进制字符

`tr` 命令可以将字符串中的小写字母转换为大写字母。

``` bash
$ echo "Who is the standard text editor?" |tr [:lower:] [:upper:]
WHO IS THE STANDARD TEXT EDITOR?
```

`tr` 可用于从字符串中删除命名字符。

示例 `tr` - 从字符串中删除字符：

``` bash
$ echo 'ed, of course!' |tr -d aeiou # 删除字符 a,e,i,o,u
d, f crs!
```

使用 `tr` 将字符串中的任何命名字符转换为空格。当按顺序遇到多个命名字符时，它们将被转换为单个空格。

`-s` 选项标志的行为在系统之间不同。

示例 `tr` - 将字符转换为空格：

``` bash
$ echo 'The ed utility is the standard text editor.' |tr -s astu ' '
The ed ili y i he nd rd ex edi or.
```

`-s` 选项标志可用于抑制 sting 中的额外空白。

``` bash
$ echo 'extra     spaces – 5’ | tr -s [:blank:]
extra spaces - 5
$ echo ‘extra       tabs – 2’ | tr -s [:blank:]
extra   tabs – 2
```

在基于 UNIX 和 Windows 的系统之间传输文件时的常见问题是行分隔符。在 UNIX 系统上，分隔符是一个新行，而 Windows 系统使用两个字符，回车符后跟换行符。使用 `tr` 进行重定向是解决此格式问题的一种方法。

示例 `tr` - 删除回车：

``` bash
$ tr -d '\r' < dosfile.txt > unixfile.txt
```

## colrm

使用 `colrm` 文本列可以从流中剪切。在第一个示例中， `colrm` 用于从管柱 4 切割到管道每条管线的末端。接下来，将相同的文件发送到 `colrm` 以删除第 4 列到第 5 列。

用于删除列的示例 `colrm` ：

``` bash
$ cat grocery.list |colrm 4
app
ban
plu
car
$ cat grocery.list |colrm 4 5
apps
banas
plu
carts
```

## expand 和 unexpand

`expand` 命令将制表符更改为空格，而 `unexpand` 将空格更改为制表符。这些命令从 stdin 或命令行上指定的文件中获取输入。使用 `-t` 选项，可以设置一个或多个制表位。

``` bash
$ cat grocery.list|head -2|nl|nl
     1       1  apples
     2       2  bananas
$ cat grocery.list|head -2|nl|nl|expand -t 5
     1         1    apples
     2         2    bananas
$ cat grocery.list|head -2|nl|nl|expand -t 5,20
     1                   1 apples
     2                   2 bananas
$ cat grocery.list|head -2|nl|nl|expand -t 5,20|unexpand -t 1,5
                1                   1 apples
                2                   2 bananas
```

## comm, cmp, 和 diff

要演示这些命令，需要创建两个新文件。

创建演示文件：

``` bash
cat << EOF > dummy_file1.dat
011 IBM 174.99
012 INTC 22.69
013 SAP 59.37
014 VMW 102.92
EOF
cat << EOF > dummy_file2.dat
011  IBM 174.99
012 INTC 22.78
013 SAP 59.37
014 vmw 102.92
EOF
```

`diff` 命令比较两个文件，报告它们之间的差异。`diff` 需要许多选项标志。在下面的示例中，首先显示默认差异，然后使用忽略空格的 `-w` 选项显示 `diff` ，然后使用 `-i` 选项标志的示例结束，该标志在进行比较时忽略大小写之间的差异。

``` bash
$ diff dummy_file1.dat dummy_file2.dat
1,2c1,2
< 011 IBM 174.99
< 012 INTC 22.69
---
> 011  IBM 174.99
> 012 INTC 22.78
4c4
< 014 VMW 102.92
---
> 014 vmw 102.92
 
$ diff -w dummy_file1.dat dummy_file2.dat
2c2
< 012 INTC 22.69
---
> 012 INTC 22.78
4c4
< 014 VMW 102.92
---
> 014 vmw 102.92
 
$ diff -i dummy_file1.dat dummy_file2.dat
1,2c1,2
< 011 IBM 174.99
< 012 INTC 22.69
---
> 011  IBM 174.99
> 012 INTC 22.78
```

`comm` 命令比较两个文件，但行为与 `diff` 有很大不同。 `comm` 生成三列输出 - 仅在 file1（第1列）中的行，仅在 file2（第2列）中的行，以及两个文件共有的行（第3列）。选项标志可用于抑制输出列。此命令可能最有用于抑制第 1 列和第 2 列，仅显示两个文件共有的行，如下所示。

``` bash
$ comm dummy_file1.dat dummy_file2.dat
        011  IBM 174.99
011 IBM 174.99
012 INTC 22.69
        012 INTC 22.78
                013 SAP 59.37
014 VMW 102.92
        014 vmw 102.92
 
$ comm -12 dummy_file1.dat dummy_file2.dat
013 SAP 59.37
```

`cmp` 命令也是比较两个文件。但是，与 `comm` 或 `diff` 不同， `cmp` 命令（默认情况下）报告两个文件首先不同的字节和行号。

``` bash
$ cmp dummy_file1.dat dummy_file2.dat
dummy_file1.dat dummy_file2.dat differ: char 5, line 1
```

## fold

使用 `fold` 命令，行以指定的宽度断开。最初，此命令用于帮助格式化无法包装文本的固定宽度输出设备的文本。 `-w` 选项标志允许使用指定的行宽而不是默认的 80 列。

``` bash
$ fold -w8 dummy_file1.dat
011 IBM
174.99
012 INTC
 22.69
013 SAP
59.37
014 VMW
102.92
```

## paste

paste 命令用于并排排列文件，分别合并每个文件的记录。使用重定向，可以通过连接一个文件的每个记录来创建新文件，并将记录放在另一个文件中。

创建演示文件：

``` bash
cat << EOF > dummy1.txt
IBM
INTC
SAP
VMW
EOF
cat << EOF > dummy2.txt
174.99
22.69
59.37
102.92
EOF
```

示例1 - 来自多个文件的行：

``` bash
$ paste dummy1.txt dummy2.txt grocery.list
IBM     174.99  apples
INTC    22.69   bananas
SAP     59.37   plums
VMW     102.92  carrots
```

有一个 `-s` 选项标志用于一次一个（串行）处理文件而不是并行处理。请注意，下面的列与上面示例中的行对齐。

示例2 - 来自多个文件的行：

``` bash
$ paste -s dummy1.txt dummy2.txt grocery.list
IBM     INTC    SAP     VMW
174.99  22.69   59.37   102.92
apples  bananas plums   carrots
```

如果只指定了一个文件，或者粘贴正在处理 stdin，则默认情况下输入列在一列中。使用 `-s` 选项标志，输出列在一行上。由于输出压缩为单行，因此使用分隔符分隔返回的字段（选项卡是默认分隔符）。在此示例中，find命令用于查找可能找到64位库的目录，并构建适合附加到 `$LD_LIBRARY_PATH` 变量的路径。

示例 - 带分隔符：

``` bash
$ find /usr -name lib64 -type d|paste -s -d:
/usr/lib/qt3/lib64:/usr/lib/debug/usr/lib64:/usr/X11R6/lib/X11/locale/lib64:/usr/X11R6/
lib64:/usr/lib64:/usr/local/ibm/gsk7_64/lib64:/usr/local/lib64
 
$ paste -d, dummy1.txt dummy2.txt
IBM,174.99
INTC,22.69
SAP,59.37
VMW,102.92
```

## bc

为了便于在 shell 中进行数学运算，请考虑 `bc` ，“基本计算器” 或 “台式计算器”。有些 shell 提供了本地进行数学运算的方法，有些 shell 依赖于 expr 来计算表达式。使用 `bc` ，计算可以跨 shell 和 UNIX 系统移植，只需要小心供应商扩展。

`bc` 示例 - 简单计算：

``` bash
$ echo 2+3|bc
5
 
$ echo 3*3+2|bc
11
 
$ VAR1=$(echo 2^8|bc)
$ echo $VAR1
256
 
$ echo "(1+1)^8"|bc
256
```

`bc` 可以执行的不仅仅是这些简单的计算。它还是一个解释器，用于定义自己的内部和用户定义的函数，语法和流控制，类似于编程语言。默认情况下，bc 不包括小数点右侧的任何数字。使用特殊比例变量提高输出精度。如示例所示，bc 会扩展到大数，并执行冗长的精度。使用 obase 或 ibase 控制输入和输出数字的转换基数。在下面的示例中：

* obase 更改默认输出基数（十），将结果转换为十六进制
* 对于 2 近似的平方根，scale 指定小数点右边的位数
* 通过计算2到128来说明对大数的支持
* 调用 sqrt() 内部函数来计算 2 的平方根
* 从 ksh，计算并打印一个百分比

bc 示例 - 更多计算：

``` bash
$ echo "obase=16; 2^8-1"|bc
FF
 
$ echo "99/70"|bc
1
 
$ echo "scale=20; 99/70"|bc
1.41428571428571428571
 
$ echo "scale=20;sqrt(2)"|bc
1.41421356237309504880
 
$ echo 2^128|bc
340282366920938463463374607431768211456
 
$ printf "Percentage: %2.2f%%\n" $(echo .9963*100|bc)
Percentage: 99.63%
```
?> bc 的手册页是详细的，包括示例。

## split

split 命令的一个有用任务是将大型数据文件拆分为较小的文件以进行处理。在此示例中，使用 `wc` 命令显示 BigFile.dat 具有 165782 行。`-l` 选项标志告诉 `split` 每个输出文件的最大行数。 `split` 允许为输出文件名指定前缀， `BigFile_` 是指定的前缀。其他选项允许后缀控制，而在 BSD 上，`-p` 选项标志允许在正则表达式（如 csplit（上下文拆分）命令）上进行拆分。

``` bash
$ wc BigFile.dat
165782  973580 42557440 BigFile.dat
 
$ split -l 15000 BigFile.dat BigFile_
 
$ wc BigFile*
  165782  973580 42557440 BigFile.dat
   15000   87835 3816746 BigFile_aa
   15000   88483 3837494 BigFile_ab
   15000   89071 3871589 BigFile_ac
   15000   88563 3877480 BigFile_ad
   15000   88229 3855486 BigFile_ae
    7514   43817 1908914 BigFile_af
  248296 1459578 63725149 total
```

## cut

`cut` 命令用于“修剪”文件的基于列的部分或从 stdin 传送给它的数据。它按列表指定的字节（`-b`），字符（`-c`）或字段（`-f`）剪切数据。使用逗号分隔的列表和连字符指定字段列表或字节/字符位置。如果输出只需要一个，只需指定所需的位置或字段即可。可以使用连字符指定一系列字段，以便1-3打印字段（或位置）1到3，-2从行的开头打印到字段2（或字节/字符2），并指定3-切割到打印字段（或位置3）到行尾。可以使用逗号指定多个字段。一些其他有用的标志是 `-d` 来指定一个字段分隔符，而 `-s` 来压缩没有分隔符的行。

``` bash
$ cat << EOF > dummy_cut.dat
# this is a data file
ID,Name,Score
13BA,John Smith,100
24BC,Mary Jones,95
34BR,Larry Jones,94
36FT,Joe Ruiz,93
40RM,Kay Smith,91
EOF
 
$ cat dummy_cut.dat |cut -d, -f1,3
# this is a data file
ID,Score
13BA,100
24BC,95
34BR,94
36FT,93
40RM,91
 
$ cat dummy_cut.dat |cut -b6-
s is a data file
me,Score
John Smith,100
Mary Jones,95
Larry Jones,94
Joe Ruiz,93
Kay Smith,91
 
$ cat dummy_cut.dat |cut -f1- -d, -s
ID,Name,Score
13BA,John Smith,100
24BC,Mary Jones,95
34BR,Larry Jones,94
36FT,Joe Ruiz,93
40RM,Kay Smith,91
```

## uniq

`uniq` 命令通常用于唯一地列出来自输入源（通常是文件或标准输入）的行。要正确操作，必须在输入中连续定位重复的行。通常，对 `uniq` 命令的输入已排序，因此重复的行对齐。与 `uniq` 命令一起使用的两个常用标志是 `-c`，用于打印每行出现次数的计数，`-d` 可用于显示重复行的一个实例。

``` bash
$ cat << EOF > dummy_uniq.dat
 
13BAR   Smith   John    100
13BAR   Smith   John    100
24BC    Jone    Mary    95
34BRR   Jones   Larry   94
36FT    Ruiz    Joe     93
40REM   Smith   Kay     91
13BAR   Smith   John    100
99BAR   Smith   John    100
13XIV   Smith   Cindy   91
 
 
EOF
 
$ cat dummy_uniq.dat | uniq
 
13BAR   Smith   John    100
24BC    Jone    Mary    95
34BRR   Jones   Larry   94
36FT    Ruiz    Joe     93
40REM   Smith   Kay     91
13BAR   Smith   John    100
99BAR   Smith   John    100
13XIV   Smith   Cindy   91
 
$ cat dummy_uniq.dat | sort |uniq
 
13BAR   Smith   John    100
13XIV   Smith   Cindy   91
24BC    Jone    Mary    95
34BRR   Jones   Larry   94
36FT    Ruiz    Joe     93
40REM   Smith   Kay     91
99BAR   Smith   John    100
 
$ cat dummy_uniq.dat | sort |uniq -d
 
13BAR   Smith   John    100
 
$ cat dummy_uniq.dat | sort |uniq -c
   3
   3 13BAR   Smith   John    100
   1 13XIV   Smith   Cindy   91
   1 24BC    Jone    Mary    95
   1 34BRR   Jones   Larry   94
   1 36FT    Ruiz    Joe     93
   1 40REM   Smith   Kay     91
   1 99BAR   Smith   John    100
```

## sort

要按特定顺序（例如字母或数字）排列 stdin 中的行或文件，可以使用 `sort` 命令。默认情况下， `sort` 的输出写在 stdout 上。环境变量（如 LC_ALL，LC_COLLATE 或 LANG）可以影响排序和其他命令的输出。注意示例文件如何显示 2 个单独的重复记录 - 一个用于IBM的欺骗，另一个欺骗是一个空行。

排序示例 - 默认行为：

``` bash
$ cat << EOF > dummy_sort1.dat
 
014 VMW, 102.92
013 INTC, 22.69
012 sap,  59.37
011 IBM, 174.99
011 IBM, 174.99
 
EOF
 
$ sort dummy_sort1.dat
 
 
011 IBM, 174.99
011 IBM, 174.99
012 sap,  59.37
013 INTC, 22.69
014 VMW, 102.92
```

`sort` 有一个很好的标志，在许多情况下可以代替 `uniq` 命令。`-u` 选项标志对文件进行排序，删除重复的行，以便生成唯一的输出行列表。

``` bash
$ sort -u dummy_sort1.dat
 
011 IBM, 174.99
012 sap,  59.37
013 INTC, 22.69
014 VMW, 102.92
```

有时，需要输入的反向排序。默认情况下，排序从最小到最大（对于数字）排列顺序，并按字母顺序排列字符数据。使用 `-r` 选项标志可以反转默认排序顺序。

``` bash
$ sort -ru dummy_sort1.dat
014 VMW, 102.92
013 INTC, 22.69
012 sap,  59.37
011 IBM, 174.99
```

不同的情况要求在某些字段或 “键” 上对文件进行排序。幸运的是，`sort` 具有 `-k` 选项标志，允许按位置指定排序键。默认情况下，字段由空格分隔。

``` bash
$ sort -k2 -u dummy_sort1.dat
 
011 IBM, 174.99
013 INTC, 22.69
014 VMW, 102.92
012 sap,  59.37
```

区分大小写是一个问题， `sort` 提供 `-f` 选项标志，在进行比较时忽略大小写。当组合如下所示的多个标志时，某些版本的UNIX需要以不同的顺序指定这些标志。

排序示例 - 排序忽略大小写：

``` bash
$ sort -k2 -f -u dummy_sort1.dat
 
011 IBM, 174.99
013 INTC, 22.69
012 sap,  59.37
014 VMW, 102.92
```

到目前为止，所有种类都是字母变种。当数据需要按数字顺序排序时，请使用 `-n` 选项标志。

``` bash
$ sort -n -k3 -u dummy_sort1.dat
 
013 INTC, 22.69
012 sap,  59.37
014 VMW, 102.92
011 IBM, 174.99
```

某些输入可能使用除空格之外的字符来分隔行中的字段。使用 `-t` 选项标志指定非默认分隔符，例如逗号。

排序示例 - 使用非默认分隔符对字段进行排序：

``` bash
$ sort -k2 -t"," -un dummy_sort1.dat
 
013 INTC, 22.69
012 sap,  59.37
014 VMW, 102.92
011 IBM, 174.99
```

## join

熟悉编写数据库查询的任何人都会识别 `join` 命令的实用程序。与大多数 UNIX 命令一样，输出显示在 stdout 上。为了“连接”文件，两个文件中的指定字段逐行进行比较。如果未指定任何字段，则在每行开头的字段上加入匹配项。默认字段分隔符是空格（某些系统只是空格或相邻空格）。当发生字段匹配时，为具有匹配字段的每对行写入一行输出。对于合法结果，应该在要匹配的字段上对两个文件进行排序。并非所有系统都以相同的方式实现连接。

此示例使用 `-t` 指定字段分隔符，并演示在用逗号分隔的第一个字段（默认）上连接两个文件。数据库操作员会将其识别为内连接，仅显示匹配的行。

示例 - 使用非默认字段分隔符：

``` bash
cat << EOF > dummy_join1.dat
011,IBM,Palmisano
012,INTC,Otellini
013,SAP,Snabe
014,VMW,Maritz
015,ORCL,Ellison
017,RHT,Whitehurst
EOF
 
cat << EOF > dummy_join2.dat
011,174.99,14.6
012,22.69,10.4
013,59.37,26.4
014,102.92,106.1
016,27.77,31.2
EOF
 
cat << EOF > dummy_join3.dat
IBM,Armonk
INTC,Santa Clara
SAP,Walldorf
VMW,Palo Alto
ORCL,Redwood City
EMC,Hopkinton
EOF
 
$ join -t, dummy_join1.dat dummy_join2.dat
011,IBM,Palmisano,174.99,14.6
012,INTC,Otellini,22.69,10.4
013,SAP,Snabe,59.37,26.4
014,VMW,Maritz,102.92,106.1
```

要指定在每个文件中 “加入” 的字段，可以使用 `-j[1,2] x` 选项标志（或简称为 `-1 x` or `-2 x`）。选项标志 `-j1 2` 或 `-1 2` 指定文件 1 的第二个字段，即命令中列出的第一个文件。此示例显示如何基于第一个文件中的字段 1 和第二个文件中的字段 2 加入文件，也是仅匹配行的内部联接。

示例 - 指定的字段：

``` bash
$ join -t, -j1 1 -j2 2 dummy_join3.dat dummy_join1.dat
IBM,Armonk,011,Palmisano
INTC,Santa Clara,012,Otellini
SAP,Walldorf,013,Snabe
VMW,Palo Alto,014,Maritz
ORCL,Redwood City,015,Ellison
```

保持数据库相关示例的精神，标志可用于产生左外连接。左外连接包括左第一个文件或表中的所有行以及来自第二个文件或表的匹配行。使用 `-a` 包含指定文件中的所有行。

示例 - 左外连接：

``` bash
$ join -t, -a1 dummy_join1.dat dummy_join2.dat
011,IBM,Palmisano,174.99,14.6
012,INTC,Otellini,22.69,10.4
013,SAP,Snabe,59.37,26.4
014,VMW,Maritz,102.92,106.1
015,ORCL,Ellison
017,RHT,Whitehurst
```

完全外部联接包括来自文件或表的所有行，如果字段匹配则无关紧要。可以通过使用 `-a` 选项标志指定两个文件来完成外部连接。

示例 - 完全外连接：

``` bash
$ join -t, -a1 -a2 -j1 2 -j2 1 dummy_join1.dat dummy_join3.dat
IBM,011,Palmisano,Armonk
INTC,012,Otellini,Santa Clara
SAP,013,Snabe,Walldorf
VMW,014,Maritz,Palo Alto
ORCL,015,Ellison,Redwood City
EMC,Hopkinton
017,RHT,Whitehurst
```