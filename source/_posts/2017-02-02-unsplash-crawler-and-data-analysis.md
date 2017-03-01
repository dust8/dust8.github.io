---
title: unsplash爬虫与数据分析
date: 2017-02-02 05:15:31
tags:
  - 爬虫
---
## 说明
*unsplash* 网站比较慢, 只爬数据就用了30分钟, 爬了945页,每页有24条数据. 尝试下载图片,更加慢, 平均每张图片大约10M, 如果要下载完全部图片大约要 *945*24*10/1024 = 222G*, 还是算了吧.

## 爬虫代码
根据  [aosabook/500lines](https://github.com/aosabook/500lines/tree/master/crawler) 的爬虫修改而来, 尝试使用异步来写爬虫.
使用chrome浏览器开发工具来分析入口,在 network 里面的 xhr 就可以看到了, 还可以看到请求的头部,因为该网站需要 *authorization* 才可以爬取.

crawl.py      
```python
import asyncio
import crawling


def main():
    loop = asyncio.get_event_loop()
    crawler = crawling.Crawler()

    try:
        loop.run_until_complete(crawler.crawl())
    except KeyboardInterrupt:
        print('\nInterrupted\n')
    finally:
        crawler.report()
        crawler.close()

        loop.stop()
        loop.run_forever()
        loop.close()


if __name__ == '__main__':
        main()
```

crawling.py     
```python
import aiohttp
import asyncio
import time
import urllib.parse
from db import insert_many


def is_redirect(response):
    return response.status in (300, 301, 301, 303, 307)


class Crawler:
    def __init__(self,
                 max_redirect=10,
                 max_tries=4,
                 max_tasks=10,
                 *,
                 loop=None):
        self.max_redirect = max_redirect
        self.max_tries = max_tries
        self.max_tasks = max_tasks
        self.loop = loop or asyncio.get_event_loop()

        self.q = asyncio.Queue(loop=self.loop)
        self.seen_urls = set()
        self.session = aiohttp.ClientSession(loop=self.loop)
        self.headers = {
            'authorization':
            'Client-ID d69927c7ea5c770fa2ce9a2f1e3589bd896454f7068f689d8e41a25b54fa6042'
        }
        self.base_url = 'https://unsplash.com/napi/photos?per_page=24&order_by=latest&page=%s'
        self.page = 1
        self.add_url(self.base_url % (1))

        self.t0 = None
        self.t1 = None

    def add_url(self, url, max_redirect=None):
        if max_redirect is None:
            max_redirect = self.max_redirect
        self.seen_urls.add(url)
        self.q.put_nowait((url, max_redirect))

    async def crawl(self):
        workers = [
            asyncio.Task(self.work(), loop=self.loop)
            for _ in range(self.max_tasks)
        ]
        self.t0 = time.time()
        await self.q.join()
        self.t1 = time.time()
        for w in workers:
            w.cancel()

    def report(self):
        print('total time: ', self.t1 - self.t0)
        print('total page:', self.page)

    async def work(self):
        try:
            while True:
                url, max_redirect = await self.q.get()
                await self.fetch(url, max_redirect)
                self.q.task_done()
        except asyncio.CancelledError:
            pass

    async def fetch(self, url, max_redirect):
        tries = 0
        while tries < self.max_tries:
            try:
                response = await self.session.get(
                    url, headers=self.headers, allow_redirects=False)
                break
            except aiohttp.ClientError:
                pass
            tries += 1
        else:
            # all tries failed.
            return

        try:
            if is_redirect(response):
                location = response.headers['location']
                next_url = urllib.parse.urljoin(url, location)
                if next_url in self.seen_urls:
                    return
                if max_redirect > 0:
                    self.add_url(next_url, max_redirect - 1)
            else:
                links = await self.parse_links(response)
                for link in links.difference(self.seen_urls):
                    self.q.put_nowait((link, self.max_redirect))
                self.seen_urls.update(links)
        finally:
            await response.release()

    async def parse_links(self, response):
        links = set()

        if response.status == 200:
            data = await response.json()
            if data:
                insert_many(data)
                self.page += 1
                print('next', self.base_url % (self.page))
                links.add(self.base_url % (self.page))

        return links

    def close(self):
        self.session.close()    
```

请求回来的数据比较乱，还是用 mongdb 存储比较方便  
db.py    
```python
import pymongo

client = pymongo.MongoClient()
db = client.unsplash
photo = db.photo


def insert_many(photos):
    photo.insert_many(photos)
```

## 数据分析    
### 提取数据    
```python
import pymongo

client = pymongo.MongoClient()
db = client.unsplash
photo = db.photo
photos = []
for p in photo.find():
    photos.append(p)
```
### 数据总量

    len(photos)  

    22648

### 定义常用函数
```python
from collections import defaultdict

def get_counts(sequence):
    counts = defaultdict(int)
    for x in sequence:
        counts[x] += 1
    return counts

def top_counts(count_dict, n=10):
    value_key_pairs = [(count, key) for key, count in count_dict.items()]
    value_key_pairs.sort()
    value_key_pairs_n = value_key_pairs[-n:]
    value_key_pairs_n.sort(reverse=True)
    return value_key_pairs_n
```
### top10 图片大小
```python
width_heights = [str(p['width'])+'_'+str(p['height']) for p in photos]
width_heights_counts = get_counts(width_heights)
print(len(width_heights_counts))
top_counts(width_heights_counts)
```    

    7891
    [(1751, '5184_3456'),
     (1132, '6000_4000'),
     (999, '5472_3648'),
     (575, '5760_3840'),
     (462, '6016_4016'),
     (427, '5616_3744'),
     (404, '4896_3264'),
     (392, '4928_3264'),
     (321, '3264_2448'),
     (316, '4288_2848')]

### top10 颜色
```python
colors = [p['color'] for p in photos if 'color' in p and p['color'] is not None]
colors_counts = get_counts(colors)
print(len(colors_counts))
top_counts(colors_counts)
```    

    19434
    [(119, '#FFFFFF'),
     (73, '#FEFEFE'),
     (45, '#010101'),
     (42, '#FCFCFC'),
     (41, '#FDFDFD'),
     (41, '#FBFBFB'),
     (29, '#020202'),
     (28, '#F9F9F9'),
     (24, '#FAFAFA'),
     (22, '#FEFEFD')]

### top10 每年照片总数
```python
created_ats = [ p['created_at'][:4] for p in photos]
created_ats_counts = get_counts(created_ats)
top_counts(created_ats_counts)
```
    [(12761, '2016'),
     (5467, '2015'),
     (2702, '2017'),
     (1498, '2014'),
     (220, '2013')]
