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

# Spider Middleware

作用

* 在 `Response` 从 Downloader 发送给 Spider 之前对其进行处理
* 在 `Request` 从 Spider 发送给 Scheduler 之前对其进行处理
* 在 `Item` 从 Spider 发送给 Item Pipeline 之前对其进行处理

## 使用说明

Scrapy 内置了许多爬虫中间件，都定义在 `SPIDER_MIDDLEWARES_BASE` 变量中

``` python
{
    'scrapy.spidermiddlewares.httperror.HttpErrorMiddleware': 50,
    'scrapy.spidermiddlewares.offsite.OffsiteMiddleware': 500,
    'scrapy.spidermiddlewares.referer.RefererMiddleware': 700,
    'scrapy.spidermiddlewares.urllength.UrlLengthMiddleware': 800,
    'scrapy.spidermiddlewares.depth.DepthMiddleware': 900,
}
```

自己写的中间件应该放在 `SPIDER_MIDDLEWARES` 中

``` python
SPIDER_MIDDLEWARES = {
    'myproject.middlewares.CustomSpiderMiddleware': 543,
}
```

##  核心方法

* `process_spider_input(response, spider)`

    当 `Response` 被中间件处理时调用

    应该返回 `None` 或者抛出异常

    如果返回 None，一切照旧

    如果抛出异常，终止调用链，而调用 `Request` 的 `errback()` 方法。`errback` 的输出将会被重新输入到中间件中，使用 `process_spider_output()` 方法来处理，当其抛出异常时则调用 `process_spider_exception()` 来处理。

* `process_spider_output(response, result, spider)`

    当 Spider 处理 `Response` 返回结果时被调用

    `result` 时包含 `Request` 或 `Item` 对象的可迭代对象，即 Spider 返回的结果

    必须返回包含 `Request` 或 `Item` 对象的可迭代对象

* `process_spider_exception(response, exception, spider)`

    当 Spider 或 Spider Middleware 的 `process_spider_input()` 方法抛出异常时被调用

    必须返回 `None` 或返回包含 `Response` 或 `Item` 对象的可迭代对象

    如果返回 `None`，Scrapy 将继续处理异常

    如果返回可迭代对象，则其他 Spider Middleware 的 `process_spider_output()` 方法被调用，其他的  `process_spider_exception()` 方法则不会被调用


* `process_start_requests(start_requests, spider)`

    该方法以 Spider 启动的 `Request` 为参数被调用，执行过程类似 `process_spider_output()` ，只不过它没有关联的 `Response`，且必须返回 `Request`。

    必须返回包含 `Request` 对象的可迭代对象

# Item Pipeline

当 Spider 解析完 Response 之后，Item 就会传递到 Item Pipeline，被定义的 Item Pipeline 组件会顺次调用，完成一连串的处理过程，如数据清洗，存储等。

主要功能：

* 清理 HTML 数据
* 验证爬取数据，检查爬取字段。
* 查重并丢弃重复字段
* 将爬取结果保存到数据库

## 核心方法

自定义 Item Pipeline 必须要实现 `process_item(self, item, spider)` 方法。还有几个比较实用的方法。

* `process_item(self, item, spider)`

    必须实现的方法，必须返回 Item 类型的值或者抛出 `DropItem` 异常

    如果返回 Item ，一切照旧，调用链继续

    如果抛出 `DropItem` 异常，此 Item 会被丢弃，不在处理

* `open_spider(self, spider)`

    Spider 开启的时候被自动调用

* `close_spider(self, spider)`

    Spider 关闭的时候自动调用

* `from_crawler(cls, crawler)`

    类方法，用于创建 Pipeline 实例，返回一个 Class 实例，时一种依赖注入方式，可以从 crawler 中拿到 Scrapy 的所有核心组件，如 settings。

## 实例：360图片抓取

### 创建项目

```
scrapy startproject images360
cd images360/
scrapy genspider images image.so.com
```

`images.py`

``` python
# -*- coding: utf-8 -*-
from urllib.parse import urlencode
import scrapy

# http://image.so.com/zj?ch=photography&sn=30&listtype=new&temp=1
class ImagesSpider(scrapy.Spider):
    name = 'images'
    allowed_domains = ['image.so.com']
    start_urls = ['http://image.so.com/']
    MAX_PAGE = 4

    def start_requests(self):
        data = {
            'ch': 'photography',
            'listtype': 'new',
            'temp': 1
        }
        base_url = 'http://image.so.com/zj?'
        for sn in range(30, self.MAX_PAGE*30, 30):
            data['sn'] = sn
            params = urlencode(data)
            url = base_url + params
            yield scrapy.Request(url, callback=self.parse)

    def parse(self, response):
        pass
```

`settings.py`

``` python
# Obey robots.txt rules
ROBOTSTXT_OBEY = False
```

