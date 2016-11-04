---
title: 使用python3的scrapy来写落网爬虫(一)
date: '2016/3/27 20:00:00'
tags:
  - scrapy
---

`Scrapy` 是一个 `python` 爬虫框架。又因为我是个 `python3` 的死忠， 所以一直听说，却没用过。现在好了，它的测试版已经支持 `python3`， 终于可以体验下了传说中的东西。<br>
用它写个落网爬虫-- [luoospider](https://github.com/dust8/luoospider), 可以 `Star` 支持下.

# 官网

```
http://scrapy.org/
```

# 安装Scrapy

```
pip install scrapy==1.1.0rc3
```

# 项目分析

## 1\. 目的：抓取并下载落网的音乐

## 2\. 分析落网的页面结构

```
  * 期刊入口: `http://www.luoo.net/music/`
  * 每一页的期刊列表在`<div class="vol-list">`
  * 下一页在 `<a class="next">`
  * 期刊刊号在 `<span class="vol-number">`
  * 期刊标题在 `<span class="vol-title>"`
  * 每期期刊列表在 `<div class="vol-tracklist">` 里面，有每首歌的详细内容
  * 一首歌的url为 `http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio805/01.mp3`, `radio805` 是第805期期刊，`01.mp3` 是这期期刊的第一首歌曲。
```

# 创建项目

```
scrapy startproject luoo
```

进入luoo目录后显示的文件如下：

```
├── luoo
│   ├── __init__.py
│   ├── items.py
│   ├── pipelines.py
│   ├── settings.py
│   └── spiders
│       ├── __init__.py
│       └── luoo_spider.py
└── scrapy.cfg
```

其中 `luoo_spider.py` 是我们自己要新建的文件.

# 代码

`items.py`

```
class LuooItem(scrapy.Item):
    vol_number = scrapy.Field()
    vol_title = scrapy.Field()
    trackname = scrapy.Field()
    artist = scrapy.Field()
```

`luoo_spider.py`

```
class LuooSpider(Spider):
    name = 'luoo'
    allowed_domains = ['luoo.net']
    start_urls = ['http://www.luoo.net/music/']

    def __init__(self, last_vol_number=0, *args, **kwargs):
        super(LuooSpider, self).__init__(*args, **kwargs)
        # 上次抓取的最大期刊期号
        self.last_vol_number = last_vol_number
        self.stop = False

    def parse(self, response):
        # 抓取一页里面的每期期刊地址
        for href in response.css('div.vol-list>div.item>a::attr("href")'):
            url = href.extract()
            if url.split('/')[-1] <= self.last_vol_number:
                self.stop = True
                break
            yield Request(url, callback=self.parse_vol_contents)

        # 抓取下一页地址
        if not self.stop:
            for href in response.css('a.next::attr("href")'):
                url = href.extract()
                yield Request(url, callback=self.parse)

    def parse_vol_contents(self, response):
        '''抓取一期期刊里面的内容
        '''
        vol_number = response.css('span.vol-number::text').extract_first()
        vol_title = response.css('span.vol-title::text').extract_first()

        for sel in response.xpath('//div[contains(@class, "vol-tracklist")]/ul/li'):
            item = LuooItem()
            item['vol_number'] = vol_number
            item['vol_title'] = vol_title
            item['trackname'] = sel.css(
                'div.track-wrapper>a.trackname::text').extract_first()
            item['artist'] = sel.css(
                'div.track-wrapper>span.artist::text').extract_first()
            yield item
```

`pipelines.py`

```
class MongoPipeline:
    collection_name = 'music'

    def __init__(self, mongo_uri, mongo_db):
        self.mongo_uri = mongo_uri
        self.mongo_db = mongo_db

    @classmethod
    def from_crawler(cls, crawler):
        return cls(
            mongo_uri=crawler.settings.get('MONGO_URI'),
            mongo_db=crawler.settings.get('MONGO_DATABASE', 'luoo')
        )

    def open_spider(self, sipder):
        self.client = pymongo.MongoClient(self.mongo_uri)
        self.db = self.client[self.mongo_db]

    def close_spider(self, spider):
        self.client.close()

    def process_item(self, item, spider):
        self.db[self.collection_name].insert(dict(item))
        return item
```

`luoo` 爬虫支持 `last_vol_number` 参数，它指定了上次爬取的最大期刊刊号，可以避免重复 抓取以前抓取过的期刊。

# 运行爬虫

```
scrapy crawl luoo
```

或者

```
scrapy crawl luoo -a last_vol-number=800
```
