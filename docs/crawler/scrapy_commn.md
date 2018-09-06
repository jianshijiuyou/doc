# CrawlSpider

`CrawlSpider` 继承自 `Spider` 类，除了 `Spider` 类的所有属性和方法外，还有一个非常重要的属性和方法。

* `rules`:

    它是爬取规则属性，是包含一个或多个 `Rule` 对象的列表。每个 `Rule` 对爬取网站的动作都做了定义，CrawlSpider 会读取 `rules` 的每一个 `rule` 并进行解析。

* `parse_start_url()`:

    它是一个可重写的方法。当 `start_urls` 里对应的 `Request` 得到 `Resposne` 时被调用，它会分析 `Response` 并返回 `Item` 对象或者 `Request` 对象。

* `Rule` 的定义和参数如下

    `class scrapy.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)`

    * link_extractor

        通过它，Spider 可以知道从爬取的页面中提取哪些链接。提取出来的链接会自动生成 `Request`，常用 `LxmlLinkExtractor` 对象作为参数：

        `class scrapy.linkextractors.lxmlhtml.LxmlLinkExtractor(allow=(), deny=(), allow_domains=(), deny_domains=(), deny_extensions=None, restrict_xpaths=(), restrict_css=(), tags=('a', 'area'), attrs=('href', ), canonicalize=False, unique=True, process_value=None, strip=True)`

        * `allow`: 正则表达式或正则表达式列表，定义了从当前页面提取出的链接哪些是符合要求的。
        * `deny`: 和 `allow` 相反。
        * `allow_domains`: 定义符合要求的域名
        * `deny_domains`: 和 `allow_domains` 相反
        * `deny_extensions`: 定义要忽略的扩展名，默认为 [`scrapy.linkextractors`](https://github.com/scrapy/scrapy/blob/master/scrapy/linkextractors/__init__.py) 包中定义的`IGNORED_EXTENSIONS` 列表。
        * `restrict_xpaths`: 限定范围，从 xpath 匹配的区域提取链接
        * `restrict_css`: 限定范围，从 css 选择器匹配的区域提取链接
        * `tags`: 提取链接时要考虑的标签或标签列表。默认为 `('a', 'area')`。
        * `attrs`: 查找要提取的链接时应考虑的属性或属性列表（仅适用于 `tags` 参数中指定的那些标签）。默认为 `('href', )`
        * `canonicalize`: 规范化每个提取的 URL（使用 w3lib.url.canonicalize_url）。默认为 `False`，建议不要修改。
        * `unique`: 是否应对提取的链接应用重复过滤。
        * `process_value`: 接收从标签中提取的每个值和扫描的属性的函数，可以修改该值并返回一个值，或者返回 `None` 以完全忽略该链接。如果没有给出，`process_value` 默认为 `lambda x:x`。 例如，要从此代码中提取链接：

        ``` html
        <a href="javascript:goToPage('../other/page.html'); return false">Link text</a>
        ```

        您可以在 `process_value` 中使用以下函数：

        ``` python
        def process_value(value):
            m = re.search("javascript:goToPage\('(.*?)'", value)
            if m:
                return m.group(1)
        ```
        * `strip`: 是否从提取的属性中去除空格。

    * callback

    是一个可调用对象的或一个字符串（在这种情况下，将使用具有该名称的 spider 对象的方法）为使用指定的 `link_extractor` 提取的每个链接调用。此回调接收 `response` 作为其第一个参数，并且必须返回包含 `Item` 和/或 `Request` 对象（或其任何子类）的列表。

    !> 在编写 crawl spider rules 时，请避免使用 `parse` 作为回调，因为 `CrawlSpider` 使用 `parse` 方法本身来实现其逻辑。因此，如果您覆盖解析方法，则 crawl spider 将不再起作用。

    * cb_kwargs 

    是一个包含要传递给回调函数的关键字参数的 `dict`。

    * follow

    是一个布尔值，它指定是否应该从使用此规则提取的每个响应中跟踪链接。如果 `callback` 为 `None`，则默认为 `True`，否则默认为 `False`。

    * process_links

    是一个可调用的对象，或一个字符串（在这种情况下，将使用来自具有该名称的 spider 对象的方法），从 `link_extractor` 中获取链接列表时调用该方法，主要用于过滤。

    * process_request 

    是一个可调用的对象，或一个字符串（在这种情况下，将使用来自具有该名称的 spider 对象的方法），该方法将在此规则提取的每个 `request` 中调用，并且必须返回 `Request` 或 `None`（以过滤掉请求） 。

> [详细信息](https://doc.scrapy.org/en/latest/topics/spiders.html#crawling-rules)

# Item Loader

`class scrapy.loader.ItemLoader([item, selector, response, ]**kwargs)`

[详细信息](https://doc.scrapy.org/en/latest/topics/loaders.html)

典型实例如下：

``` python
from scrapy.loader import ItemLoader
from myproject.items import Product

def parse(self, response):
    l = ItemLoader(item=Product(), response=response)
    l.add_xpath('name', '//div[@class="product_name"]')
    l.add_xpath('name', '//div[@class="product_title"]')
    l.add_xpath('price', '//p[@id="price"]')
    l.add_css('stock', 'p#stock]')
    l.add_value('last_updated', 'today') # you can also use literal values
    return l.load_item()
```

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

构造爬取详情页和下一页的 Rule

``` python
rules = (
    Rule(LinkExtractor(allow=r'article/.*\.html', restrict_css='#left_side .con_item'), callback='parse_item'),
    Rule(LinkExtractor(restrict_xpaths='//div[@id="pageStyle"]/a[contains(.,"下一页")]'))
)
```

## 限制请求

现在只要满足规则就会一直抓取，如果只是做测试，比如只想抓取前两页，可以做个限制

``` python
rules = (
    Rule(LinkExtractor(allow=r'article/.*\.html', restrict_css='#left_side .con_item'),
            callback='parse_item'),
    Rule(LinkExtractor(restrict_xpaths='//div[@id="pageStyle"]/a[contains(.,"下一页")]'),
            process_request='limit_pages'),
)

def limit_pages(self, request):
    if request.url.endswith('index_3.html'):
        return None
    else:
        return request
```

## 解析页面

首先时 itme 和 item loader

``` python
from scrapy import Item, Field
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, Join, Compose


class NewsItem(Item):
    title = Field()
    url = Field()
    text = Field()
    datetime = Field()
    source = Field()
    website = Field()


class ChinaLoader(ItemLoader):
    default_output_processor = TakeFirst()
    text_out = Compose(Join(), str.strip)
    source_out = Compose(Join(), str.strip)
```

然后开始解析

``` python
def parse_item(self, response):
    loader = ChinaLoader(item=NewsItem(), response=response)
    loader.add_css('title', '#chan_newsTitle::text')
    loader.add_value('url', response.url)
    loader.add_xpath('text', '//div[@id="chan_newsDetail"]//text()')
    loader.add_css('datetime', '#chan_newsInfo::text', re=r'(\d+-\d+-\d+\s\d+:\d+:\d+)')
    loader.add_css('source', '#chan_newsInfo::text', re='来源：(.*)')
    loader.add_value('website', '中华网')
    yield loader.load_item()
```

!> css 选择器有时候可能会出问题，如果发现和预期不符合建议使用 xpath