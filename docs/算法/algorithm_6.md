
### 各种排序算法的性能特点

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm6/sort-characteristics.png)

### 稳定性

<font color="red">如果一个排序算法能够保留数组中重复元素的相对位置则可以被称为是稳定的。</font>  

什么意思？看张图就明白了  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm6/stability.png)

就是说，如果一组数据先按照时间排序了，然后再按照地理位置排序，排序后相同地理位置的时间依然是有序的，说明该排序算法是稳定的，反之则是不稳定的。
> 相同地理位置可以理解为<font color="red">重复元素</font>，相同地理位置的时间依然是有序可以理解为<font color="red">保留数组中重复元素的相对位置</font>。

### 一个重要的性质

快速排序是最快的通用排序算法。

### 延伸阅读

Java 的系统程序员选择对原始数据类型使用（三向切分的）快速排序，对引用类型使用归并排序。这些选择实际上也暗示着用速度和空间（对于原始数据类型）来换取稳定性（对于引用类型）。

### 说明
本文所有图片均来自 『算法』第四版 中文译版