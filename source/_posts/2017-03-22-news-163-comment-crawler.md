---
title: 网易新闻评论爬虫
date: 2017-03-22 05:15:31
tags:
 - 爬虫
---

## 前言
昨天有 * boss * 给我出了一道爬虫题，让我爬网易新闻的评论，我说试下。

## 分析
### 一级入口
网易新闻的入口：* news.163.com *

### 二级人口：
![](/assert/2017-03-22-1.png)
点击进去查看源代码，发现没有内容，可以知道它是 ** 动态加载 ** 的。
通过 * Network * 里面的 * XHR *，并没有发现请求，查看 ** JS **
发现一个可疑请求   
![](/assert/2017-03-22-2.png)
就是它了 ** http://temp.163.com/special/00804KVA/cm_guonei.js?callback=data_callback **
通过它就可以得到 ** guonei ** 的最近新闻内容）。

> 注意这里 ** cm_guonei ** 和开始进来的 ** domestic **，它们名字并不统一

### 评论入口
同理可以发现
![](/assert/2017-03-22-3.png)
- 最热评论 hotList
- 最新评论 newList

> 注意每个二级的评论接口有可能不一样。比如国内新闻的评论接口是 ** http://comment.news.163.com **，
而军事评论接口是 ** http://comment.war.163.com **

## 代码
用 * scrapy * 来写也很简单，平常那种连续2次解析就可以了。
```python
import re
import json
import scrapy

from ..items import NewsItem, CommentItem


class NewsSpider(scrapy.Spider):
    name = 'news'
    CM_API_BASE_URL = 'http://temp.163.com/special/00804KVA/cm_{}.js?callback=data_callback'
    NEW_LIST_API_BASE_URL = 'http://comment.news.163.com/api/v1/products/a2869674571f77b5a0867c3d71db5856/threads/{}/comments/newList?offset=0&limit=30&showLevelThreshold=72&headLimit=1&tailLimit=2&callback=getData&ibc=newspc'
    pingdao = ['shehui', 'guoji','war']
    start_urls = [CM_API_BASE_URL.format('guonei')]

    def parse(self, response):
        cm_re = re.compile('data_callback\((.*)\)', re.DOTALL)
        m = cm_re.match(response.body.decode('gb18030'))
        if m:
            newses = json.loads(m.group(1))
            for news in newses:
                item = NewsItem()
                item['channelname'] = news['channelname']
                item['docurl'] = news['docurl']
                item['time'] = news['time']
                item['title'] = news['title']
                item['threadid'] = news['docurl'][-21:-5]
                yield item

                request = scrapy.Request(self.NEW_LIST_API_BASE_URL.format(item['threadid']),
                    callback=self.parse_comment)
                request.meta['threadid'] = item['threadid']
                yield request

    def parse_comment(self, response):
        threadid = response.meta['threadid']
        new_list_re = re.compile('getData\((.*)\)', re.DOTALL)
        m = new_list_re.match(response.body.decode())
        if m:
            comments = json.loads(m.group(1))['comments']
            for _, value in comments.items():
                # print(value)
                item = CommentItem()
                item['commentid'] = value['commentId']
                item['content'] = value['content']
                item['createtime'] = value['createTime']
                item['threadid'] = threadid
                item['nickname'] = value['user'].get('nickname', 'anonymous')
                item['userid'] = value['user'].get('userId', '0')
                yield item

```
## 抓取结果
只抓取了一次国内新闻。共 * 70 * 条新闻，* 2012 * 条评论。
![](/assert/2017-03-22-4.png)
![](/assert/2017-03-22-5.png)

## 需要改进的
- 因为抓取的都是最近新闻，可能没有多少评论，需要以后再次抓取评论来提高评论的数量
- 过去的新闻没有去抓
