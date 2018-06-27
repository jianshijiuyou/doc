> [w3school - xpath](http://www.w3school.com.cn/xpath/index.asp)  
> [lxml.de](https://lxml.de/)

# 基本

在 python 中使用 xpath 需要先安装 lxml 库。

``` python
from lxml import etree

text = '''
<div>
    <ul>
        <li class="item-0"><a href="link1.html">first item</a></li>
        <li class="item-1"><a href="link2.html">second item</a></li>
        <li class="item-inactive"><a href="link3.html">third item</a></li>
        <li class="item-1"><a href="link4.html">fourth item</a></li>
        <li class="item-0"><a href="link5.html">fifth item</a>
    </ul>
</div>
'''

html = etree.HTML(text)
result = etree.tostring(html)
print(result.decode('utf-8'))
```
etree 模块可以自动修正 HTML 文本

结果

``` html
<html><body><div>
    <ul>
        <li class="item-0"><a href="link1.html">first item</a></li>
        <li class="item-1"><a href="link2.html">second item</a></li>
        <li class="item-inactive"><a href="link3.html">third item</a></li>
        <li class="item-1"><a href="link4.html">fourth item</a></li>
        <li class="item-0"><a href="link5.html">fifth item</a>
    </li></ul>
</div>
</body></html>
```

还可以直接从文件中读取解析

``` python
html = etree.parse('test.html', etree.HTMLParser())
result = etree.tostring(html)
```

## 所有节点

``` python
html = etree.parse('test.html', etree.HTMLParser())
result = html.xpath('//*')
print(result)
```
`*` 代表所有节点

结果

```
[<Element html at 0x7f1aff7f3b88>, <Element body at 0x7f1aff7f3c88>, <Element div at 0x7f1aff7f3cc8>, <Element ul at 0x7f1aff7f3d08>, <Element li at 0x7f1aff7f3d48>, <Element a at 0x7f1aff7f3dc8>, <Element li at 0x7f1aff7f3e08>, <Element a at 0x7f1aff7f3e48>, <Element li at 0x7f1aff7f3e88>, <Element a at 0x7f1aff7f3d88>, <Element li at 0x7f1aff7f3ec8>, <Element a at 0x7f1aff7f3f08>, <Element li at 0x7f1aff7f3f48>, <Element a at 0x7f1aff7f3f88>]
```

获取所有 `li` 节点

``` python
result = html.xpath('//li')
```

## 子节点

获取 `li` 节点的所有直接 `a` 子节点

``` python
result = html.xpath('//li/a')
```

获取 `ul` 节点的所有子孙 `a` 节点

``` python
result = html.xpath('//ul//a')
```

!> `/` 用于获取直接子节点，`//` 用于获取子孙节点

## 父节点

利用 `..` 查找父节点

查找 `href` 属性为 `link4.html` 的 `a` 节点，然后再找到其父节点，然后再获取其 `class` 属性

``` python
html.xpath('//a[@href="link4.html"]/../@class')

# ['item-1']
```


或者通过 XPath Axes（轴）来查找

比如用 `parent::` 查找父节点的内容

``` python
html.xpath('//a[@href="link4.html"]/parent::*/@class')
```

## 属性匹配

通过 `@` 符号进行属性过滤

``` python
html.xpath('//li[@class="item-0"]')

# [<Element li at 0x7fc057e52c88>, <Element li at 0x7fc057e52cc8>]
```

## 文本获取

``` python
html.xpath('//li[@class="item-0"]/a/text()')

# ['first item', 'fifth item']
```

## 属性获取

也是通过 `@` 符号

``` python
html.xpath('//li/a/@href')

# ['link1.html', 'link2.html', 'link3.html', 'link4.html', 'link5.html']
```

## 属性多值匹配

如果一个属性有多个值，就不能简单的用 `@` 的方式匹配了，需要用到 `contains` 函数

``` python
text = '<li class="li li-first"><a href="link.html">first item</a></li>'
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "li")]/a/text()')
print(result)

# ['first item']
```

## 多属性匹配

使用 and 运算符进行多个属性的匹配

``` python
text = '<li class="li li-first" name="item"><a href="link.html">first item</a></li>'
html = etree.HTML(text)
result = html.xpath('//li[contains(@class, "li") and @name="item"]/a/text()')
print(result)

# ['first item']
```

## 按序选择

可以利用索引获取特定次序的节点

``` python
html = etree.parse('test.html', etree.HTMLParser())
result = html.xpath('//li[1]/a/text()')
print(result)
result = html.xpath('//li[last()]/a/text()')
print(result)
result = html.xpath('//li[position()<3]/a/text()')
print(result)
result = html.xpath('//li[last()-2]/a/text()')
print(result)

# ['first item']
# ['fifth item']
# ['first item', 'second item']
# ['third item']
```

!> 序号是 1 开头，不是 0，注意了。

## 节点轴选择

``` python
from lxml import etree

text = '''
<div>
    <ul>
        <li class="item-0"><a href="link1.html"><span>first item</span></a></li>
        <li class="item-1"><a href="link2.html">second item</a></li>
        <li class="item-inactive"><a href="link3.html">third item</a></li>
        <li class="item-1"><a href="link4.html">fourth item</a></li>
        <li class="item-0"><a href="link5.html">fifth item</a>
    </ul>
</div>
'''
html = etree.HTML(text)
result = html.xpath('//li[1]/ancestor::*') # 获取所有祖先节点
print(result)
result = html.xpath('//li[1]/ancestor::div') # 获取 div 祖先节点
print(result)
result = html.xpath('//li[1]/attribute::*') # 获取所有属性值
print(result)
result = html.xpath('//li[1]/child::a[@href="link1.html"]') # 获取直接 a 子节点，且 a 子节点必须满足特点条件
print(result)
result = html.xpath('//li[1]/descendant::span') # 获取所有子孙 span 节点
print(result)
result = html.xpath('//li[1]/following::*[2]') # 获取当前节点之后的所有节点中的第二个节点
print(result)
result = html.xpath('//li[1]/following-sibling::*') # 获取当前节点之后的所有同级节点
print(result)

# [<Element html at 0x7f1589781c08>, <Element body at 0x7f1589781b88>, <Element div at 0x7f1589781b48>, <Element ul at 0x7f1589781c48>]
# [<Element div at 0x7f1589781b48>]
# ['item-0']
# [<Element a at 0x7f1589781c48>]
# [<Element span at 0x7f1589781b48>]
# [<Element a at 0x7f1589781c48>]
# [<Element li at 0x7f1589781b88>, <Element li at 0x7f1589781c88>, <Element li at 0x7f1589781cc8>, <Element li at 0x7f1589781d08>]
```