测试是否可抓取

```
scrapy crawl images
```

### 信息提取

`items.py`

``` python
from scrapy import Item, Field

class ImageItem(Item):
    collection = 'images'
    id_ = Field()
    url = Field()
    title = Field()
    thumb = Field()
```

`images.py`

``` python
def parse(self, response):
    result = json.loads(response.text) 
    items = result.get('list')
    for item in items:
        img = ImageItem()
        img['id_'] = item.get('imageid')
        img['url'] = item.get('qhimg_url')
        img['title'] = item.get('group_title')
        img['thumb'] = item.get('qhimg_thumb_url')
        yield img
```

### 存储信息

#### MongoDB

`pipelines.py`

``` python
import pymongo

class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def process_item(self, item, spider):
        self.db[item.collection].insert_one(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

`settings.py`

``` python
MONGO_URI = 'localhost'
MONGO_DB = 'images360'

ITEM_PIPELINES = {
   'images360.pipelines.MongoPipeline': 300,
}
```

#### MySQL

`pipelines.py`

``` python
from images360.mysql_entity import Images, create_all
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

class MysqlPipeline(object):
    def __init__(self, host, database, user, password, port):
        self.host = host
        self.database = database
        self.user = user
        self.password = password
        self.port = port

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            host=crawler.settings.get('MYSQL_HOST'),
            database=crawler.settings.get('MYSQL_DATABASE'),
            user=crawler.settings.get('MYSQL_USER'),
            password=crawler.settings.get('MYSQL_PASSWORD'),
            port=crawler.settings.get('MYSQL_PORT')
        )

    def open_spider(self, spider):
        self.engine = create_engine('mysql+pymysql://{}:{}@{}:{}/{}'.format(
            self.user,
            self.password,
            self.host,
            self.port,
            self.database
        ))
        create_all(self.engine)
        self._Session = sessionmaker(bind=self.engine)
        # Session.configure(bind=engine) 
        self.session = self._Session()
        

    def process_item(self, item, spider):
        image = Images(
            idstr=item['id_'],
            url=item['url'],
            title=item['title'],
            thumb=item['thumb']
        )
        self.session.add(image)
        self.session.commit()
        return item

    def close_spider(self, spider):
        self.session.close()
```

`mysql_entity.py`

``` python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String


Base = declarative_base()

class Images(Base):
    __tablename__ = 'images'

    id_ = Column(Integer, primary_key=True)
    idstr = Column(String(50))
    url = Column(String(200))
    title = Column(String(200))
    thumb = Column(String(200))

def create_all(engine):
    Base.metadata.create_all(engine)
```

`settings.py`

``` python
MYSQL_HOST = 'localhost'
MYSQL_PORT = '3306'
MYSQL_USER = 'root'
MYSQL_PASSWORD = '123456'
MYSQL_DATABASE = 'images360'

ITEM_PIPELINES = {
   'images360.pipelines.MongoPipeline': 300,
   'images360.pipelines.MysqlPipeline': 301,
}
```

#### Image Pipeline

> [官方文档](https://doc.scrapy.org/en/latest/topics/media-pipeline.html)

Scrapy 提供了专门处理下载的 Pipeline，包括文件下载和图片下载。

在 `settings.py` 中配置存储路径

``` python
IMAGES_STORE = './images'
```

路径为项目下的 `images` 文件夹

内置的 `ImagesPipeline` 会默认读取 `Item` 的 `image_urls` 字段，并认为该字段是一个列表，它会遍历 `Item` 的 `image_urls` 字段，取出 URL 进行图片下载。

这里需要自定义 `ImagesPipeline`

`pipelines.py`

``` python
from scrapy import Request
from scrapy.exceptions import DropItem
from scrapy.pipelines.images import ImagesPipeline

class ImagePipeline(ImagesPipeline):
    def file_path(self, request, response=None, info=None):
        url = request.url
        file_name = url.split('/')[-1]
        return file_name

    def item_completed(self, results, item, info):
        image_paths = [x['path'] for ok, x in results if ok]
        if not image_paths:
            raise DropItem('Image Download Filed')
        return item

    def get_media_requests(self, item, info):
        yield Request(item['url'])
```

* get_media_requests

    返回包含图片 URL 的 Request 的可迭代对象

* file_path

    `request` 就是下载当前图片的 request，该方法用来返回需要保存的图片的文件名

* item_completed

    当单个 Item 下载完成时调用，`results` 就是下载结果，它时一个列表，每一项都是一个元组，包含了下载成功或失败的信息。这里的处理是如果下载失败就丢弃这个 Item。

`settings.py`

``` python
ITEM_PIPELINES = {
    'images360.pipelines.ImagePipeline': 300,
    'images360.pipelines.MongoPipeline': 301,
    'images360.pipelines.MysqlPipeline': 302,
}
```

这里把下载图片的优先级设置的最高，如果下载失败，那 item 就不用保存了

完成

```
scrapy crawl images
```

# Scrapy 对接 Selenium

以抓取淘宝为例

## 新建项目

```
scrapy startproject scrapyseleniumtest
cd scrapyseleniumtest/
scrapy genspider taobao www.taobao.com
```

修改 `settings.py` 文件

``` python
ROBOTSTXT_OBEY = False
```

定义 Item

``` python
from scrapy import Item, Field

