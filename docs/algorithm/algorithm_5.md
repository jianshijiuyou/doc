﻿
### 介绍
许多应用程序都需要处理有序的元素，但不一定要求它们全部有序，或是不一定要一次就将它们排序，很多情况下是先收集一些元素，处理当前键值最大的元素，然后再收集更多的元素，再处理当前键值最大的元素，如此这般。就像操作系统中的进程一样，每次执行优先级最高的进程，执行完后再从进程队列中取出最高优先级的进程继续执行，同时还有新的进程进入队列等待执行。

像这样**支持 删除最大元素 和 插入元素 两种操作的数据结构叫做优先队列**。

本篇文章将讲述基于二叉堆数据结构的一种优先队列的经典实现方法，用数组保存元素并按照一定条件排序，以实现高效地（对数级别）删除最大元素和插入元素操作。

**通过插入一列元素然后一个个地删除其中最小的元素，就能实现排序，堆排序的重要排序算法就来自于基于堆的优先队列的实现**。

### 优先队列的基本 API
![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/pq-api.png)

### 实现方式
优先队列可以使用有序或无序的数组或链表来实现。在队列较小时，大量使用两种主要操作之一时，或是所操作元素的顺序已知时，它们十分有用。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/priority_queues_1.png)

在一个优先队列上执行一系列操作

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/pq-array.png)

### 堆的定义

数据结构二叉堆能够很好地实现优先队列的基本操作。在二叉堆的数组中，每个元素都要保证大于等于另两个特定位置的元素，相应地，这些位置的元素又至少要大于等于数组中的另两个元素，以此类推。如果将所有元素画成一棵二叉树，将每个较大的元素和两个较小的元素用边线连接就可以很容易看出这种结构。

**当一棵二叉树的每个节点都大于等于它的两个子结点时，它被称为堆有序。**

**从任意结点向上，都能得到一列非递减的元素，从任意节点向下，都能得到一列非递增的元素**

**根节点是堆有序的二叉树中的最大结点**

### 二叉堆表示法

如果用指针来表示堆有序的二叉树，每个元素都需要三个指针来找到它的上下结点。然是如果用完全二叉树，只用数组就可以表示。

根节点在位置 1，它的子结点在位置 2 和 3，而子结点的子结点分别在位置 4，5，6 和 7，以此类推。

**二叉堆是一组能够用堆有序的完全二叉树排序的元素，并在数组中按照层级存储（不使用数组的第一个位置）**

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/heap.png)

（下文将二叉堆简称堆）在一个堆中，位置 k 的结点的父结点的位置为 k/2 （向下取整），它的两个子结点的位置分别为 2k 和 2k+1。这样不使用指针也可以通过计算数组的索引在树中上下移动（可以这样计算的前提是不使用数组的第一个位置）。

这样就能实现对数级别的插入元素和删除最大元素的操作。

因为 **一棵大小为 N 的完全二叉树的高度为 lgN（向下取整）。**

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/heap-representations.png)

### 堆的算法

用长度为 N+1 的私有数组 pq[] 表示一个大小为 N 的堆，不使用 pq[0] ，堆元素放在 pq[1] 至 pq[N] 中。通过辅助函数 less() 和 exch() 访问元素。

``` java
private boolean less(int i, int j) {
    return pq[i].compareTo(pq[j]) < 0;
}

private void exch(int i, int j) {
    Key swap = pq[i];
    pq[i] = pq[j];
    pq[j] = swap;
}
```

**打破堆的状态，然后再遍历堆并按照要求将堆的状态恢复，这个过程叫做堆的有序化 (reheapifying)。**

有序化的过程中会遇到两种情况。当某个结点的优先级上升（或是在堆底加入一个新的元素）时，需要由下至上恢复堆的顺序。当某个结点的优先级下降（例如，将跟结点替换为一个较小的元素）时，需要由上至下恢复堆的顺序。首先需要学习如何实现这两种辅助操作，然后再用它们实现插入元素和删除最大元素的操作。

#### 由下至上的堆有序化（上浮）

