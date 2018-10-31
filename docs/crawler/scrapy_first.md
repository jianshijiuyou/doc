# 介绍

![](https://pikachu666.oss-cn-hongkong.aliyuncs.com/images/8c591d54457bb033812a2b0364011e9c.png)

> 图片来自网络

# 基本用法

## 创建项目

`scrapy startproject <project name>`

例如

``` python
scrapy startproject tutorial
```

结构如下

```
tutorial
├── scrapy.cfg      # 部署配置文件
└── tutorial        # 项目的模块，需要从这里引入
    ├── __init__.py
    ├── items.py            # 爬取的数据结构
    ├── middlewares.py      # 爬取的中间件
    ├── pipelines.py        # 数据管道
    ├── __pycache__         
    ├── settings.py         # 项目配置文件
    └── spiders             # Spiders 文件夹
        ├── __init__.py
        └── __pycache__
```

## 创建 Spider

`scrapy genspider <spider name> <url>`

例如

```
scrapy genspider quotes quotes.toscrape.com
```

!> 这个命令需要在自己的项目文件夹中执行

```
tutorial/spiders/
├── __init__.py
├── __pycache__
└── quotes.py
```

`quotes.py` 文件

``` python
# -*- coding: utf-8 -*-
import scrapy


class QuotesSpider(scrapy.Spider):
    name = 'quotes'
    allowed_domains = ['quotes.toscrape.com']
    start_urls = ['http://quotes.toscrape.com/']

    def parse(self, response):
        pass
```

* `name`: 项目唯一的名字
* `allowed_domains`: 允许爬取的域名
* `start_urls`: Spider 启动时爬取的 url 列表，初始请求
* `parse`: `start_urls` 中的链接请求完的结果交给它处理

## 创建 Item

``` python
import scrapy


class QuoteItem(scrapy.Item):
    text = scrapy.Field()
    author = scrapy.Field()
    tags = scrapy.Field()
```

## 解析 Response

``` python
def parse(self, response):
    quotes = response.css('.quote')
    for quote in quotes:
        text = quote.css('.text::text').extract_first()
        author = quote.css('.author::text').extract_first()
        tags = quote.css('.tags .tag::text').extract()
```

## 使用 Item

``` python
from tutorial.items import QuoteItem

def parse(self, response):
    quotes = response.css('.quote')
    for quote in quotes:
        item = QuoteItem()
        item['text'] = quote.css('.text::text').extract_first()
        item['author'] = quote.css('.author::text').extract_first()
        item['tags'] = quote.css('.tags .tag::text').extract()
        yield item
```

##  后续 Request

``` python
def parse(self, response):
    quotes = response.css('.quote')
    for quote in quotes:
        item = QuoteItem()
        item['text'] = quote.css('.text::text').extract_first()
        item['author'] = quote.css('.author::text').extract_first()
        item['tags'] = quote.css('.tags .tag::text').extract()
        yield item
    
    next_ = response.css('.pager .next a::attr(href)').extract_first()
    url = response.urljoin(next_)
    yield scrapy.Request(url=url, callback=self.parse)
```

## 运行

`scrapy crawl <Spider name>`

例如

```
scrapy crawl quotes
```

## 保存到文件

将结果直接保存成 JSON 格式的文件

```
scrapy crawl quotes -o quotes.json
```

每个 Item 一行

```
scrapy crawl quotes -o quotes.j1
# or
scrapy crawl quotes -o quotes.jsonlines
```

其他格式输出

```
scrapy crawl quotes -o quotes.csv
scrapy crawl quotes -o quotes.xml
scrapy crawl quotes -o quotes.pickle
scrapy crawl quotes -o quotes.marshal
scrapy crawl quotes -o ftp://user:pass@ftp.exmple.com/path/to/quotes.csv
```

## 使用 Item Pipeline

它的作用

* 清理 HTML 数据
* 验证爬取数据，检查爬取字段
* 查重
* 保存到数据库


举个栗子，处理长度大于 50 的 text

``` python
from scrapy.exceptions import DropItem

class TextPipeline(object):
    def __init__(self):
        self.limit = 50
    
    def process_item(self, item, spider):
        if item['text']:
            if len(item['text']) > self.limit:
                item['text'] = item['text'][0:self.limit].rstrip() + '...'
            return item
        else:
            return DropItem('Missing Text')
```

将结果保存到数据库

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
        name = item.__class__.__name__
        self.db[name].insert_one(dict(item))
        return item

    def close_spider(self, spider):
        self.client.close()
```

* `from_crawler`: 程序会先调用这个类方法来创建类，通过它可以获取 settings 中的参数
* `open_spider`: Spider 开启时调用
* `close_spider`: Spider 关闭时调用

配置 `settings.py`

``` python
ITEM_PIPELINES = {
   'tutorial.pipelines.TextPipeline': 300, # 优先级，越小越高
   'tutorial.pipelines.MongoPipeline': 400,
}

MONGO_URI = 'localhost'
MONGO_DB = 'tutorial'
```

# Scrapy shell

测试地址

```
scrapy shell http://doc.scrapy.org/en/latest/_static/selectors-sample1.html
```