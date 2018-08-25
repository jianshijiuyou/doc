# 爬取分析

通过 `m.weibo.cn` 找到一个大 V 分析 Ajax 请求可以发现：

用户详情 API

`https://m.weibo.cn/api/container/getIndex?uid={uid}&type=uid&value={uid}&containerid=100505{uid}`

关注列表 API

`https://m.weibo.cn/api/container/getIndex?containerid=231051_-_followers_-_{uid}&type=uid&value={uid}&page={page}`

粉丝列表 API

`https://m.weibo.cn/api/container/getIndex?containerid=231051_-_fans_-_{uid}&type=uid&value={uid}&since_id={page}`

微博列表 API

`https://m.weibo.cn/api/container/getIndex?uid={uid}&type=uid&value={uid}&containerid=107603{uid}&page={page}`

# 新建项目

```
scrapy startproject weibo
cd weibo/
scrapy genspider weibocn m.weibo.cn
```

根据一个或多个大 V 构建起始 url

``` python
import scrapy


class WeibocnSpider(scrapy.Spider):
    name = 'weibocn'
    allowed_domains = ['m.weibo.cn']
    user_url = 'https://m.weibo.cn/api/container/getIndex?uid={uid}&type=uid&value={uid}&containerid=100505{uid}'
    follow_url = 'https://m.weibo.cn/api/container/getIndex?containerid=231051_-_followers_-_{uid}&type=uid&value={uid}&page={page}'
    fan_url = 'https://m.weibo.cn/api/container/getIndex?containerid=231051_-_fans_-_{uid}&type=uid&value={uid}&since_id={page}'
    weibo_url = 'https://m.weibo.cn/api/container/getIndex?uid={uid}&type=uid&value={uid}&containerid=107603{uid}&page={page}'
    start_users = ['2214257545'] # 大 V 的 uid 列表

    def start_requests(self):
        for uid in self.start_users:
            yield scrapy.Request(url=self.user_url.format(uid=uid),  callback=self.parse_user)

    def parse_user(self, response):
        self.logger.debug(response)
```

# 创建 Item

``` python
from scrapy import Item, Field


class UserItem(Item):
    collection = 'users'
    id = Field()
    name = Field()
    avatar = Field()
    cover = Field()
    gender = Field()
    description = Field()
    fans_count = Field()
    follows_count = Field()
    weibos_count = Field()
    verified = Field()
    verified_reason = Field()
    verified_type = Field()
    follows = Field()
    fans = Field()
    crawled_at = Field()


class UserRelationItem(Item):
    collection = 'users'
    id = Field()
    follows = Field()
    fans = Field()


class WeiboItem(Item):
    collection = 'weibos'
    id = Field()
    attitudes_count = Field()
    comments_count = Field()
    reposts_count = Field()
    picture = Field()
    pictures = Field()
    source = Field()
    text = Field()
    raw_text = Field()
    thumbnail = Field()
    user = Field()
    created_at = Field()
    crawled_at = Field()
```

# 提取数据

解析用户信息

``` python
    def parse_user(self, response):
        result = json.loads(response.text)
        user_info = result.get('data').get('userInfo')
        user_item = UserItem()
        if user_info:
            field_map = {
                'id': 'id', 'name': 'screen_name', 'avatar': 'profile_image_url', 'cover': 'cover_image_phone',
                'gender': 'gender', 'description': 'description', 'fans_count': 'followers_count',
                'follows_count': 'follow_count', 'weibos_count': 'statuses_count', 'verified': 'verified',
                'verified_reason': 'verified_reason', 'verified_type': 'verified_type'
            }
            for field, attr in field_map.items():
                user_item[field] = user_info.get(attr)
            yield user_item
            # 关注
            uid = user_info.get('id')
            yield scrapy.Request(url=self.follow_url.format(uid=uid, page=1), callback=self.parse_follows,
                                meta={'page': 1, 'uid': uid})
            # 粉丝
            yield scrapy.Request(url=self.fan_url.format(uid=uid, page=1), callback=self.parse_fans,
                                meta={'page': 1, 'uid': uid})
            # 微薄
            yield scrapy.Request(url=self.weibo_url.format(uid=uid, page=1), callback=self.parse_weibos,
                                meta={'page': 1, 'uid': uid})
```

