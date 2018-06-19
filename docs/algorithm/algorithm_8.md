
### 二叉查找树

二叉查找树可以将链表插入的灵活性和有序数组查找的高效性结合起来。

就是使用每个结点含有两个链接（链表中每个结点只含有一个链接）的二叉查找树来高效地实现符号表。


首先看看二叉树的一些性质：
 - 结点包含的链接可以指向 `null` 或者其他结点。
 - 每个结点有且只能有一个父结点指向自己（根节点除外，没有父结点）。
 - 每个结点都只有两个链接，分别指向自己的左子结点和右子结点。
 - 每个结点包含一个键和一个值。

> 尽管链接指向的是结点，但可以将每个链接看做指向了另一棵二叉树，而这棵数的根结点就是被指向的结点。因此可以将二叉树定义为一个空链接，或者是一个有左右两个链接的结点，每个链接都指向一棵子二叉树。

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/binary-tree-anatomy.png)


**定义：一棵二叉查找树 (BST) 是一棵二叉树，其中每个结点都含有一个键（以及相关联的值）且每个结点的键都大于其左子树中的任意结点的键而小于右子树的任意结点的键。**

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-anatomy.png)

### 基本实现

每个结点都含有一个键、一个值、一个左链接，一个右链接和一个结点计数器。左链接指向一棵由小于该结点的所有键组成的二叉查找树，右链接指向一棵由大于该结点的所有键组成的二叉查找数。每棵树的大小（结点总数）可以看成是其左子树的大小加上右子树的大小加 1 （根结点自己）。

size(x) = size(x.left) + size(x.right) + 1

一棵二叉查找树代表了一组键值对的集合，而同一个集合可以用多棵不同的二叉查找树表示：

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-subtree-count2.png)


#### 查找 

在二叉查找树中查找一个键的递归算法：如果数是空的，则查找未命中；如果被查找的键和根节点的键相等，查找命中，否则就（递归地）在适当的子树中继续查找。如果被查找的键较小就选择左子树，较大则选择右子树。

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-search.png)

``` java
public class BST<Key extends Comparable<Key>, Value> {
	private Node root;             // 二叉查找树的根结点

    private class Node {
        private Key key;           // 键
        private Value val;         // 值
        private Node left, right;  // 指向子树的链接
        private int size;          // 以该结点为根的子树中的结点总数

        public Node(Key key, Value val, int size) {
            this.key = key;
            this.val = val;
            this.size = size;
        }
    }
    
    public int size() {
        return size(root);
    }

    private int size(Node x) {
        if (x == null) return 0;
        else return x.size;
    }

    public Value get(Key key) {
        return get(root, key);
    }

    private Value get(Node x, Key key) {
        if (x == null) return null;
        int cmp = key.compareTo(x.key);
        if      (cmp < 0) return get(x.left, key);
        else if (cmp > 0) return get(x.right, key);
        else              return x.val;
    }
	
    public void put(Key key, Value val) {
        if (val == null) {
            delete(key);
            return;
        }
        root = put(root, key, val);
    }

    private Node put(Node x, Key key, Value val) {
        if (x == null) return new Node(key, val, 1);
        int cmp = key.compareTo(x.key);
        if      (cmp < 0) x.left  = put(x.left,  key, val);
        else if (cmp > 0) x.right = put(x.right, key, val);
        else              x.val   = val;
        x.size = 1 + size(x.left) + size(x.right);
        return x;
    }
}
```

> 完整代码见 [BST.java](http://algs4.cs.princeton.edu/32bst/BST.java.html)

#### 插入

二叉查找树的插入的实现和查找差不多。如果数是空的，就返回一个含有该键值对的新结点；如果被查找的键小于根结点的键，就继续在左子树中插入该键，否则在右子树中插入该键。

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-insert.png)

新结点会连接到树底层的空链接上，树的其他部分则不会改变。只有查找或者插入路径上的结点才会被访问，所以随着树的增长，被访问的结点数量占树的总结点数的比例也会不断的降低。

> ![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/sdfsd7878gsdf8962346dfsg.png)
> 使用二叉查找树的标准索引用例的轨迹

### 分析
使用二叉查找树的算法的运行时间取决于树的形状，而树的形状又取决于键被插入的先后顺序。

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-best.png) ![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-typical.png) ![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-worst.png)


> 二叉查找树和快速排序几乎就是“双胞胎”。树的根结点就是快速排序中的第一个切分元素（左侧的键都比它小，右侧的键都比它大），而这对于所有的子树同样适应，这和快速排序中对子数组的递归排序完全对应。

**性质：在由 N 个随机键构造的二叉查找树中，查找命中平均所需的比较次数为 ～2lnN(1.39lgN)。查找未命中和插入相同。** 

### 删除结点

如果要删除的结点只有一个子结点，那么很好操作，下面只说有两个子结点的结点如何删除。

假设要删除的结点为 x， 在删除它后用它的后继结点（右子树中最小结点）填补它的位置。

具体步骤： 
 - 将指向即将被删除的结点的链接保存为 t。
 - 将 x 指向它的后继结点 min(t.right)。
 - 将 x 的右链接（原本指向一棵所有结点都大于 x.key 的二叉查找树）指向 deleteMin(t.right)。也就是在删除后所有结点仍然都大于 x.key 的子二叉查找树。
 - 将 x 的左链接（本为空）设为 t.left （其下所有的键都小于被删除的结点和它的后继结点）。

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-deletemin.png) ![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/bst-delete.png)


### 最后

**性质：在一棵二叉查找树中，所有操作在最坏情况下所需的时间都和树的高度成正比。**

> ![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm8/hsfgdsfgg453gd34fs68fd.png)
>  简单的符号表实现的成本总结







### 说明
本文内容是对『算法』第四版 的摘要总结！  
文章所用图片部分来自 [『算法』第四版 英文原版](http://algs4.cs.princeton.edu/home/) ，部分来自 **『算法』第四版 中文译版**