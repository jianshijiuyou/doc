# 初始化

## 字符串初始化

``` python
html = '''
<div>
<ul>
<li class="item-0">first item</li>
<li class="item-1"><a href="link2.html">second item</a></li>
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-1 active"><a href="link4.html">fourth item</a></li>
<li class="item-0"><a href="link5.html">fifth item</a></li>
</ul>
</div>
'''

from pyquery import PyQuery as pq
doc = pq(html)
print(doc('li'))

# <li class="item-0">first item</li>
# <li class="item-1"><a href="link2.html">second item</a></li>
# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
# <li class="item-1 active"><a href="link4.html">fourth item</a></li>
# <li class="item-0"><a href="link5.html">fifth item</a></li>
```

## URL 初始化

``` python
from pyquery import PyQuery as pq
doc = pq(url='http://jiuyou.info')
print(doc('title'))

# <title>剑弑九幽的博客</title>
```

## 文件初始化

``` python
from pyquery import PyQuery as pq
doc = pq(filename='test.html')
print(doc)
```

# 基本 CSS 选择器

``` python
html = '''
<div id="container">
<ul class="list">
<li class="item-0">first item</li>
<li class="item-1"><a href="link2.html">second item</a></li>
<li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
<li class="item-1 active"><a href="link4.html">fourth item</a></li>
<li class="item-0"><a href="link5.html">fifth item</a></li>
</ul>
</div>
'''

from pyquery import PyQuery as pq
doc = pq(html)
print(doc('#container .list li'))
print(type(doc('#container .list li')))

# <li class="item-0">first item</li>
# <li class="item-1"><a href="link2.html">second item</a></li>
# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
# <li class="item-1 active"><a href="link4.html">fourth item</a></li>
# <li class="item-0"><a href="link5.html">fifth item</a></li>
# 
# <class 'pyquery.pyquery.PyQuery'>
```

# 查找节点

## 子节点

``` python
from pyquery import PyQuery as pq
doc = pq(html)
items = doc('.list')
print(type(items))
print(items)
lis = items.find('li')
print(type(lis))
print(lis)

# <class 'pyquery.pyquery.PyQuery'>
# <ul class="list">
# <li class="item-0">first item</li>
# <li class="item-1"><a href="link2.html">second item</a></li>
# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
# <li class="item-1 active"><a href="link4.html">fourth item</a></li>
# <li class="item-0"><a href="link5.html">fifth item</a></li>
# </ul>
# 
# <class 'pyquery.pyquery.PyQuery'>
# <li class="item-0">first item</li>
# <li class="item-1"><a href="link2.html">second item</a></li>
# <li class="item-0 active"><a href="link3.html"><span class="bold">third item</span></a></li>
# <li class="item-1 active"><a href="link4.html">fourth item</a></li>
# <li class="item-0"><a href="link5.html">fifth item</a></li>

```

`find` 方法查找的是所有的子孙节点，如果只想查找子节点，可以用 `children` 方法。

``` python
items = doc('.list')
lis = items.children()
```

可以同时传入 CSS 选择器

``` python
items = doc('.list')
lis = items.children('.active')
```

## 父节点

直接父节点

``` python
items = doc('.list')
container = items.parent()
```

祖先节点，可能有多个

``` python
items = doc('.list')
container = items.parents()
```

返回符合条件的祖先节点

``` python
items = doc('.list')
container = items.parents('.wrap')
```

## 兄弟节点

``` python
doc = pq(html)
li = doc('.list .item-0.active')
print(li.siblings())
```

做筛选

``` python
doc = pq(html)
li = doc('.list .item-0.active')
print(li.siblings('.active'))
```

# 遍历

不过结果是多个还是单个 pyquery 返回的都是 PyQuery 类型。

对于单个节点，可以直接打印或者转换成字符串

``` python
doc = pq(html)
li = doc('.item-0.active')
print(li)
print(str(li))
```

对于多个节点可以直接遍历

``` python
doc = pq(html)
lis = doc('li').items()
for li in lis:
    print(li)
```

# 获取信息

## 获取属性

``` python
doc = pq(html)
a = doc('.item-0.active a')
print(a, type(a))
print(a.attr('href'))

# <a href="link3.html"><span class="bold">third item</span></a> <class 'pyquery.pyquery.PyQuery'>
# link3.html

```

或者

``` python
a.attr.href
```

## 获取文本

``` python
doc = pq(html)
a = doc('.item-0.active a')
print(a)
print(a.text())

# <a href="link3.html"><span class="bold">third item</span></a>
# third item

```

!> text 会忽略节点中的所有 HTML ，返回纯文本

如果想带 HTML，请使用 html() 方法

``` python
doc = pq(html)
a = doc('.item-0.active a')
print(a)
print(a.html())

# <a href="link3.html"><span class="bold">third item</span></a>
# <span class="bold">third item</span>

```

!> 当结果是多个节点的时候，调用 text 方法会将多个节点的纯文本合并成一个字符串，其中用空格分隔开

# 节点操作

``` python
doc = pq(html)
a = doc('.item-0.active a')
a.removeClass('active')
a.addClass('active')
a.attr('name', 'link')
a.text('changed item')
a.html('<span>23333</span>')
```

还可以删除节点

``` python
html = '''
<div class="wrap">
    Hello, World
    <p>This is a paragraph.</p>
</div>
'''

from pyquery import PyQuery as pq
doc = pq(html)
div = doc('.wrap')
div.children('p').remove() # 删除
print(div.text())

# Hello, World
```

# 伪类选择器

``` python
from pyquery import PyQuery as pq
doc = pq(html)
li = doc('li:first-child')
li = doc('li:last-child')
li = doc('li:nth-child(2)') # 第二个 li 节点
li = doc('li:gt(2)') # 第三个 li 之后的 li 节点
li = doc('li:nth-child(2n)') # 偶数位置 li 节点
li = doc('li:contains(second)') # 包含 second 文本的 li 节点
```
