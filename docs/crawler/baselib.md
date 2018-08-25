# urllib

在 python2 中有 urllib 和 urllib2，在 Python3 中统一为 urllib 库。

主要有 4 个模块

* request 基本请求模块
* error 异常处理模块
* parse 提供 URL 的处理方法，如拆分、解析、合并等。
* robotparser 不常用

## urlopen 方法

`def urlopen(url, data=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT,
            *, cafile=None, capath=None, cadefault=False, context=None):`

``` python
response = request.urlopen('https://www.python.org')
# print(response.read().decode('utf-8'))

print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```

`data` 参数

使用了 data 参数，请求方式就编程 POST 了，不再是默认的 GET 请求。

``` python
from urllib import request
from urllib import parse

data = bytes(parse.urlencode({'word': 'hello'}), encoding='utf-8')
response = request.urlopen('http://httpbin.org/post', data=data)
print(response.read().decode('utf-8'))
```

在终端用 `python -m json.tool` 可以看到格式化的 json 数据

```
$ python urllib_test.py | python -m json.tool
{
    "args": {},
    "data": "",
    "files": {},
    "form": {
        "word": "hello"
    },
    "headers": {
        "Accept-Encoding": "identity",
        "Connection": "close",
        "Content-Length": "10",
        "Content-Type": "application/x-www-form-urlencoded",
        "Host": "httpbin.org",
        "User-Agent": "Python-urllib/3.6"
    },
    "json": null,
    "origin": "118.112.57.14",
    "url": "http://httpbin.org/post"
}

```

`timeout` 参数

用于设置超时时间

``` python
from urllib import request
from urllib import error
import socket


try:
    response = request.urlopen('http://httpbin.org/get', timeout=0.1)
    print(response.read())
except error.URLError as e:
    if isinstance(e.reason, socket.timeout):
        print('TIME OUT')
```

`context` 参数是 `ssl.SSLContext` 类型，用来指定 SSL 设置

`cafile` 和 `capath` 参数用于指定 CA 证书和它的路径，在请求 HTTPS 链接是会用到

## Request 类

可以通过 `Request` 类来构造一个请求。

`Request(url, data=None, headers={}, origin_req_host=None, unverifiable=False, method=None)`

``` python
from urllib import request

req = request.Request('https://python.org')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

* `data` 参数必须是 `bytes` 类型的，如果要传字典，可以用 `urllib.parse` 中的 `urlencode()` 编码。
* `headers` 是一个字典，用于构造请求头。请求头也可以通过 `add_header()` 方法添加。
* `origin_req_host` 是指请求方的 host 名称或 IP 地址。
* `method` 指示请求方法， `GET`、`POST`、`PUT` 等。

``` python
from urllib import request, parse

url = 'http://httpbin.org/post'
header = {
    'Host': 'httpbin.org'
}
data = {
    'name': 'germey'
}
data = bytes(parse.urlencode(data), encoding='utf8')
req = request.Request(url=url, data=data, headers=header, method='POST')
req.add_header('User-agent', 'Mozilla/4.0 (compatible; MSIE 5.6; Windows NT)')
response = request.urlopen(req)
print(response.read().decode('utf-8'))
```

结果

``` json
{
    "args": {},
    "data": "",
    "files": {},
    "form": {
        "name": "germey"
    },
    "headers": {
        "Accept-Encoding": "identity",
        "Connection": "close",
        "Content-Length": "11",
        "Content-Type": "application/x-www-form-urlencoded",
        "Host": "httpbin.org",
        "User-Agent": "Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)"
    },
    "json": null,
    "origin": "118.112.57.14",
    "url": "http://httpbin.org/post"
}

```

## 高级用法

利用各种 Hander 和 Opener 完成更加复杂的功能

### 验证

利用 `HTTPBasicAuthHandler` 完成 Basic 认证。

``` python
from urllib.request import HTTPPasswordMgrWithDefaultRealm, HTTPBasicAuthHandler, build_opener
from urllib.error import URLError

username = 'username'
password = 'password'
url = 'http://localhost:7000/'

p = HTTPPasswordMgrWithDefaultRealm()
p.add_password(None, url, username, password)
auth_handler = HTTPBasicAuthHandler(p)
opener = build_opener(auth_handler)

try:
    result = opener.open(url)
    html = result.read().decode('utf-8')
    print(html)
except URLError as e:
    print(e.reason)
```

### 代理

``` python
from urllib.request import ProxyHandler, build_opener
from urllib.error import URLError