class ProductItem(Item):
    collection = 'products'
    image = Field()
    price = Field()
    deal = Field()
    title = Field()
    shop = Field()
    location = Field()
```

`taobao.py`

``` python
import scrapy
from urllib.parse import quote
from scrapyseleniumtest.items import ProductItem

class TaobaoSpider(scrapy.Spider):
    name = 'taobao'
    allowed_domains = ['www.taobao.com']
    base_url = 'https://s.taobao.com/search?q={}'

    def start_requests(self):
        for keyword in self.settings.get('KEYWORDS'):
            for page in range(1, self.settings.get('MAX_PAGE') + 1):
                url = self.base_url.format(quote(keyword))
                yield scrapy.Request(url=url, callback=self.parse, meta={'page': page}, dont_filter=True)

```

`settings.py`

``` python
KEYWORDS = ['iPad']
MAX_PAGE = 10
```

由于每次请求的 url 相同，所以用 `meta` 参数来标记页码，同时设置 `dont_filter` 不去重。后面会用 selenium 跳转到指定页面。

## 对接 Selenium

采用 Downloader Middleware 来实现。 在中间件的 `process_request()` 方法里对每个请求进行处理，启动浏览器进行渲染，再将结果构造成 `HtmlResponse` 对象返回。

``` python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.support.wait import WebDriverWait
from selenium.common.exceptions import TimeoutException
from scrapy.http import HtmlResponse
from logging import getLogger


class SeleniumMiddleware(object):
    def __init__(self):
        self.logger = getLogger(__name__)
        self.timeout = 20
        chrome_options = webdriver.ChromeOptions()
        chrome_options.add_argument('--headless')
        self.browser = webdriver.Chrome(chrome_options=chrome_options)
        self.browser.set_window_size(1400, 700)
        self.browser.set_page_load_timeout(self.timeout)
        self.wait = WebDriverWait(self.browser, self.timeout)

    def __del__(self):
        self.browser.close()

    def process_request(self, request, spider):
        self.logger.debug('Chrome is Starting')
        page = request.meta.get('page', 1)
        try:
            self.browser.get(request.url)
            if page > 1:
                input_ = self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#mainsrp-pager div.form .input')))
                button = self.wait.until(EC.element_to_be_clickable((By.CSS_SELECTOR, '#mainsrp-pager div.form .btn')))
                input_.clear()
                input_.send_keys(page)
                button.click()
                ############## 页面跳转完成
            # 等待页面加载
            self.wait.until(EC.text_to_be_present_in_element((By.CSS_SELECTOR, '#mainsrp-pager .item.active > span'), str(page)))
            self.wait.until(EC.presence_of_element_located((By.CSS_SELECTOR, '#mainsrp-itemlist .items .item')))
            # 返回 HtmlReponse
            return HtmlResponse(url=request.url, body=self.browser.page_source, request=request,
                                encoding='utf-8', status=200)
        except TimeoutException:
            return HtmlResponse(url=request.url, status=500, request=request)
```

`settings.py`

``` python
DOWNLOADER_MIDDLEWARES = {
   'scrapyseleniumtest.middlewares.SeleniumMiddleware': 543,
}
```

## 解析页面

``` python
def parse(self, response):
    products = response.css('#mainsrp-itemlist .items .item')
    for product in products:
        item = ProductItem()
        item['image'] = product.css('.J_ItemPic.img::attr(data-src)').extract_first()
        item['price'] = product.css('.price strong::text').extract_first()
        item['deal'] = product.css('.deal-cnt::text').extract_first()
        item['title'] = ''.join(product.xpath('.//div[contains(@class, "title")]//text()').extract()).strip()
        item['shop'] = ''.join(product.xpath('.//div[contains(@class, "shop")]//text()').extract()).strip()
        item['location'] = product.css('.location::text').extract_first()
        yield item
```

## 存储结果

``` python
import pymongo

class MongoPipeline(object):
    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DB')
        )

    def open_spider(self, spider):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def process_item(self, item, spider):
        self.db[item.collection].insert_one(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

`settings.py`

``` python
ITEM_PIPELINES = {
   'scrapyseleniumtest.pipelines.MongoPipeline': 300,
}

MONGO_URI = 'localhost'
MONGO_DB = 'taobao'
```

完成

```
scrapy crawl taobao
```