
### 什么是符号表

可以把符号表看成一张抽象的表格，我们可以将信息（值）存储在其中，然后按照指定的键来搜索并获取这些信息。键和值的具体意义取决不同的应用。符号表中可能会保存很多键和很多信息，因此实现一张高效的符号表也是一项很有挑战性的任何。

符号表有时被称为字典，小时候应该都用过新华字典吧，其中，每个汉字就可以看成是键，那值就是对应汉字的读音和解释。

常见的、经典的、高效的符号表实现有三种：二叉查找树、红黑树、散列表。

<font color="red">定义：符号表是一种存储键值对的数据结构，支持两种操作：插入 （put），即将一组新的键值对存入表中；查找 （get），即根据给定的键得到相应的值。</font>

### 典型的符号表应用
![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/symbol-table-applications.png)

### 一般符号表的 API

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/symbol-table-api2.png)

### 有序符号表的 API
![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/ordered-symbol-table-api.png)


### 无序链表中的顺序查找

符号表中使用数据结构的一个简单选择是链表，每个结点存储一个键值对， get() 的实现为遍历链表，用 equals() 方法比较 **被查找的键** 和 **每个结点中的键**。如果匹配成功就用第二个参数指定的值更新和该键相关联的值，否则就用给定的键值对创建一个新的结点并将其插入到链表的开头。这种方法也被称为顺序查找：<font color="red">在查找中我们一个一个地顺序遍历符号表中的所有键并使用 equals() 方法来寻找与被查找的键匹配的键</font>。

算法 SequenttalSearchST 用链表实现了符号表的基本 API。

``` java
public class SequentialSearchST<Key, Value> {
    
    private Node first;      // 链表首结点

   
    private class Node {
        private Key key;
        private Value val;
        private Node next;

        public Node(Key key, Value val, Node next)  {
            this.key  = key;
            this.val  = val;
            this.next = next;
        }
    }
	//查找给定的键，返回相关联的值
    public Value get(Key key) {
        for (Node x = first; x != null; x = x.next) {
            if (key.equals(x.key))
                return x.val;
        }
        return null;
    }
	//查找给定的键，找到则更新其值，否则在表中新建结点
    public void put(Key key, Value val) {
        for (Node x = first; x != null; x = x.next) {
            if (key.equals(x.key)) {
                x.val = val;
                return;
            }
        }
        first = new Node(key, val, first);
        n++;
    }
}
```


> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/sequential-search.png)
> 使用基于链表的符号表的索引用例的轨迹

<font color="red">在含有 N 对键值的基于（无序）链表的符号表中，未命中的查找和插入操作都需要 N 次比较。命中的查找在最坏情况下需要 N 次比较。向一个空表中插入 N 个不同的键需要 ～(N^2)/2 次比较</font>。


### 有序数组中的二分查找


二分查找 可以实现有序符号表的完整 API。它使用的数据结构是一对平行的数组，一个存储键一个存储值。算法 BinarySearchST 可以保证数组中的键有序，然后使用数组的索引来高效地实现 get() 和其他操作。

核心是 rank() 方法，它返回表中小于给定键的键的数量。对于 get() 方法，只要给定的键存在于表中， rank() 方法就能精确地告诉我们在哪里能够找到它（找不到说明不在表中）。

对于 put() 方法，只要给定的键存在于表中，rank() 方法就能够精确地告诉我们到哪里去更新它的值，以及当键不在表中时将键存储到表中的何处。将所有更大的键向后移动一格来腾出位置（从后向前移动）并将给定的键值对分别插入到各自数组中的合适位置。

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/binary-search.png)
> 使用基于有序数组的符号表实现的索引用例的轨迹

``` java
public class BinarySearchST<Key extends Comparable<Key>, Value> {
    private Key[] keys;
    private Value[] vals;
    private int n = 0;

    public BinarySearchST(int capacity) { 
        keys = (Key[]) new Comparable[capacity]; 
        vals = (Value[]) new Object[capacity]; 
    }   

    public int size() {
        return n;
    }

    public Value get(Key key) {
     
        if (isEmpty()) return null;
        int i = rank(key); 
        if (i < n && keys[i].compareTo(key) == 0) return vals[i];
        return null;
    } 

    public int rank(Key key) {
		//.....
    } 

    public void put(Key key, Value val)  {

        int i = rank(key);

        if (i < n && keys[i].compareTo(key) == 0) {
            vals[i] = val;
            return;
        }

        for (int j = n; j > i; j--)  {
            keys[j] = keys[j-1];
            vals[j] = vals[j-1];
        }
        keys[i] = key;
        vals[i] = val;
        n++;
    } 

    public void delete(Key key) {
		//.....
    } 
}
```
> 完整代码见 [BinarySearchST.java](http://algs4.cs.princeton.edu/31elementary/BinarySearchST.java.html)

#### 二分查找

使用有序索引数组来标识被查找的键可能存在的子数组的大小范围。在查找时，先将被查找的键和子数组的中间键比较。如果被查找的键小于中间键，就在左子数组中继续查找，如果大于就在右子数组中继续查找，否则中间键就是要找的键。

递归的二分查找

``` java
public int rank(Key key,int lo,int hi){
	if(hi<lo)
		return lo;
	int mid = lo + (hi-lo)/2;
	int cmp = key.compareTo(keys[mid]);
	if(cmp<0){
		return rank(key,lo,mid-1);
	}else if(cmp>0){
		return rank(key,mid+1,hi);
	}else{
		return mid;
	}
}
```

迭代的二分查找

``` java
public int rank(Key key) {

    int lo = 0, hi = n-1; 
    while (lo <= hi) { 
        int mid = lo + (hi - lo) / 2; 
        int cmp = key.compareTo(keys[mid]);
        if      (cmp < 0) hi = mid - 1; 
        else if (cmp > 0) lo = mid + 1; 
        else return mid; 
    } 
    return lo;
} 
```

> lo 的初始值为 0，且永远不会变小

首先将 key 和中间键比较，如果相等则返回其索引；如果小于中间键则在左半部份查找；大于则走有伴部分查找。

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/rank.png)
> 在有序数组中使用二分法查找排名的轨迹


性质：**在 N 个键的有序数组中进行二分查找最多需要 (lgN + 1) 次比较（无论是否成功）**

简单的符号表实现的成本总结

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/srh835hk7edgfj6ggdfsswt.png)
> 
> 表中给出的是总成本中的最高级项（对于二分查找是数组的访问次数，对于其他则是比较次数），即运行时间的增长数量级。


### 符号表的各种实现的优缺点

> ![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/github/blog/algorithm7/jlkdsgf6sdfg9sdf678dfgh.png)

### 说明
本文内容是对『算法』第四版 的摘要总结！  
文章所用图片部分来自 [『算法』第四版 英文原版](http://algs4.cs.princeton.edu/home/) ，部分来自 **『算法』第四版 中文译版**

