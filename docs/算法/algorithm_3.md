### 归并排序
要将一个数组排序，可以先（递归地）将它分成两半分别排序，然后将结果归并起来。你将会看到，归并排序最吸引人的性质是它能够保证将<font color="red">任意长度为 N 的数组排序所需时间和 NlogN 成正比</font>；它的主要缺点则是它所<font color="red">需要的额外空间和 N 成正比</font>。  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/mergesort-overview.png)  
>归并排序示意图

### 原地归并的抽象方法
> 此方法可以将两个有序的子数组合并成一个有序的数组


``` java
public static void merge(Comparable[] a,int lo,int mid,int hi)
{
	//将 a[lo..mid] 和 a[mid+1..hi] 归并
	int i=lo, j=mid+1

	//将 a[lo..hi] 复制到 aux[lo..hi]
	for(int k=lo;k <= hi;k++){
		aux[k] = a[k]
	}

	//归并回到a[lo..hi]
	for(int k=lo;k <= hi;k++){

		if(i>mid){    //说明前半段已经比对结束，后半段可以直接赋值了
			a[k]=aux[j++]
		} else if(j>hi){ //说明后半段已经比对结束，前半段可以直接赋值了
			a[k]=aux[i++]
		} else if(less(aux[j],aux[i])){  //比对，将较小的赋值回原数组
			a[k]=a[j++]
		} else {
			a[k]=a[i++]
		}
	}
}
```

该方法先将所有元素复制到 aux[] 中，然后再归并回 a[] 中，方法在归并时（第二个 for 循环）进行了 4 个条件判断：
 1. **左半边用尽（取右半边的元素）**
 2. **右半边用尽（取左半边的元素）**
 3. **右半边的当前元素小于左半边的当前元素（取右半边的元素）**
 4. **右半边的当前元素大于左半边的当前元素（取左半边的元素）**

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/merge.png)  
> 原地归并的抽象方法的轨迹

### 自顶向下的归并排序
**原地归并的抽象方法** 能将两个有序的子数组排序，那它就能通过归并将整个无序数组排序
> 在归并无序数组时，当递归到两个子数组的长度都是 1 的时候，可以认为此时两个子数组是有序的。  

``` java
public class Merge {
	//归并所需的辅助数组
	private static Comparable[] aux;

    public static void sort(Comparable[] a) {
        aux = new Comparable[a.length]
        sort(a, 0, a.length-1);
    }

    private static void sort(Comparable[] a, int lo, int hi) {
        if (hi <= lo) return;
        int mid = lo + (hi - lo) / 2;
        sort(a, lo, mid);    //左半边排序
        sort(a, mid + 1, hi);   //右半边排序
        merge(a, lo, mid, hi);  //归并结果
    }
}
```

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/mergesortTD.png)  
> 自顶向下的归并排序中归并结果的轨迹

要理解归并排序就要仔细研究该方法调用的动态情况。  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/algorithm_3_1.png)  
> 自顶向下的归并排序的调用轨迹

命题：<font color="red">对于长度为 N 的任意数组，自顶向下的归并排序需要 1/2NlgN 至 NlgN 次比较。</font>
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/algorithm_3_2.png)   
> N=16 时归并排序中子数组的依赖树

命题：<font color="red">对于长度为 N 的任意数组，自顶向下的归并排序最多需要访问数组 6NlgN 次。</font>  

### 提高性能
#### 对小规模数组使用插入排序  
递归会使小规模问题中的方法的调用过于频繁，所以改进它们的处理方法就能改进整个算法，使用插入排序处理小规模的子数组（比如长度小于 15）一般可以将归并排序的运行时间缩短 10% ～ 15%。  
![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/mergesortTD-bars.png)     
> 改进了小规模子数组排序方法后的自顶向下的归并排序的可视轨迹

#### 测试数组是否已经有序
添加一个判断条件，如果 a[mid] 小于等于 a[mid+1] ，就认为数组已经是有序的并跳过 merge() 方法。这个改动不影响排序的递归调用，但是任意有序的子数组算法的运行时间就变为线性的了。

### 自底向上的归并排序
实现归并排序的另一种方法是先归并那些微型数组，然后再成对归并得到的子数组，知道将整个数组归并在一起。

首先进行两两归并（把每个元素想象成一个大小为 1 的数组），然后四四归并（将两个大小为 2 的数组归并成一个有 4 个元素的数组），然后是八八归并，一直下去。  

最后一次归并的第二个子数组可能比第一个子数组要小（但这对 merge() 方法不是问题），如果不是的话所有的归并中两个数组大小都应该一样，而在下一轮中子数组的大小会翻倍。  

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/algorithm_3_3.png)  
> 自底向上的归并排序的可视轨迹

``` java
public class MergeBU {

	private static Comparable[] aux;

    public static void sort(Comparable[] a) {
        int N = a.length;
        aux = new Comparable[N];
        for (int sz = 1; sz < N; sz = sz+sz) {  //sz 子数组大小
            for (int lo = 0; lo < N-sz; lo += sz+sz) {   //lo 子数组索引
                int mid  = lo+sz-1;
                int hi = Math.min(lo+sz+sz-1, N-1);
                merge(a, lo, mid, hi);
            }
        }

    }
}
```

自底向上的归并排序会多次遍历整个数组，根据子数组大小进行两两归并。子数组的大小 sz 的初始值为 1，每次加倍。最后一个子数组的大小只有在数组大小是 sz 的偶数倍的时候才会等于 sz（否则会比 sz 小）。  

![](http://os6ycxx7w.bkt.clouddn.com/github/blog/algorithm_3/mergesortBU.png)  
> 自底向上的归并排序的归并结果

命题：<font color="red">对于长度为 N 的任意数组，自底向上的归并排序需要 1/2NlgN 至 NlgN 次比较，最多访问数组 6NlgN 次。</font>

命题：<font color="red">没有任何基于比较的算法能够保证使用少于 lg(N!) ~ NlgN 次比较将长度为 N 的数组排序。</font>  

命题：<font color="red">归并排序是一种渐进最优的基于比较排序的算法。</font>   
> 归并排序在最坏情况下的比较次数和任意基于比较的排序算法所需的最少比较次数都是 ～NlgN。


### 说明
本文内容是对『算法』第四版 的摘要总结！  
文章所用图片部分来自 [『算法』第四版 英文原版](http://algs4.cs.princeton.edu/home/) ，部分来自 **『算法』第四版 中文译版**