解析用户关注（粉丝同理）

``` python
    def parse_follows(self, response):
        result = json.loads(response.text)

        # 解析用户
        try:
            follows = result.get('data').get('cards')[0].get('card_group')
        except Exception as e:
            self.logger.error(e)
        
        for follow in follows:
            uid = follow.get('user').get('id')
            yield scrapy.Request(url=self.user_url.format(uid=uid),  callback=self.parse_user)

        # 关注列表
        uid = response.meta.get('uid')
        user_relation_item = UserRelationItem()
        follows = [{'id': follow.get('user').get('id'), 'name': follow.get('user').get('screen_name')} for follow in follows]
        user_relation_item['id'] = uid
        user_relation_item['follows'] = follows
        user_relation_item['fans'] = []
        yield user_relation_item

        # 下一页
        page = response.meta.get('page') + 1
        yield scrapy.Request(url=self.follow_url.format(uid=uid, page=page), callback=self.parse_follows,
                                meta={'page': page, 'uid': uid})
```


解析微博

``` python
    def parse_weibos(self, response):
        result = json.loads(response.text)

        try:
            weibos = result.get('data').get('cards')
        except Exception as e:
            self.logger.error(e)
        
        for weibo in weibos:
            mblog = weibo.get('mblog')
            if mblog:
                weibo_item = WeiboItem()
                field_map = {
                    'id': 'id', 'attitudes_count': 'attitudes_count', 'comments_count': 'comments_count',
                    'created_at': 'created_at', 'reposts_count': 'reposts_count', 'source': 'source',
                    'text': 'text'
                }
                for field, attr in field_map.items():
                    weibo_item[field] = mblog.get(attr)
                    weibo_item['user'] = response.meta.get('uid')
                    yield weibo_item
        
        
        # 下一页
        uid = response.meta.get('uid')
        page = response.meta.get('page') + 1
        yield scrapy.Request(url=self.weibo_url.format(uid=uid, page=page), callback=self.parse_weibos,
                                meta={'page': page, 'uid': uid})
```

# 数据清洗

有的时间显示的是刚刚、几分钟前、几小时前等等。需要进行转化

``` python
import time
import re

from weibo.items import WeiboItem, UserItem, UserRelationItem

class WeiboPipeline(object):
    def process_item(self, item, spider):
        if isinstance(item, WeiboItem):
            if item.get('created_at'):
                item['created_at'] = item['created_at'].strip()
                item['created_at'] = self.parse_time(item.get('created_at'))
        return item


    def parse_time(self, date):
        if re.match('刚刚', date):
            date = time.strftime('%Y-%m-%d %H:%M', time.localtime())
        if re.match(r'\d+分钟前', date):
            minute = re.match(r'(\d+)', date).group(1)
            date = time.strftime('%Y-%m-%d %H:%M', time.localtime(time.time() - float(minute) * 60))
        if re.match(r'\d+小时前', date):
            hour = re.match(r'(\d+)', date).group(1)
            date = time.strftime('%Y-%m-%d %H:%M', time.localtime(time.time() - float(hour) * 60 * 60))
        if re.match(r'昨天.*', date):
            date = re.match('昨天(.*)', date).group(1).strip()
            date = time.strftime('%Y-%m-%d',time.localtime(time.time() - 24 * 60 * 60)) + ' ' + date
        if re.match(r'\d{2}-\d{2}', date):
            date = time.strftime('%Y-', time.localtime()) + date + ' 00:00'
        return date


class TimePipeline(object):
    def process_item(self, item, spider):
        if isinstance(item, UserItem) or isinstance(item, WeiboItem):
            now = time.strftime('%Y-%m-%d %H:%M', time.localtime())
            item['crawled_at'] = now # 抓取的时间
        return item
```

# 数据存储

