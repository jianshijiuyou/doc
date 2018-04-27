
## 选择排序
一种最简单的排序算法：首先，找到数组中最小的那个元素，其次，将他和数组中的第一个元素交换位置。再次，在剩下的元素中找到最小的元素，将它和数组中的第二个元素换位置。如此往复，知道整个数组有序，这种方法叫做选择排序，因为它在不断地选择剩余元素中之中的最小者。  
>**每次交换都能排定一个元素，因此交换的总次数是 N。所以算法的时间效率取决于比较的次数。**  

### 特点
 - 运行时间和输入无关
 - 数据移动是最少的
	每次交换都会改变两个数组元素的值，因此选择排序用了 N 次交换--------**交换次数和数组的大小是线性关系（大部分算法都是线性对数或是平方级别）**  

### 具体代码
``` kotlin
fun main(args: Array<String>) {
    val a = arrayOf(1, 32, 2, 23, 5, 234, 12, 978, 2, 2, 0, 4)

    for (i in 0 until a.size) {
        var min=i
        for (j in i until a.size){
            if(a[j]<a[min]){
                min=j
            }
        }
        val t=a[i]
        a[i]=a[min]
        a[min]=t
    }

    a.forEach { print("$it ") }
}
```

选择排序的轨迹（每次交换后的数组内容）  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/selection.png)  
>图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)

## 插入排序
通常人们整理扑克牌的方法是一张一张的来，将每一张牌插入到其他已经有序的牌中的适合位置。在计算机的实现中，为了给要插入的元素腾出空间，我们需要将其余所有元素在插入之前都向右移动一位，这种算法叫做插入排序。
### 特点
 - 和选择排序一样，当前**索引左边的所有元素都是有序**的，但它们是我最终位置还不确定，为了给更小的元素腾出空间，它们可能会移动。当**索引到达数组的右端时，数组就排序完成了**。
 - **运行时间和输入有关**，对一个很大且其中元素已经基本或接近有序的数组进行排序会比对随机顺序的数组或是逆序数组进行排序要快得多。  

### 具体代码
``` kotlin
fun main(args: Array<String>) {
    val a = arrayOf(1, 32, 2, 23, 5, 234, 12, 978, 2, 2, 0, 4)

    for (i in 1 until a.size) {
        for (j in i downTo 1){
            if(a[j]<a[j-1]){
                val t=a[j]
                a[j]=a[j-1]
                a[j-1]=t
            }
        }
    }

    a.forEach { print("$it ") }
}
```
>对于 0 到 a.size-1 之间的每一个 i，将 a[i] 与 a[0] 到 a[i-1] 中比它小的所有元素一次有序地交换。在索引 i 由左向右变化的过程中，它左侧的元素总是有序的，所以当 i 到达数组的右端时排序就完成了。

插入排序的轨迹（每次交换后的数组内容）  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/insertion.png)  
>图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)

插入排序非常适合部分有序的数组，那“部分有序”是如何定义的？  
部分有序数组的几种典型表现：
 - 数组中每个元素距离它的最终位置都不远。  
 - 一个有序的大数组接一个小数组。
 - 数组中只有几个元素的位置不正确。

插入排序和选择排序的轨迹比对图
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/bars.png)  
>图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)




## 希尔排序
希尔排序是一种基于插入排序的快速的排序算法，它为了加快速度简单地改进了插入排序，交换不相邻的元素以对数组的局部进行排序，并最终用插入排序将局部有序的数组排序。  

### 特点
使数组中任意间隔为 h 的元素都是有序的。这样的数组被称为 h 有序数组。换句话说，一个 h 有序的数组就是 h 个互相独立的有序数组编织在一起组成的一个数组。

在进行排序时，如果 h 很大，我们就能将元素移动到很远的地方，为实现更小的 h 有序创造方便。用这种方式，对于任意以 1 结尾的 h 序列，我们都能将数组排序。  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/h-sorted.png)  
>一个 h 有序数组（即一个由 h 个有序子数组组成的数组）  
>*图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)*

希尔排序的方式是：在 h 个子数组中将每个元素交换到比它大的元素之前。只需要在插入排序的代码中将移动元素的距离由 1 改为 h 即可。

这样，希尔排序的实现就转化为了一个类似插入排序但使用不同增量的过程。
>希尔排序高效的原因是它先将数组排成部分有序，而部分有序刚好很适合插入排序

### 具体代码
``` kotlin
fun main(args: Array<String>) {
    val a = arrayOf(1, 32, 2, 23, 5, 234, 12, 978, 2, 2, 0, 4)

    var h=1
    while (h<a.size/3){
        h=3*h+1
    }

    while (h>=1){
        for (i in h until a.size){
            for(j in i downTo h step h){
                if(a[j]>a[j-h]){
                    val t=a[j]
                    a[j]=a[j-h]
                    a[j-h]=t
                }
            }
        }
        h /= 3
    }
    a.forEach { print("$it ") }
}
```
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/shell.png)  
>希尔排序的详细轨迹  
>*图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)*  

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_2/shell-bars.png)  
>希尔排序的可视轨迹
>*图片来自 [algs4.cs.princeton.edu](http://algs4.cs.princeton.edu)*  

## 说明
本文内容是对『算法』第四版 的摘要总结！