proxy_handler = ProxyHandler({
    'https': '127.0.0.1:1080'
})
opener = build_opener(proxy_handler)

try:
    response = opener.open('https://httpbin.org/get')
    print(response.read().decode('utf-8'))
except URLError as e:
    print(e.reason)
```

### Cookies

打印 cookie

``` python
from urllib.request import HTTPCookieProcessor, build_opener
from http.cookiejar import CookieJar

cookie = CookieJar()
handler = HTTPCookieProcessor(cookie)
opener = build_opener(handler)
response = opener.open('http://www.baidu.com')
for item in cookie:
    print(item.name+'='+item.value)
```

将 cookie 保存到文件

``` python
from urllib.request import HTTPCookieProcessor, build_opener
from http.cookiejar import MozillaCookieJar

cookie = MozillaCookieJar('cookies.txt')
handler = HTTPCookieProcessor(cookie)
opener = build_opener(handler)
response = opener.open('http://www.baidu.com')
cookie.save(ignore_discard=True, ignore_expires=True)
```

从文件读取 cookie

``` python
from urllib.request import HTTPCookieProcessor, build_opener
from http.cookiejar import MozillaCookieJar

cookie = MozillaCookieJar()
cookie.load('cookies.txt', ignore_discard=True, ignore_expires=True)
handler = HTTPCookieProcessor(cookie)
opener = build_opener(handler)
response = opener.open('http://www.baidu.com')
print(response.read().decode('utf-8'))
```

## 处理异常

* `URLError`

    `error` 异常模块的基类。有一个 `reason` 属性，返回错误的原因。

* `HTTPError`

    `URLError` 的子类，专门处理 HTTP 请求错误

    属性有：

    * `code` 返回 HTTP 状态吗
    * `reason` 返回错误原因
    * `headers` 返回请求头

## 解析链接

### urlparse

`def urlparse(url, scheme='', allow_fragments=True):`

实现 URL 的识别和分段

``` python
from urllib.parse import urlparse

result = urlparse('http://baidu.com/index.html;user?id=5#comment')
print(type(result), result)

# <class 'urllib.parse.ParseResult'> ParseResult(scheme='http', netloc='baidu.com', 
# path='/index.html', params='user', query='id=5', fragment='comment')
```

* url 解析的 URL
* scheme 当链接中不包含 scheme 前缀时使用的默认值
* allow_fragments 是否忽略 fragment，如果设置为 False ，它会作为 path 的一部分

获取方法

``` python
result = urlparse('http://baidu.com/index.html;user?id=5#comment')
print(result[0])
print(result.scheme) # 结果同上
```

### urlunparse

`urlunparse([cheme, netloc, url, params, query, fragment])`

用于构造 URL，参数长度必须是 6 

### urlsplit

类似 urlparse，不过不单独解析 params 部分

### urlunsplit

类似 urlunparse

### urljoin

`def urljoin(base, url, allow_fragments=True):`

合并 base 和 url，url 中重复的部分（scheme、netloc、path）会覆盖 base 中的

``` python
print(urljoin('http://www.baidu.com', 'FAQ.html'))
print(urljoin('www.baidu.com', '?category=2#comment'))
print(urljoin('http://www.baidu.com', 'https://cuiqingcai.com/FAQ.html'))

# http://www.baidu.com/FAQ.html
# www.baidu.com?category=2#comment
# https://cuiqingcai.com/FAQ.html
```

### urlencode

可用于构造 GET 请求参数

``` python
from urllib.parse import urlencode

params = {
    'name': 'jack',
    'age': '18'
}
base_url = 'http://www.baidu.com?'
url = base_url + urlencode(params)
print(url)
# http://www.baidu.com?name=jack&age=18
```

### parse_qs

将 GET 请求参数转化为字典

``` python
query = 'name=jack&age=18'
parse_qs(query) # {'name': ['jack'], 'age': ['18']}
```

### parse_qsl

将 GET 参数转化成元组

``` python
query = 'name=jack&age=18'
parse_qsl(query) # [('name', 'jack'), ('age', '18')]
```

### quote

编码 URL，比如将 URL 中的中文转化成 URL 编码

``` python
from urllib.parse import quote

print('https://www.baidu.com/s?wd=' + quote('中文'))
# https://www.baidu.com/s?wd=%E4%B8%AD%E6%96%87
```

### unquote

解码 URL

``` python
unquote('%E4%B8%AD%E6%96%87') # 中文
```