# 安装

```
pip install mitmproxy
```

# 证书配置

首先，运行以下命令产生 CA 证书，并启动 mitmdump：

``` 
mitmdump
```

接下来，我们就可以在「用户目录」下的 `.mitmproxy` 目录里面找到 CA 证书

```
$ pwd
/home/jianxin/.mitmproxy
$ ll
总用量 32
drwxrwxr-x  2 jianxin jianxin 4096 7月   4 13:55 ./
drwxr-xr-x 32 jianxin jianxin 4096 7月   4 13:55 ../
-rw-rw-r--  1 jianxin jianxin 1318 7月   4 13:55 mitmproxy-ca-cert.cer
-rw-rw-r--  1 jianxin jianxin 1140 7月   4 13:55 mitmproxy-ca-cert.p12
-rw-rw-r--  1 jianxin jianxin 1318 7月   4 13:55 mitmproxy-ca-cert.pem
-rw-rw-r--  1 jianxin jianxin 2529 7月   4 13:55 mitmproxy-ca.p12
-rw-rw-r--  1 jianxin jianxin 3022 7月   4 13:55 mitmproxy-ca.pem
-rw-rw-r--  1 jianxin jianxin  770 7月   4 13:55 mitmproxy-dhparam.pem

```

证书说明

![](http://os6ycxx7w.bkt.clouddn.com/20180704140316.png)

# 设置代理

再电脑端开启服务，默认是 `8080` 端口

```
mitmproxy
```

在 android 端安装 `mitmproxy-ca-cert.cer` 证书，并配置代理到对应的 `8080` 端口。

配置好后打开 app 请求网络，电脑端即可看到数据。

![](http://os6ycxx7w.bkt.clouddn.com/20180704144815.png)
![](http://os6ycxx7w.bkt.clouddn.com/images/20180704144945.png)

# 命令

mitmproxy 中的命令

| 命令 | 说明
|:-----------
| q | 退出，返回
| e | 选择编辑项
| ESC | 取消编辑，编辑条目是按 ESC 是确定编辑内容
| a | 增加一个条目，比如在编辑 query 的时候增加一个参数。在 flow details 界面是保存修改
| r | 在 flow details 界面是重新发起请求

# mitmdump

将数据保存到文件中

`mitmdump -w <outfile>`


用脚本处理请求过程

`script.py`

``` python
def request(flow):
    flow.request.headers['User-Agent'] = 'MitmProxy'
    print(flow.request.headers)
```

执行命令

```
mitmdump -s script.py
```

然后 android 端请求 `http://httpbin.org/get`，就可以发现 `User-Agent` 已经被修改了，数据也在终端打印出来了

## 日志输出

``` python
from mitmproxy import ctx

def request(flow):
    flow.request.headers['User-Agent'] = 'MitmProxy'
    ctx.log.info(str(flow.request.headers))
    ctx.log.warn(str(flow.request.headers))
    ctx.log.error(str(flow.request.headers))
```

## Request

``` python
from mitmproxy import ctx

def request(flow):
    request = flow.request
    info = ctx.log.info
    info(request.url)
    info(str(request.headers))
    info(str(request.cookies))
    info(request.host)
    info(request.method)
    info(str(request.port))
    info(request.scheme)
```

## Response

``` python
from mitmproxy import ctx

def response(flow):
    response = flow.response
    info = ctx.log.info

    info(str(response.status_code))
    info(str(response.headers))
    info(str(response.cookies))
    info(str(response.text))
```

# 得到 app 电子书目录抓取

``` python
import json
from pymongo import MongoClient


def response(flow):
    client = MongoClient('mongodb://127.0.0.1:27017/')
    igetget = client.igetget
    books = igetget.books

    url = 'https://dedao.igetget.com/v3//discover/bookList'
    if flow.request.url.startswith(url):
        text = flow.response.text
        data = json.loads(text)
        items = data.get('c').get('list')
        for item in items:
            data = {
                'title': item.get('operating_title'),
                'cover': item.get('cover'),
                'summary': item.get('other_share_summary'),
                'price': item.get('price'),
                'author': item.get('author')
            }
            books.insert_one(data)
            print(data)
```

!> MongoDB 连接语句不能放在函数外面，不然会报错，暂时不知道为什么。