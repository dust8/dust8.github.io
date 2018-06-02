---
title: pyppeteer使用总结
date: 2018-06-03 05:15:31
---

最近有个爬虫项目网站太多，绝大部分接口都是json，有些还有参数加密，
一个个分析接口就太麻烦，就想用浏览器全部渲染出来，就可以省掉这些步骤。
最近流行 `headless` 就用它了。`puppeteer` 是 `nodejs` 的，它的 `python`
绑定比较好的是 [pyppeteer](https://github.com/miyakogi/pyppeteer)。

## 基本用法
中文资料非常少，接口看文档，例子看 `tests` 下面的测试用例。
还可以看看 `puppeteer` 的教程。    

### 启动参数
```python
browser = await pyppeteer.launch({
            'devtools':
            False,
            'args': ['--no-sandbox', '--user-agent="' + UserAgent + '"']
        })
```
`devtools` 控制界面的显示，用来调试。`args` 是浏览器启动的命令行参数，可以设置浏览器头部，
不然会标示为无头浏览器。`--no-sandbox` 在 `docker` 里使用时需要加入的参数，不然会报错。  


### 请求钩子
为了加快渲染速度，可以禁止加载图片之类的。
```python
async def request_check(req):
    '''请求过滤'''
    if req.resourceType in ['image', 'media', 'eventsource', 'websocket']:
        await req.abort()
    else:
        await req.continue_()

await page.setRequestInterception(True)
page.on('request', request_check)
```

### 网络问题
请求加载是否完成，无网都需要处理
```python
async def goto(page, url):
    while True:
        try:
            await page.goto(url, {
                'timeout': 0,
                'waitUntil': 'networkidle0'
            })
            break
        except (pyppeteer.errors.NetworkError,
                pyppeteer.errors.PageError) as ex:
            # 无网络 'net::ERR_INTERNET_DISCONNECTED','net::ERR_TUNNEL_CONNECTION_FAILED'
            if 'net::' in str(ex):
                await asyncio.sleep(10)
            else:
                raise
```

### 注入js文件
比如一些js库，我使用 [Ajax-hook](https://github.com/wendux/Ajax-hook) 来统计 ajax 的请求完成情况，
需要在网页头部注入js文件，一些自己的库，比较大，也这样注入。    
```python
from pathlib import Path

CURDIR = Path(__file__).parent
JS_AJAX_HOOK_LIB = str(CURDIR / 'static' / 'ajaxhook.min.js')

await page.addScriptTag(path=JS_AJAX_HOOK_LIB)
```
这样注入的js文件不能有中文，因为 `pyppeteer` 里面打开文件用的是默认编码，可以 hook 住
`open` 函数来解决。
```python
# 因为 pyppeteer 库里面 addScriptTag 用的是系统默认编码,导致js文件里面不能有中文
pyppeteer.frame_manager.open = lambda file: open(file, encoding='utf8')
```

### 在docker里使用
在 window10 里开发很流程，部署到 windows server 上，可能由于配置比较差或其他原因，网站渲染很慢。
可以放在容器里，效果明显。注意点是上面提到了的关闭沙盒模式，需要下一些浏览器的依赖，还有就是最好先把浏览器下好，做到镜像里，这样
就不会在容器里一个一个下了。    
```dockerfile
FROM python:slim

WORKDIR /usr/src/app

RUN apt-get update && apt-get install -y gconf-service libasound2 libatk1.0-0 libc6 libcairo2 libcups2 libdbus-1-3 libexpat1 libfontconfig1 libgcc1 libgconf-2-4 libgdk-pixbuf2.0-0 libglib2.0-0 libgtk-3-0 libnspr4 libpango-1.0-0 libpangocairo-1.0-0 libstdc++6 libx11-6 libx11-xcb1 libxcb1 libxcomposite1 libxcursor1 libxdamage1 libxext6 libxfixes3 libxi6 libxrandr2 libxrender1 libxss1 libxtst6 ca-certificates fonts-liberation libappindicator1 libnss3 lsb-release xdg-utils wget
RUN apt-get install -y vim

COPY requirements.txt ./
RUN pip install --no-cache-dir -r requirements.txt
RUN python -c "import pyppeteer;pyppeteer.chromium_downloader.download_chromium();"


COPY . .

VOLUME /data
```
