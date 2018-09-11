# 基本用法

``` python
import requests

r = requests.get('https://www.baidu.com/')
print(type(r))
print(r.status_code)
print(type(r.text))
print(r.cookies)
print(r.text)

# <class 'requests.models.Response'>
# 200
# <class 'str'>
# <RequestsCookieJar[<Cookie BDORZ=27315 for .baidu.com/>]>
# ......
```

## GET

### 携带参数

``` python
import requests

data = {
    'name': 'jack',
    'age': 18
}
r = requests.get('http://httpbin.org/get', params=data)
print(r.text)
```

结果

``` json
{
    "args": {
        "age": "18",
        "name": "jack"
    },
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Connection": "close",
        "Host": "httpbin.org",
        "User-Agent": "python-requests/2.19.1"
    },
    "origin": "118.112.57.14",
    "url": "http://httpbin.org/get?name=jack&age=18"
}

```

如果结果是 json 数据，可以直接调用 json() 方法转化成字典

``` python
r.json()
```

### 添加 headers

``` python
import requests

headers = {
    'user-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36'
}
r = requests.get('https://zhihu.com/explore', headers=headers)
print(r.text)
```

### 抓取二进制数据

``` python
r = requests.get('https://github.com/favicon.ico')
with open('favicon.ico', 'wb') as f:
    f.write(r.content)
```

`content` 返回的是 `bytes` 类型数据

## POST

``` python
data = {
    'name': 'jack',
    'age': 18
}
r = requests.post('http://httpbin.org/post', data=data)
print(r.text)
```

结果

``` json
{
    "args": {},
    "data": "",
    "files": {},
    "form": {
        "age": "18",
        "name": "jack"
    },
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Connection": "close",
        "Content-Length": "16",
        "Content-Type": "application/x-www-form-urlencoded",
        "Host": "httpbin.org",
        "User-Agent": "python-requests/2.19.1"
    },
    "json": null,
    "origin": "118.112.57.14",
    "url": "http://httpbin.org/post"
}
```

## JSON

请求发送 json 数据

``` python
>>> import json

>>> url = 'http://httpbin.org/post'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, data=json.dumps(payload))
```

结果

``` json
{
    "args": { }, 
    "data": "{\"some\": \"data\"}", 
    "files": { }, 
    "form": { }, 
    "headers": {
        "Accept": "*/*", 
        "Accept-Encoding": "gzip, deflate", 
        "Connection": "close", 
        "Content-Length": "16", 
        "Host": "httpbin.org", 
        "User-Agent": "python-requests/2.19.1"
    }, 
    "json": {
        "some": "data"
    }, 
    "origin": "125.70.30.147", 
    "url": "http://httpbin.org/post"
}
```

直接使用 json 参数

``` python
>>> url = 'http://httpbin.org/post'
>>> payload = {'some': 'data'}

>>> r = requests.post(url, json=payload)
```

结果

``` json
{
    "args": { }, 
    "data": "{\"some\": \"data\"}", 
    "files": { }, 
    "form": { }, 
    "headers": {
        "Accept": "*/*", 
        "Accept-Encoding": "gzip, deflate", 
        "Connection": "close", 
        "Content-Length": "16", 
        "Content-Type": "application/json", 
        "Host": "httpbin.org", 
        "User-Agent": "python-requests/2.19.1"
    }, 
    "json": {
        "some": "data"
    }, 
    "origin": "221.237.92.7", 
    "url": "http://httpbin.org/post"
}
```

## 响应

``` python
r = requests.get('http://www.jianshu.com/')
print(r.status_code)
print(r.headers)
print(r.cookies)
print(r.url)
print(r.history) # 请求历史
print(r.text)
print(r.content)
# print(r.json())
```

requests 内置了一个状态码查询对象 `requests.codes`

例：`requests.codes.ok` 表示 `200`

