---
title: 使用python3的scrapy来写落网爬虫(二)
date: '2016/3/29 20:00:00'
tags:
  - scrapy
---

上一篇文章说了怎么抓取期刊里面的歌曲信息，这一篇文章就讲讲怎么去下载歌曲。

随便播放一首歌，用 `chrome` 的 `Network` 就可以观察到，歌曲的地址。<br>
例如，

```
http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio805/01.mp3
```

观察这个地址就可以发现， `805` 是期刊的刊号, `01.mp3` 就是歌曲的序号构成。 我们就可以根据它构造每一首歌曲的 `url` 去下载 `mp3` 格式的文件。

我用 `12` 个小时 下载了 `第1期` 到 `第805期` 的歌曲, 大约有 `八千` 首歌曲，占用空间 `40G` 左右。<br>
在下载过程中 `第1期` 到 `第99期` 出错了， 是因为下载地址构建错了，<br>
比如，我构建的地址是

```
http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio099/01.mp3
```

而正确的地址是

```
http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio99/01.mp3
```

前面的 `0` 是不需要的。

代码如下：

```python
import argparse
import os
import logging
import sys

from urllib.request import urlopen
from concurrent.futures import ThreadPoolExecutor
from pymongo import MongoClient

# http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio805/01.mp3
# http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio5/01.mp3
BASE_URL = 'http://luoo-mp3.kssws.ks-cdn.com/low/luoo/radio'

client = MongoClient()
db = client.luoo

logger = logging.getLogger('download')
logger.setLevel(logging.DEBUG)

fh = logging.FileHandler('luoo.log')
fh.setLevel(logging.DEBUG)

ch = logging.StreamHandler()
ch.setLevel(logging.DEBUG)

formatter = logging.Formatter(
    '%(asctime)s - %(name)s - %(levelname)s - %(message)s')

fh.setFormatter(formatter)
ch.setFormatter(formatter)

logger.addHandler(fh)
logger.addHandler(ch)


def download(song, basedir='luoo_download'):
    logger.info(('downloading ', song['vol_number'], song['trackname']))
    try:
        dirpath = os.path.join(
            basedir, song['vol_number'] + '_' + song['vol_title'])
        if not os.path.exists(dirpath):
            os.makedirs(dirpath)
        url = BASE_URL + str(int(song['vol_number'])) + '/' + \
            song['trackname'].split('.')[0] + '.mp3'
        data = urlopen(url).read()
        filename = os.path.join(
            dirpath, song['trackname'] + '_' + song['artist'] + '.mp3')
        if os.path.exists(filename):
            raise FileExistsError(filename)
        with open(filename, 'wb') as f:
            f.write(data)
        logger.info(('download complete', song[
                    'vol_number'], song['trackname']))
    except Exception as inst:
        logger.error(
            ('download error', song['vol_number'], song['trackname'], inst))


def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--min", type=int, default=0,
                        help="min number to download")
    parser.add_argument("--max", type=int,
                        help="max number to download")
    args = parser.parse_args()

    songs = []
    if args.max is None:
        songs = [song for song in db.music.find(
            {'vol_number': {'$gt': str(args.min)}})]
    elif args.max > 0:
        songs = [song for song in db.music.find(
            {'vol_number': {'$gt': str(args.min), '$lt': str(args.max)}})]

    with ThreadPoolExecutor(max_workers=4) as executor:
        executor.map(download, songs)


if __name__ == '__main__':
    main()
```