当堆的有序状态因为某个结点变得比它的父结点更大而被打破，就需要通过交换它和它的父结点来修复堆。交换后，这个结点比它的两个子结点都大，但这个结点仍然可能比它现在的父结点更大。需要用同样的方法恢复秩序，只要记住位置 k 的结点的父结点的位置是 k/2 （向下取整），具体代码如下：

``` java
private void swim(int k) {
    while (k > 1 && less(k/2, k)) {
        exch(k, k/2);
        k = k/2;
    }
}
```

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/swim.png)

#### 由上至下的堆有序化（下沉）

当堆的有序状态因为某个结点变得比它的两个子结点或是其中之一更小了而被打破了，需要通过将它和它的两个子结点中的较大者交换来恢复堆，同样，这个过程可能需要重复，直到整个堆恢复有序状态。由位置为 k 的结点的子结点位于 2k 和 2k+1 可以得到对应的代码：

``` java
private void sink(int k) {
   while (2*k <= N) {
      int j = 2*k;
      if (j < N && less(j, j+1)) j++;
      if (!less(k, j)) break;
      exch(k, j);
      k = j;
   }
}
```

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/sink.png)


#### 插入元素

将新元素加到数组末尾，增加堆的大小并让这个新元素上浮到合适的位置

#### 删除最大元素

从数组顶端删去最大的元素并将数组的最后一个元素放到顶端，减小堆的大小并让这个元素下沉到合适的位置

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/heap-ops.png)
> 插入元素 和 删除最大元素 示意图

#### 算法实现

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/priority_queues_2.png)

<font color="red">**对于一个含有 N 个元素的基于堆的优先队列，插入元素操作只需要不超过（lgN+1）次比较，删除最大元素的操作需要不超过 2lgN 次比较。**</font>

### 堆排序

在基于堆的优先队列中，重复调用删除最小（大）元素的操作将它们按顺序删去，这种排序算法就是堆排序

堆排序可以分为两个阶段。在堆的构造阶段中，将原始数组重新组织安排进一个堆中。然后在下沉排序阶段，从堆中按递减顺序取出所有元素并得到排序结果。

#### 堆的构造

从右至左用 sink() 函数构造子堆。把数组的每个位置都看成是一个子堆的根结点， sink() 对于这些子堆也适用。如果一个结点的两个子结点都已经是堆了，那么在该结点上调用 sink() 可以将它们变成一个堆。这个过程会递归的建立起堆的秩序。

只需扫描数组中的一半元素，因为可以跳过大小为 1 的子堆（如果从左至右用 swim() 函数构造则需要扫描整个数组）。最后在位置 1 （根结点）上调用 sink() 方法，扫描结束，此时，有序堆已经构成。

<font color="red">**用下沉操作由 N 个元素构造堆只需少于 2N 次比较以及少于 N 次交换**</font>

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/heapsort.png)
>  堆排序的轨迹

#### 下沉排序

堆有序后，将堆中最大元素删除，放入堆缩小后数组中空出的位置。这个过程和选择排序有些类似，但所需的比较要少得多，因为堆提供了一种从未排序部分找出最大元素的有效方法。

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm/priority_queues/heapsort-trace.png)
>  堆排序：堆的构造（左） 和下沉排序（右）

### 最后

堆排序是唯一能够同时最优地利用空间和时间的方法，在最坏的情况下它也能保证使用 ～ 2NlgN 次比较和恒定的额外空间。当空间十分紧张的时候它很流行，因为它只用几行就能实现（甚至机器码也是）较好的性能。

但现代系统的许多应用很少使用它，因为它无法利用缓存。数组元素很少和相邻的其他元素进行比较，因此缓存未命中的次数要远远高于大多数比较都在相邻元素间进行的算法，如快速排序，归并排序，甚至是希尔排序。

但堆实现的优先队列在现代应用程序中越来越重要，因为它能在插入操作和删除最大元素操作混合的动态场景中保证对数级别的运行时间。

### 说明
本文内容是对『算法』第四版 的摘要总结！  
文章所用图片部分来自 [『算法』第四版 英文原版](http://algs4.cs.princeton.edu/home/) ，部分来自 **『算法』第四版 中文译版**