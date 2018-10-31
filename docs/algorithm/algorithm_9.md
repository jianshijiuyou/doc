
### 2-3 查找树

为了保证查找树的平衡性，需要一些灵活性，因此在这里允许树中的一个结点保存多个键。将标准的二叉查找树中的结点称为 2- 结点（含有一个键和两条链接），现在引入 3- 结点，它含有两个键和三条链接。

2- 结点 和 3- 结点中的每条链接都对应着其中保存的键所分割产生的一个区间。

#### 定义 
一棵 2-3 查找树或为一棵空树，或由以下结点组成：
 - 2- 结点，含有一个键和两条链接，左链接指向的 2-3 树中的键都小于该结点，右链接指向的 2-3 树中的键都大于该结点。
 - 3- 结点，含有两个键和三条链接，左链接指向的 2-3 树中的键都小于该结点，中链接指向的 2-3 树中的键都位于该结点的两个键之间，右链接指向的 2-3 树中的键都大于该结点。


![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-anatomy.png)

#### 查找

2-3 树的查找思路和 二叉树其实差不多，只是需要在 3- 结点多做一次判断。


> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-search.png)
> 2-3 树中的查找命中（左）和未命中（右）

#### 向 2- 结点中插入新键
直接将 2- 结点 转换成 3- 结点，简单粗暴。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-insert2.png)

#### 向一棵只含有一个 3- 结点的树中插入新键

先将 3- 结点转换成一个临时的 4- 结点。

再将 4- 结点转换为一棵由 3 个 2- 结点组成的 2-3 树。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-insert3a.png)

#### 向一个父结点为 2- 结点的 3- 结点中插入新键

先将 3- 结点转换成一个临时的 4- 结点。

再将 4- 结点的中键移动到父结点中。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-insert3b.png)


#### 向一个父结点为 3- 结点的 3- 结点中插入新键

先将 3- 结点转换成一个临时的 4- 结点。

再将 4- 结点的中键移动到父结点中。

将父结点构造成一个临时的 4- 结点，然后进行相同的变换。

直到遇到一个 2- 结点并将它替换为一个不需要继续分解的 3- 结点，或者是到达 3- 结点的根。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-insert3c.png)


#### 分解根结点

如果插入结点到根结点的路径上全都是 3- 结点，根结点最终会变成一个临时的 4- 结点。

此时将 4- 结点转换为一棵由 3 个 2- 结点组成的 2-3 树即可，树的高度加 1。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/23tree-split.png)


#### 局部变换

将一个 4- 结点分解为一棵 2-3 树可能有 6 种情况。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/fd2f720a-bc1c-48af-9c91-0db0bed5092d.png)

> 每次的变换都是局部的，变更的链接数量不会超过一个很小的常数。


#### 性质

**在一棵大小为 N 的 2-3 树中，查找和插入操作访问的结点必然不超过 lgN 个**


2-3 树的性能虽然不错，但是实现起来比较麻烦，需要处理的情况实在太多，还要维护两种不同类型的结点。


### 红黑二叉查找树

红黑树的基本思想是用标准的二叉查找树和一些额外的信息来表示 2-3 树，将树中的链接分为两种类型：红链接将两个 2- 结点连接起来构成一个 3- 结点， 黑链接则是 2-3 树中的普通链接。准确的说，就是将 3- 结点表示为由一条左斜的红色链接相连的两个 2- 结点。


![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-encoding.png)

#### 定义
红黑树的另一种定义是含有红黑链接并满足下列条件的二叉查找树：
 - 红链接均为左链接
 - 没有任何一个结点同时和两条红链接相连
 - 该树是完美黑色平衡的，即任意空链接到根结点的路径上的黑链接数量相同




> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-1-1.png)
> 红黑树即是二叉查找树，也是 2-3 树。

#### 颜色表示

每个结点只会有一条指向自己的链接，结点对应的数据类型会保存指向自己的链接的颜色（不会保存自己指向子结点链接的颜色）。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-color.png)


#### 旋转

在实现某些操作时可能会出现红色右链接或者两条连续的红链接，这时候需要通过 “旋转” 来修复。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-left-rotate.png) ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-right-rotate.png)


#### 向 2- 结点中插入新键

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/e672c95d-59bd-49bd-880e-9bdb0ccbd1b5.png)


#### 向树底部的 2- 结点插入新键


![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/efa957b9-3dc0-4413-9450-53fba780037d.png)


#### 向一棵双键（即一个 3- 结点）中插入新键

这种情况又可分为三种子情况：


![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/73a08cc0-773d-4c3a-869c-da3cce562987.png)


#### 颜色转换

需要专门用一个方法来转换一个结点的两个子结点的颜色，除了将子结点的颜色由红变黑之外，同时还要将父结点的颜色由黑变红。

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/color-flip.png)


> 根结点总是黑色的，当根结点由红变黑时树的黑链接高度就会加 1。


#### 向树底部的 3- 结点插入新键


![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/90539a41-712e-4cef-a391-b4c265b72b4e.png)


#### 将红链接在树中向上传递

沿着插入点到根结点的路径向上移动时在所经过的每个结点中顺序完成以下操作，就能完成插入操作：
 - 如果右子结点是红色的而左子结点是黑色的，进行左旋转
 - 如果左子结点是红色的且它的左子结点也是红色的，进行右旋转
 - 如果左右子结点均为红色，进行颜色转换

#### 算法实现
[RedBlackBST.java](http://algs4.cs.princeton.edu/33balanced/RedBlackBST.java.html)

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/redblack-construction.png)
> 红黑树的构造轨迹

#### 性质

红黑树能够同时实现高效的查找，插入和删除操作。

一棵大小为 N 的红黑树的高度不会超过 2lgN。

一棵大小为 N 的红黑树中，根结点到任意结点的平均路径长度为 ～ 1.00lgN。


> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm9/7dc32760-e6a8-4ea9-a578-3ed03afe8fa9.png)
> 各种符号表实现的性能总结

### 说明
本文内容是对『算法』第四版 的摘要总结！  
文章所用图片部分来自 [『算法』第四版 英文原版](http://algs4.cs.princeton.edu/home/) ，部分来自 **『算法』第四版 中文译版**