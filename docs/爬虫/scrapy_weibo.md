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

