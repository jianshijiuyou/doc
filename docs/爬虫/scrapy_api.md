# Selector

##  直接单独使用

Selector 可以单独使用

``` python
from scrapy import Selector

body = '''
<html>
    <head>
        <title>Hello World</title>
    </head>
    <body>
    </body>
</html>
'''

selector = Selector(text=body)
title = selector.xpath('//title/text()').extract_first()
# selector.css('xxxx').extract_first()
print(title) # Hello World
```

## XPath 选择器

两种调用方式

``` python
response.selector.xpath()
response.xpath()
```

使用案例

``` python
response.xpath('//a') # 所有 a 标签
response.xpath('./img') # 元素内部的 img 标签，不加 . 默认从根路径算起
response.xpath('//a/text()') # a 标签中的文本
response.xpath('//a/@href') # a 标签中的 href 属性
response.xpath('//a[@href="image1.html"]/text()') 
response.xpath('//a').extract() # 返回结果的列表形式
response.xpath('//a').extract()[0] # 返回列表的第一项，有风险，可能越界，推荐 extract_first
response.xpath('//a').extract_first() # 返回列表的第一项，没有元素就返回空字符串
response.xpath('//a').extract_first('default val') # 默认值
```

## CSS 选择器

两种调用方式

``` python
response.selector.css()
response.css()
```

使用案例

``` python
response.css('a') # 所有 a 标签
response.css('a img') # a 标签内部的 img 标签
response.css('a::text') # a 标签中的文本
response.css('a::attr(href)') # a 标签中的 href 属性
response.css('a[href="image1.html"]::text') 
response.css('a[href="image1.html"] img::attr(src)') 
response.css('a').extract() # 返回结果的列表形式
response.css('a').extract()[0] # 返回列表的第一项，有风险，可能越界，推荐 extract_first
response.css('a').extract_first() # 返回列表的第一项，没有元素就返回空字符串
response.css('a').extract_first('default val') # 默认值
```

混合使用

``` python
response.css('xxx').xpath('xxx').css('xxx').xpath('xxx')
```

## 正则匹配

``` python
response.xpath('.').re('xxxx')
response.xpath('.').re_first('xxxx')
```

!> re 不能直接使用，需要先调用 xpath 或 css 然后再调用 re

# Spider

Spider 有两个作用

* 定义爬取网站的动作
* 分析爬取下来的的网页

Spider 类的一些基础属性和方法

* `name`: 爬虫名称，启动爬虫时需要，必须唯一，一般命名为网站的域名
* `allowed_domains`: 允许爬取的域名，可选配置
* `start_urls`: 起始 url 列表
* `custom_settings`: 一个字典，专属的配置，会覆盖全局的 settings 配置
* `crawler`: 由 `from_crawler()` 方法设置，代表本 Spider 类的 `Crawler` 对象，包含很多组件，可以获取项目的配置信息。
* `settings`: 一个 settings 对象，可以直接获取项目的全局设置信息
* `start_requests()`: 用于生成初始请求，必须返回可迭代对象。默认使用 `start_urls` 里面的 `url` 来构造 `Rquest`，而且 `Request` 时 `GET` 请求方式，如果想再启动时以 `POST` 方式访问站点，可以直接重写该方法，发送 `POST` 请求时使用 `FormRequest` 即可。
* `parse()`: 当 `Response` 没有指定回调函数时，次方法默认调用。负责处理 `Response`，处理返回结果，并从中提取数据和下一步请求，然后返回。该方法需要返回一个包含 `Request` 或 `Item` 的可迭代对象
* `closed()`: 当 Spider 关闭时，该方法会被调用，一般用于释放资源和收尾工作。

# Downloader Middleware

作用

* 在 Scheduler 调度出队列的 `Request` 发送给 Downloader 下载之前，对其进行修改。
* 在下载后生成 `Response` 发送给 Spider 之前，对其进行修改。

修改 User-Agent、处理重定向、设置代理、失败重试、设置 Cookies 等都要利用它来实现

## 使用说明

Scrapy 内置了许多 Downloader Middleware，比如负责失败重试、自动重定向等功能。它们被 `DOWNLOADER_MIDDLEWARES_BASE` 变量所定义。

数字越小越靠近 Scrapy 引擎，越大越靠近 Downloader。

顺序很重要，因为中间件之间可能有依赖关系