``` python
_codes = {

    # Informational.
    100: ('continue',),
    101: ('switching_protocols',),
    102: ('processing',),
    103: ('checkpoint',),
    122: ('uri_too_long', 'request_uri_too_long'),
    200: ('ok', 'okay', 'all_ok', 'all_okay', 'all_good', '\\o/', '✓'),
    201: ('created',),
    202: ('accepted',),
    203: ('non_authoritative_info', 'non_authoritative_information'),
    204: ('no_content',),
    205: ('reset_content', 'reset'),
    206: ('partial_content', 'partial'),
    207: ('multi_status', 'multiple_status', 'multi_stati', 'multiple_stati'),
    208: ('already_reported',),
    226: ('im_used',),

    # Redirection.
    300: ('multiple_choices',),
    301: ('moved_permanently', 'moved', '\\o-'),
    302: ('found',),
    303: ('see_other', 'other'),
    304: ('not_modified',),
    305: ('use_proxy',),
    306: ('switch_proxy',),
    307: ('temporary_redirect', 'temporary_moved', 'temporary'),
    308: ('permanent_redirect',
          'resume_incomplete', 'resume',),  # These 2 to be removed in 3.0

    # Client Error.
    400: ('bad_request', 'bad'),
    401: ('unauthorized',),
    402: ('payment_required', 'payment'),
    403: ('forbidden',),
    404: ('not_found', '-o-'),
    405: ('method_not_allowed', 'not_allowed'),
    406: ('not_acceptable',),
    407: ('proxy_authentication_required', 'proxy_auth', 'proxy_authentication'),
    408: ('request_timeout', 'timeout'),
    409: ('conflict',),
    410: ('gone',),
    411: ('length_required',),
    412: ('precondition_failed', 'precondition'),
    413: ('request_entity_too_large',),
    414: ('request_uri_too_large',),
    415: ('unsupported_media_type', 'unsupported_media', 'media_type'),
    416: ('requested_range_not_satisfiable', 'requested_range', 'range_not_satisfiable'),
    417: ('expectation_failed',),
    418: ('im_a_teapot', 'teapot', 'i_am_a_teapot'),
    421: ('misdirected_request',),
    422: ('unprocessable_entity', 'unprocessable'),
    423: ('locked',),
    424: ('failed_dependency', 'dependency'),
    425: ('unordered_collection', 'unordered'),
    426: ('upgrade_required', 'upgrade'),
    428: ('precondition_required', 'precondition'),
    429: ('too_many_requests', 'too_many'),
    431: ('header_fields_too_large', 'fields_too_large'),
    444: ('no_response', 'none'),
    449: ('retry_with', 'retry'),
    450: ('blocked_by_windows_parental_controls', 'parental_controls'),
    451: ('unavailable_for_legal_reasons', 'legal_reasons'),
    499: ('client_closed_request',),

    # Server Error.
    500: ('internal_server_error', 'server_error', '/o\\', '✗'),
    501: ('not_implemented',),
    502: ('bad_gateway',),
    503: ('service_unavailable', 'unavailable'),
    504: ('gateway_timeout',),
    505: ('http_version_not_supported', 'http_version'),
    506: ('variant_also_negotiates',),
    507: ('insufficient_storage',),
    509: ('bandwidth_limit_exceeded', 'bandwidth'),
    510: ('not_extended',),
    511: ('network_authentication_required', 'network_auth', 'network_authentication'),
}
```

# 高级用法

## 文件上传

``` python
files = {'file': open('favicon.ico', 'rb')}
r = requests.post('http://httpbin.org/post', files=files)
print(r.text)
```

结果

``` json
{
    "args": {},
    "data": "",
    "files": {
        "file": "data:application/octet-stream;base64,AAABAAIAEBAAA......AEAIAAoBAAA="
    },
    "form": {},
    "headers": {
        "Accept": "*/*",
        "Accept-Encoding": "gzip, deflate",
        "Connection": "close",
        "Content-Length": "6665",
        "Content-Type": "multipart/form-data; boundary=3967e7e78dd534d68b7360368e7ac24a",
        "Host": "httpbin.org",
        "User-Agent": "python-requests/2.19.1"
    },
    "json": null,
    "origin": "118.112.57.14",
    "url": "http://httpbin.org/post"
}
```

## Cookies

### 获取 cookie

``` python
r = requests.get('https://www.baidu.com/')
for key, value in r.cookies.items():
    print(key + '=' + value)

# BAIDUID=C01847C89218E34C92E9384A0FC3160F:FG=1
# BIDUPSID=C01847C89218E34C92E9384A0FC3160F
# H_PS_PSSID=1460_21086_26350_22159
# PSTM=1530002647
# BDSVRTM=0
# BD_HOME=0
```

### 利用 cookie 维持登陆状态

以知乎为例，先用浏览器登陆获取 cookie

``` python
import requests

headers = {
    'Cookie': 'xxxxxx',
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
    'Host': 'www.zhihu.com'
}
r = requests.get('https://www.zhihu.com/people/tecakam/activities', headers=headers)
print(r.text)
```

结果中如果出现你的昵称说明成功了

也可以单独构造 cookie 对象

``` python
import requests

headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
    'Host': 'www.zhihu.com'
}
cookies = 'xxxxxxxx'
jar = requests.cookies.RequestsCookieJar()
for cookie in cookies.split(';'):
    key, value = cookie.split('=', 1) # 分割 1 次
    jar.set(key, value)
r = requests.get('https://www.zhihu.com/people/tecakam/activities',cookies=jar , headers=headers)
print(r.text)
```

## 会话维持

