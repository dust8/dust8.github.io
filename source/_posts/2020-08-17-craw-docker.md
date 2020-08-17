---
title: 爬虫与docker
date: 2020-08-17 05:16:31
tags:
  - 爬虫
  - docker
---

最近又写了个爬虫, 爬美团的商家.总结一下新写法.
因为接口是加密的,懒得去找参数了,直接用浏览器渲染, 试了下 `request-html` 效果不好, 也不想用 `pyppeteer` 来写, 选择了无脑的渲染服务 `splash`.

## javascript 渲染服务

[splash 安装](https://splash.readthedocs.io/en/stable/install.html) 直接用 `docker` 来安装, 方便快捷

```
docker run -it -p 8050:8050 --rm scrapinghub/splash
```

## 代理服务

由于有反爬, 所以需要不停的换 ip 地址, 不然会出现验证码等反爬响应.买的话性价比也不高, 选择了开源的自建代理池 [https://github.com/kagxin/proxy-pool](https://github.com/kagxin/proxy-pool) 也是支持用 `docker` 部署

## 重试库

免费代理质量一般都不咋样, 所以需要不停的重试来保障请求的完整.下面是很粗暴的解析不到数据就主动引发错误来重试.

```
from retrying import retry

splash_url = "http://localhost:8050/render.html"

@retry
def get_shop_urls(url, headers=None):
    proxy = get_proxy()
    loggers.info(f"get_shop_urls proxy {proxy} {url}")
    params = {
        "url": url,
        "http_method": "GET",
        "headers": {
            "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
            "user-agent": random.choice(USER_AGENTS),
        },
        "wait": 2,
        "images": 0,
        "proxy": proxy,
    }
    html = requests.post(
        splash_url,
        data=json.dumps(params),
        headers={"Content-Type": "application/json"},
    ).text

    if "对不起" in html:
        return []

    tree = fromstring(html)
    links = []
    for item in tree.xpath('//ul[@class="list-ul"]//div[@class="info"]//a/@href'):
        links.append(item)

    if not links:
        raise ValueError()

    return links
```

## 参考链接

- [Splash - A javascript rendering service](https://github.com/scrapinghub/splash)
- [Python3：给 splash 的 render.html 接口添加请求头](https://blog.csdn.net/u012552769/article/details/88028468)
- [rholder/retrying](https://github.com/rholder/retrying)