``` python
{
    'scrapy.downloadermiddlewares.robotstxt.RobotsTxtMiddleware': 100,
    'scrapy.downloadermiddlewares.httpauth.HttpAuthMiddleware': 300,
    'scrapy.downloadermiddlewares.downloadtimeout.DownloadTimeoutMiddleware': 350,
    'scrapy.downloadermiddlewares.defaultheaders.DefaultHeadersMiddleware': 400,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': 500,
    'scrapy.downloadermiddlewares.retry.RetryMiddleware': 550,
    'scrapy.downloadermiddlewares.ajaxcrawl.AjaxCrawlMiddleware': 560,
    'scrapy.downloadermiddlewares.redirect.MetaRefreshMiddleware': 580,
    'scrapy.downloadermiddlewares.httpcompression.HttpCompressionMiddleware': 590,
    'scrapy.downloadermiddlewares.redirect.RedirectMiddleware': 600,
    'scrapy.downloadermiddlewares.cookies.CookiesMiddleware': 700,
    'scrapy.downloadermiddlewares.httpproxy.HttpProxyMiddleware': 750,
    'scrapy.downloadermiddlewares.stats.DownloaderStats': 850,
    'scrapy.downloadermiddlewares.httpcache.HttpCacheMiddleware': 900,
}
```
这些都是默认启用的，不要自己修改。

自定义的中间件应该放在 `DOWNLOADER_MIDDLEWARES` 中

``` python
DOWNLOADER_MIDDLEWARES  =  { 
    'myproject.middlewares.CustomDownloaderMiddleware' ： 543 ，
}
```

## 核心方法

下载中间件的核心方法有三个，自定义下载中间件至少需要实现其中一个方法

* `process_request(request, spider)`

    对于通过下载中间件的每个请求，都会调用此方法。

    `process_request()` 应该：返回 `None` 或 `Response` 对象或 `Request` 对象，或者抛出 `IgnoreRequest`。

    如果它返回 `None`，Scrapy 将继续处理此请求，执行所有其他中间件，一直到 Downloader 把 `Request` 执行后得到 `Response` 才结束。

    如果它返回一个 `Response` 对象，Scrapy 将不会打扰任何其他低优先级 `process_request()` 或 `process_exception()` 方法，或适当的下载功能; 转而调用中间件的 `process_response()`，最后将 `Response` 交给 Spider 处理。

    如果它返回一个 `Request` 对象，更低优先级的下载中间件的 `process_request()` 方法会停止执行，这个 `Request` 会重新放到调度队列里等待被调度。

    如果它引发 `IgnoreRequest` 异常，所有下载中间件的 `process_exception()` 方法会依次执行，如果没有一个方法处理这个异常，那么 `Request` 的 `errorback()` 方法就会回调。如果还是没有被处理，那么会被忽略掉。


* `process_response(request, response, spider)`

    下载器执行 `Request` 下载后，会得到对应的 `Response`。Scrapy 引擎便会将 `Response` 发送给 Spider 进行解析。在发送之前，都可以用 `process_response()` 方法来对 `Response` 进行处理。

    `process_response()` 应该：返回一个 `Response` 对象，返回一个 `Request` 对象或引发一个 `IgnoreRequest` 异常。

    如果它返回一个 `Request` 对象，则暂停中间件链，并重新安排 `Request` 到队列中以便将来下载。

    如果它返回 `Response`，一切照旧。

    如果它引发 `IgnoreRequest` 异常，则 `Request` 的 `errorback()` 方法就会回调。如果还是没有被处理，那么会被忽略掉。

* `process_exception(request, exception, spider)`

    当下载器或 `process_request()` 方法抛出异常时，例如 `IgnoreRequest` 异常，该方法就会被调用。

    `process_exception()` 应该返回 `None` 或 `Response` 对象或 `Request` 对象。

    如果它返回 `None`，Scrapy 将继续处理此异常，执行优先级更低的 `process_exception()` 方法。

    如果它返回一个 `Response` 对象，`process_response()` 方法被依次调用，低优先级的 `process_exception()` 方法不在被调用。

    如果它返回一个 `Request` 对象，则暂停中间件链，并重新安排 `Request` 到队列中以便将来下载。

## 实例：随机 User-Agent

初始化一个新项目

```
scrapy startproject scrapydownloadertest
cd scrapydownloadertest/
scrapy genspider httpbin httpbin.org
```

修改 `httpbin.py`

``` python
# -*- coding: utf-8 -*-
import scrapy


class HttpbinSpider(scrapy.Spider):
    name = 'httpbin'
    allowed_domains = ['httpbin.org']
    start_urls = ['http://httpbin.org/get']

    def parse(self, response):
        self.logger.debug(response.text)

```

修改 User-Agent 有两种方式，在 settings 文件设置或者自定义下载中间件。

这里用自定义下载中间件演示随机 User-Agent

``` python
import random

class RandomUserAgentMiddleware(object):
    def __init__(self):
        self.user_agents = [
            'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36',
            'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.99 Safari/537.36',
            'Request/1.8'
        ]

    def process_request(self, request, spider):
        request.headers['User-Agent'] = random.choice(self.user_agents)
        return None
```

修改 settings 文件

``` python
DOWNLOADER_MIDDLEWARES = {
   'scrapydownloadertest.middlewares.RandomUserAgentMiddleware': 543,
}
```

完毕，查看结果

```
scrapy crawl httpbin
```