之前的请求，每一次都相当于打开了一个浏览器，请求之间互相没有关联。

``` python
import requests

requests.get('http://httpbin.org/cookies/set/number/123456789')
r = requests.get('http://httpbin.org/cookies')
print(r.text)

# {
#     "cookies": {}
# }
```

可以看到，第二次请求并没有第一次设置的 cookie 信息。

使用 session 对象可以维持会话。

``` python
import requests

s = requests.Session()
s.get('http://httpbin.org/cookies/set/number/123456789')
r = s.get('http://httpbin.org/cookies')
print(r.text)

# {
#     "cookies": {
#         "number": "123456789"
#     }
# }
```

可以发现，cookie 已经被自动保存了

## SSL 证书验证

``` python
r = requests.get('https://www.12306.cn')
print(r.status_code)
```

直接访问 12306 会报错，因为它的证书是自己做的，我们可以忽略验证

``` python
r = requests.get('https://www.12306.cn', verify=False)
print(r.status_code)
```

这样就不会报错了，但是会给出警告，忽略警告

``` python
import requests
from requests.packages import urllib3

urllib3.disable_warnings()
r = requests.get('https://www.12306.cn', verify=False)
print(r.status_code)
```

指定本地证书用作客户端证书

``` python
r = requests.get('https://www.12306.cn', cert=('/path/server.crt', '/path/key'))
print(r.status_code)
```

本地证书的 key 必须是解密状态

## 代理设置

``` python
import requests

proxies = {
    'http': 'http://127.0.0.1:1080',
    'https': 'https://127.0.0.1:1080'
}
r = requests.get('http://httpbin.org/get', proxies=proxies)
print(r.text)
```

如果代理需要 Basic auth

``` python
proxies = {
    'http': 'http://user:paaword@127.0.0.1:1080',
    'https': 'https://user:paaword@127.0.0.1:1080'
}
r = requests.get('https://www.google.com', proxies=proxies)
print(r.text)
```

支持 socks5 代理

先安装 socks 库

`pip install 'requests[socks]'`

实际是安装 `PySocks`


然后

``` python
proxies = {
    'http': 'socks5://user:pwd@host:port',
    'https': 'socks5://user:pwd@host:port'
}
requests.get('https://www.google.com', proxies=proxies)
```

## 超时设置

``` python
r = requests.get('https://www.google.com', timeout=1)
print(r.status_code)
```

请求分为 连接（connect）和读取（read）两个阶段

上面的 timeout 是设置的它们的总和，也可以分开设置

``` python
r = requests.get('https://www.google.com', timeout=(3, 2))
print(r.status_code)
```

将 timeout 设置为 None 或者不设置为永不超时

``` python
r = requests.get('https://www.google.com', timeout=None)
print(r.status_code)
```

## 身份认证

Basic Auth

``` python
import requests
from requests.auth import HTTPBasicAuth

r = requests.get('http://localhost:8000', auth=HTTPBasicAuth('username', 'pssword'))
# 简写 requests.get('http://localhost:8000', auth=('username', 'pssword'))
print(r.status_code
```

其他认证，如 OAuth1 等，请参考官方文档

[https://github.com/requests/requests-oauthlib](https://github.com/requests/requests-oauthlib)

# 猫眼电影TOP100

``` python
import requests
import re
import time
import json
from requests.exceptions import RequestException


headers = {
    'User-agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.87 Safari/537.36',
}

def get_one_page(url):
    try:
        r = requests.get(url, headers=headers)
        if r.status_code == requests.codes.ok:
            return r.text
        return None
    except RequestException:
        return None

def parse_one_page(html):
    pattern = re.compile(r'<dd>.*?board-index.*?>(\d.*?)</i>.*?<img data-src="(.*?)".*?' +
                         r'<p class="name"><a.*?>(.*?)</a></p>.*?' +
                         r'<p class="star">(.*?)</p>.*?' +
                         r'<p class="releasetime">(.*?)</p>.*?' + 
                         r'<p class="score"><i class="integer">(.*?)</i><i class="fraction">(.*?)</i></p>', re.S)
    source = pattern.finditer(html)
    for item in source:
        yield {
            'index': item[1],
            'image': item[2],
            'title': item[3],
            'actor': item[4].strip()[3:],
            'time': item[5][5:],
            'score': item[6] + item[7]
        }

def main(url):
    html = get_one_page(url)
    data = parse_one_page(html)
    with open('maoyan.txt', 'a') as f:
        for item in data:
            f.write(json.dumps(item, ensure_ascii=False) + '\n')

if __name__ == '__main__':
    base = 'http://maoyan.com/board/4?offset={}'
    for i in range(0, 100, 10):
        url = base.format(i)
        main(url)
        time.sleep(1)
```