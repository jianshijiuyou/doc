# 基本用法

Beautiful Soup 支持 Python 标准库中的 HTML 解析器，还支持一些第三方的解析器，如果我们不安装它，则 Python 会使用  Python 默认的解析器，lxml 解析器更加强大，速度更快，推荐安装。

|解析器	|使用方法	|优势	|劣势|
|:-----|:---------|:------|:---------------
|Python标准库	| `BeautifulSoup(markup, 'html.parser')`|	Python的内置标准库，执行速度适中，文档容错能力强 |Python 2.7.3 or 3.2.2 前的版本中文档容错能力差
|lxml HTML 解析器	|`BeautifulSoup(markup, 'lxml')`	|速度快，文档容错能力强|需要安装 C 语言库
|lxml XML 解析器|	`BeautifulSoup(markup, 'xml')`	|速度快，唯一支持XML的解析器|需要安装 C 语言库
|html5lib	|`BeautifulSoup(markup, 'html5lib')`	|最好的容错性，以浏览器的方式解析文档生成 HTML5 格式的文档|速度慢，不依赖外部扩展

``` python
html = """
<html><head><title>The Dormouse's story</title></head>
<body>
<p class="title" name="dromouse"><b>The Dormouse's story</b></p>
<p class="story">Once upon a time there were three little sisters; and their names were
<a href="http://example.com/elsie" class="sister" id="link1"><!-- Elsie --></a>,
<a href="http://example.com/lacie" class="sister" id="link2">Lacie</a> and
<a href="http://example.com/tillie" class="sister" id="link3">Tillie</a>;
and they lived at the bottom of a well.</p>
<p class="story">...</p>
"""

from bs4 import BeautifulSoup
soup = BeautifulSoup(html, 'lxml')
print(soup.prettify())
print(soup.title.string)
```

# 节点选择器

## 选择元素

``` python
soup = BeautifulSoup(html, 'lxml')
print(soup.title)
print(type(soup.title))
print(soup.title.string)
print(soup.head)
print(soup.p) # 只会获取第一个 p 节点

# <title>The Dormouse's story</title>
# <class 'bs4.element.Tag'>
# The Dormouse's story
# <head><title>The Dormouse's story</title></head>
# <p class="title" name="dromouse"><b>The Dormouse's story</b></p>
```

## 提取信息

用 name 获取名称

``` python
soup.title.name
```

用 attrs 获取属性

``` python
soup.p.attrs
soup.p.attrs['name']

# {'class': ['title'], 'name': 'dromouse'}
# dromouse
```

简写方式

``` python
soup.p['class']
soup.p['name']

# ['title']
# dromouse
```

用 string 获取文本内容

``` python
soup.p.string

# The Dormouse's story
```

## 嵌套选择

``` python
print(soup.head.title)
print(type(soup.head.title))
print(soup.head.title.string)

# <title>The Dormouse's story</title>
# <class 'bs4.element.Tag'>
# The Dormouse's story
```

## 关联选择

### 子节点和子孙节点

contents 返回直接子节点，结果是一个列表

``` python
soup.p.contents
```

还可以调用 children，结果是一个生成器类型

``` python
for child in soup.p.children:
    print(child)
```

descendants 返回所有子孙节点，结果是一个生成器

``` python
soup.p.descendants
```

### 父节点和祖先节点

parent 返回直接父节点

``` python
soup.a.parent
```

parents 返回所有祖先节点

``` python
soup.a.parents
```

### 兄弟节点

``` python
soup.a.next_sibling  # 下一个
soup.a.previous_sibling # 上一个
soup.a.next_siblings
soup.a.previous_siblings
```

### 提取信息

``` python
soup.a.next_sibling.string
soup.a.parents[0].attrs['class']
```

# 方法选择器

## find_all()

`find_all(name=None, attrs={}, recursive=True, text=None, limit=None, **kwargs)`

根据节点名查询

``` python
soup.find_all(name='ul') # 返回所有的 ul 节点
```

根据属性查询

``` python
soup.find_all(attrs={'name': 'dromouse'})
soup.find_all(class_='sister') # 避免关键字冲突
soup.find_all(id='link2') # 常用的可以不用 attrs 属性
```

根据文本查找

可以是普通文本或者正则表达式

``` python
soup.find_all(text=re.compile('story'))
```

## find()

和 find_all() 一样，只不过返回的是第一个结果，而不是所有的

## 其他

find_parents(), find_parent(): 所有祖先节点，直接父节点

find_next_siblings(), find_next_sibling(): 后面所有兄弟节点，后面第一个兄弟节点

find_previous_siblings(), find_previous_sibling(): 前面所有兄弟节点，前面第一个兄弟节点

find_all_next(), find_next(): 节点后所有符合条件的节点，节点后第一个符合条件的节点

find_all_previous(), find_prevous(): 节点前面所有符合条件的节点，节点前面第一个符合条件的节点

## CSS 选择器

``` python
soup.select('.panel .panel-heading')
soup.select('ul li')
soup.select('#list-2 .element')
```

# 知乎发现列表抓取

``` python
import requests
from bs4 import BeautifulSoup


headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
}
url = 'https://www.zhihu.com/explore'
text = requests.get(url, headers=headers).text
html = BeautifulSoup(text, 'lxml')
items = html.find_all(class_='explore-feed feed-item')
with open('zhihu.txt', 'w') as f:
    for item in items:
        question = item.h2.a.string
        author_ = item.find(class_='author-link-line')
        author = author_.a.string if author_.a else author_.span.string
        answer = item.find(class_='content').string
        print(question, file=f)
        print(author, file=f)
        print(answer, file=f)
        print('='*30, file=f)

```