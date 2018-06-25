# urllib

在 python2 中有 urllib 和 urllib2，在 Python3 中统一为 urllib 库。

主要有 4 个模块

* request 基本请求模块
* error 异常处理模块
* parse 提供 URL 的处理方法，如拆分、解析、合并等。
* robotparser 不常用

## urlopen

`def urlopen(url, data=None, timeout=socket._GLOBAL_DEFAULT_TIMEOUT,
            *, cafile=None, capath=None, cadefault=False, context=None):`

``` python
response = request.urlopen('https://www.python.org')
# print(response.read().decode('utf-8'))

print(response.status)
print(response.getheaders())
print(response.getheader('Server'))
```

data 参数

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
```