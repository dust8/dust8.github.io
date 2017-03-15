---
---
title: stuq动态爬虫

date: 2017-03-08 05:15:31
tags:
  - 爬虫
---

## 前言
最近看了下 **openrestry** ，它是 **ngnix** 和 **lua** 的结合体，用于方便地搭建能够处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。可以构造出足以胜任 10K 乃至 1000K 以上单机并发连接的高性能 Web 应用系统。搜索它的视频资料，一个在网易云课堂上，只有1/3的内容，另外一个在 *stuq* 上，是全部内容。我就想把视频爬下来。

## 分析
### 分析视频地址
通过chrome开发工具的 *Elements* 发现没有直接暴露地址，是用 flash 播放的，同时可以知道视频的id 用 **vid** 来表示。
通过 **Network** 可以发现几个有用的请求。它们是 **getvideofile** **player.swf** **playinfo**.
真实的视频地址就在 **playinfo** 这个请求里面。分析这个请求的查询参数
可以知道只有2个参数是必须的，一个是 **vid**(视频id), 一个是 **m** 它是固定的。当时还反编译了一下 **player.swf** 看了下其他参数是什么。其实这个接口没有怎么防范，通过不停的减少参数就可以把这个分析出来。

### 分析视频列表
查看网页源代码可以看到有视频的id，有2个问题是，一个是要登录，一个是它用的是 **vue** 。而且也没有发现有api请求，所以只能用 **selenium** 来模拟请求了。

## 实现
我把抓取视频id和下载视频分开，用 **selenium** 来抓取视频id和文件名列表，保存为json，在用 **aiohttp** 来下载视频文件，所以会快些。

stuq_spider.py
```python
import time
import json
from selenium import webdriver
from selenium.webdriver.common.keys import Keys


class StuqSpider:
    def __init__(self):
        self.driver = webdriver.Chrome('/Users/tom/workspace/chromedriver')

    def crawl_course(self, courseid):
        coursewares = self.get_coursewares(courseid)
        data = []
        for url in coursewares:
            info = self.get_video_info(url)
            data.append(info)
        self.store_data(data)

    def login_weibo(self, username, password):
        self.driver.get('https://passport.stuq.org/user/login')
        self.driver.find_element_by_xpath('//a[@class="button weibo"]').click()

        userid = self.driver.find_element_by_id('userId')
        userid.send_keys(username)
        passwd = self.driver.find_element_by_id('passwd')
        passwd.send_keys(password)

        # 放慢速度好伪装
        time.sleep(5)

        self.driver.find_element_by_xpath(
            '//*[@id="outer"]/div/div[2]/form/div/div[2]/div/p/a[1]'
        ).send_keys(Keys.ENTER)

        # 手动登录,要输入验证码
        time.sleep(20)

    def get_coursewares(self, courseid):
        coursewares = []
        self.driver.implicitly_wait(5)
        self.driver.get('http://www.stuq.org/course/' + courseid + '/study')
        for a in self.driver.find_elements_by_xpath('//p[@class="video"]/a'):
            coursewares.append(a.get_attribute('href'))
        return coursewares

    def get_video_info(self, url):
        self.driver.implicitly_wait(2)
        self.driver.get(url)
        h2 = self.driver.find_element_by_tag_name('h2').text
        videoid = self.driver.find_element_by_xpath(
            '//section[@id="cc-video"]/div').get_attribute('id').split('_')[2]
        return [h2, videoid]

    def store_data(self, data):
        with open('stuq.json', 'w') as fp:
            json.dump(data, fp)

    def close(self):
        self.driver.close()
        self.driver.quit()


username = '' # sina 账号
password = '' # sina 密码
courseid = '1015'

spider = StuqSpider()
try:
    spider.login_weibo(username, password)
    spider.crawl_course(courseid)
finally:
    spider.close()
```

bokecc_crawler.py
```python
import aiohttp
import asyncio
import json
import time
import os.path
import xml.etree.ElementTree as ET

base_url = 'https://p.bokecc.com/servlet/playinfo?m=1&vid='
chunk_size = 64


class Crawler:
    def __init__(self, max_task=4, loop=None):
        self.max_task = 4
        self.loop = loop or asyncio.get_event_loop()
        self.q = asyncio.Queue(loop=self.loop)
        self.session = aiohttp.ClientSession(loop=self.loop)
        self.t0 = self.t1 = 0

    def add_task(self, task):
        self.q.put_nowait(task)

    async def crawl(self):
        workers = [
            asyncio.Task(self.work(), loop=self.loop)
            for _ in range(self.max_task)
        ]
        self.t0 = time.time()
        await self.q.join()
        self.t1 = time.time()
        for w in workers:
            w.cancel()

    async def work(self):
        try:
            while True:
                filename, vid = await self.q.get()
                print('working:', filename, vid)
                xmldata = await self.fetch(base_url + vid)
                root = ET.fromstring(xmldata)
                copys = root.findall(".//copy")
                playurl = copys[-1].attrib['playurl']
                await self.download_video(playurl, filename)
                self.q.task_done()
        except asyncio.CancelledError:
            pass

    async def fetch(self, url):
        async with self.session.get(url) as response:
            return await response.text()

    async def download_video(self, url, filename):
        async with self.session.get(url) as response:
            with open(os.path.join('videos', filename), 'wb') as fd:
                while True:
                    chunk = await response.content.read(chunk_size)
                    if not chunk:
                        break
                    fd.write(chunk)

    def close(self):
        self.session.close()

    def report(self):
        print('total time: ', self.t1 - self.t0)


loop = asyncio.get_event_loop()
crawler = Crawler(loop=loop)

with open('stuq.json') as fp:
    data = json.load(fp)

for filename, vid in data:
    filename += '.flv'
    crawler.add_task((filename, vid))
try:
    loop.run_until_complete(crawler.crawl())
finally:
    crawler.report()
    crawler.close()
```

## 爬取成果
爬了20分钟，下了29个视频，共2.1G。
![stuq_spider](/assert/2017-03-08-stuq.png)