``` python
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
        self.db[UserItem.collection].create_index([('id', pymongo.ASCENDING)])
        self.db[WeiboItem.collection].create_index([('id', pymongo.ASCENDING)])

    def process_item(self, item, spider):
        if isinstance(item, UserItem) or isinstance(item, WeiboItem):
            self.db[item.collection].update({'id': item.get('id')}, {'$set': dict(item)}, True)
        if isinstance(item, UserRelationItem):
            self.db[item.collection].update(
                {'id', item.get('id')},
                {
                    '$addToSet':{
                        'follows': {'$each': item['follows']},
                        'fans': {'$each': item['fans']}
                    }
                },
                True
            )
        return item
    
    def close_spider(self, spider):
        self.client.close()
```

# Cookies 池对接

CookiesPool 地址 https://github.com/Python3WebSpider/CookiesPool

配置好后在 `http://0.0.0.0:5000/weibo/random` 就可以获得随机的 cookie 了

编写中间件

``` python
import logging
import requests
import json

class CookiesMiddleware:
    def __init__(self, cookies_url):
        self.logger = logging.getLogger(__name__)
        self.cookies_url = cookies_url
    
    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(cookies_url=settings.get('COOKIES_URL'))

    def get_random_cookies(self):
        try:
            response = requests.get(self.cookies_url)
            if response.status_code == 200:
                cookies = json.loads(response.text)
                return cookies
        except requests.ConnectionError:
            return False

    def process_request(self, request, spider):
        self.logger.debug('正在获取 Cookies')
        cookies = self.get_random_cookies()
        if cookies:
            request.cookies = cookies
            self.logger.debug('使用 Cookies ' + json.dumps(cookies))
        
        return None
```

# 代理池对接

代理池使用项目 https://github.com/jhao104/proxy_pool

编写中间件

``` python
class ProxyMiddleware:
    def __init__(self, proxy_url):
        self.logger = logging.getLogger(__name__)
        self.proxy_url = proxy_url
    
    @classmethod
    def from_crawler(cls, crawler):
        settings = crawler.settings
        return cls(proxy_url=settings.get('PROXY_URL'))

    def get_random_proxy(self):
        try:
            response = requests.get(self.proxy_url)
            if response.status_code == 200:
                proxy = json.loads(response.text)
                return proxy
        except requests.ConnectionError:
            return False

    def process_request(self, request, spider):
        if request.meta.get('retry_times'):
            proxy = self.get_random_proxy()
            if proxy:
                url = 'https://{}'.format(proxy)
                self.logger.debug('使用代理 ' + proxy)
                request.meta['proxy'] = url
```

# 设置各种配置

``` python
ROBOTSTXT_OBEY = False

# 注意：要在内置的相关中间件之前调用
DOWNLOADER_MIDDLEWARES = {
   'weibo.middlewares.CookiesMiddleware': 554,
   'weibo.middlewares.ProxyMiddleware': 555,
}

ITEM_PIPELINES = {
    'weibo.pipelines.TimePipeline': 300,
    'weibo.pipelines.WeiboPipeline': 301,
    'weibo.pipelines.MongoPipeline': 302,
}

COOKIES_URL = 'http://0.0.0.0:5000/weibo/random'
PROXY_URL = 'http://127.0.0.1:5010/get'

MONGO_URI = 'localhost'
MONGO_DB = 'weibo'

DEFAULT_REQUEST_HEADERS = {
    'X-Requested-With': 'XMLHttpRequest',
    'Host': 'm.weibo.cn'
}

USER_AGENT = 'Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1'

```

# 限速

`DOWNLOAD_DELAY`：同一网站下载连续页面之前应等待的时间（以秒为单位）

比如每 500 毫秒发起一个请求

``` python
DOWNLOAD_DELAY = 0.5
```

此设置也受 `RANDOMIZE_DOWNLOAD_DELAY` 设置（默认情况下启用）的影响。默认情况下，Scrapy 不会在请求之间等待一段固定的时间，而是使用 `0.5 * DOWNLOAD_DELAY` 和 `1.5 * DOWNLOAD_DELAY` 之间的随机时间间隔。

# 暂停和恢复抓取

https://doc.scrapy.org/en/latest/topics/jobs.html