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

# 微博热门抓取

``` python
import requests
from pyquery import PyQuery as pq
from pymongo import MongoClient
import time

headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
    'Referer': 'https://m.weibo.cn/',
    'X-Requested-With': 'XMLHttpRequest',
    'Accept': 'application/json, text/plain, */*',
    'Host': 'm.weibo.cn'
}

base_url = 'https://m.weibo.cn/api/container/getIndex?containerid=102803&openApp=0&page={}'

def get_page(page):
    try:
        url = base_url.format(page)
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
        return None
    except requests.ConnectionError as e:
        print('error====', e.args)
        return None

def parse_page(json):
    data = json.get('data')
    cards = data.get('cards')
    for card in cards:
        mblog = card.get('mblog', None)
        if mblog:
            weibo = {}
            weibo['id'] = mblog.get('id')
            weibo['text'] = pq(mblog.get('text')).text()
            weibo['comments'] = mblog.get('comments_count')
            weibo['attitudes'] = mblog.get('attitudes_count')
            weibo['reposts'] = mblog.get('reposts_count')
            yield weibo

client = MongoClient()
weibo = client.weibo
remen = weibo.remen

if __name__ == '__main__':
    for page in range(1, 11):
        json = get_page(page)
        for weibo in parse_page(json):
            remen.insert_one(weibo)
        print('爬取 {} 页...'.format(page))
        time.sleep(1)
```

# 头条街拍图片抓取

``` python
import requests
import os
import time
from hashlib import md5
from urllib.parse import urlencode
from concurrent import futures


headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
    'Referer': 'https://www.toutiao.com/search/?keyword=%E8%A1%97%E6%8B%8D',
    'X-Requested-With': 'XMLHttpRequest',
    'Accept': 'application/json, text/javascript',
    'content-type': 'application/x-www-form-urlencoded',
}

# img_base_url = 'http://p3.pstatp.com/origin/pgc-image/{}'
page_base_url = 'https://www.toutiao.com/search_content/?'

def get_page(offset):
    data = {
        'offset': offset,
        'format': 'json',
        'keyword': '街拍',
        'autoload': True,
        'count': 20,
        'cur_tab': 1,
        'from': 'search_tab'
    }
    url = page_base_url + urlencode(data)
    try:
        response = requests.get(url, headers=headers)
        if response.status_code == 200:
            return response.json()
        return None
    except requests.ConnectionError as e:
        print('error', e.args)
        return None

def get_img_urls(json):
    data = json.get('data')
    for item in data:
        image_list = item.get('image_list', None)
        if image_list:
            for img_url in image_list:
                # img_name = os.path.basename(img_url.get('url'))
                yield img_url.get('url')[2:]

def img_counter():
    count = 0
    def counter():
        nonlocal count
        count += 1
        print('已保存 {} 张图片'.format(count))
    return counter

img_add_msg = img_counter()

def save_img(url):
    if not os.path.exists('img'):
        os.mkdir('img')
    try:
        response = requests.get("http://"+url)
        if response.status_code == 200:
            file_md5 = md5(response.content).hexdigest()
            filename = '{}/{}.{}'.format('img', file_md5, 'jpg')
            if not os.path.exists(filename):
                with open(filename, 'wb') as f:
                    f.write(response.content)
                img_add_msg()
    except requests.ConnectionError as e:
        print('error', e.args)
        

if __name__ == '__main__':
    with futures.ThreadPoolExecutor(max_workers=10) as executor:
        for offset in range(0, 100, 20):
            json = get_page(offset)
            for img_url in get_img_urls(json):
                executor.submit(save_img, img_url)
            time.sleep(1)

```