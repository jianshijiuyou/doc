# CrawlSpider

`CrawlSpider` 继承自 `Spider` 类，除了 `Spider` 类的所有属性和方法外，还有一个非常重要的属性和方法。

* `rules`:

    它是爬取规则属性，是包含一个或多个 `Rule` 对象的列表。每个 `Rule` 对爬取网站的动作都做了定义，CrawlSpider 会读取 `rules` 的每一个 `rule` 并进行解析。

* `parse_start_url()`:

    它是一个可重写的方法。当 `start_urls` 里对应的 `Request` 得到 `Resposne` 时被调用，它会分析 `Response` 并返回 `Item` 对象或者 `Request` 对象。

* `Rule` 的定义和参数如下

    `class scrapy.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)`

    * link_extractor

        111
    * callback
    * cb_kwargs 
    * follow
    * process_links
    * process_request 

> [详细信息](https://doc.scrapy.org/en/latest/topics/spiders.html#crawling-rules)

# Item Loader

`class scrapy.loader.ItemLoader([item, selector, response, ]**kwargs)`

[详细信息](https://doc.scrapy.org/en/latest/topics/loaders.html)

Item Loader 每个字段中都包含一个 Input Processor 和 一个 Output Processor

常见内置 Processor

* Identity

    不处理，原样输出

* TakeFirst

    返回列表的第一个非空值

    ``` python
    from scrapy.loader.processors import TakeFirst
    processor = TakeFirst()
    print(processor(['', 1, 2]))
    # 1
    ```

* Join

    默认用空格拼接字符串

    ``` python
    from scrapy.loader.processors import Join
    processor = Join()
    print(processor(['one', 'two', 'three']))
    # one two three
    processor = Join(',')
    print(processor(['one', 'two', 'three']))
    # one,two,three
    ```

* Compose

    用给定的多个函数的组合而构造的 Processor，每个输入值被传递到第一个函数，其输出值再传递到第二个函数...

    ``` python
    from scrapy.loader.processors import Compose
    processor = Compose(str.upper, lambda s: s.strip())
    print(processor(" hello world"))
    # HELLO WORLD
    ```

* MapCompose

    和 Compose 类似，MapCompose 可以迭代处理一个列表输入值

    ``` python
    from scrapy.loader.processors import MapCompose
    processor = MapCompose(str.upper, lambda s: s.strip())
    print(processor([' hello', 'world', 'Python']))
    # ['HELLO', 'WORLD', 'PYTHON']
    ```

* SelectJmes

    查询 JSON，传入 key，返回查询所得的 Value，需要安装 jmespath 库。

    ``` python
    pipenv install jmespath
    ```

    用法

    ``` python
    from scrapy.loader.processors import SelectJmes
    processor = SelectJmes('foo')
    print(processor({'foo': 'bar'}))
    # bar
    ```

# 实例：抓取中华网科技类新闻

[https://tech.china.com/articles/](https://tech.china.com/articles/)

## 新建项目

```
scrapy startproject scrapyuniversal
```

查看可用模板

```
$ scrapy genspider -l
Available templates:
  basic
  crawl
  csvfeed
  xmlfeed
```

默认是用的 basic 模板

这次用 crawl 模板

```
scrapy genspider -t crawl china tech.china.com
```

`china.py`

``` python
# -*- coding: utf-8 -*-
import scrapy
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule


class ChinaSpider(CrawlSpider):
    name = 'china'
    allowed_domains = ['tech.china.com']
    start_urls = ['http://tech.china.com/']

    rules = (
        Rule(LinkExtractor(allow=r'Items/'), callback='parse_item', follow=True),
    )

    def parse_item(self, response):
        i = {}
        #i['domain_id'] = response.xpath('//input[@id="sid"]/@value').extract()
        #i['name'] = response.xpath('//div[@id="name"]').extract()
        #i['description'] = response.xpath('//div[@id="description"]').extract()
        return i

```

## 定义 Rule

首先修改起始链接

``` python
start_urls = ['https://tech.china.com/articles/']
```

构造 Rule

``` python
rules = (
    Rule(LinkExtractor(allow=r'articles/.*\.html'), restrict_css='#left_side .con_item' callback='parse_item'),
)
```