---
title: 使用 you-get 下载 youtube 稍后观看视频
date: 2017-12-13 05:15:31
tags:
  - you-get
---

# 起因

前不久在帮瓦工买了个 vps，把月付看成了年付，马上就过期了。
本着不浪费的精神想把流量用完，想下载 youtube 的视频来用完它。
you-get 是不错，可惜只能一个一个下。

# 获取所有下载的链接地址

在浏览器里打开审查，用 xpath 找出所有的视频链接。

```js
var hrefs = $x(
  '//ytd-playlist-video-renderer[@class="style-scope ytd-playlist-video-list-renderer"]//a[@is="yt-endpoint"]/@href'
);
```

打印出所有的视频链接

```js
for (var i = 0, len = hrefs.length; i < len; i++) {
  console.log(hrefs[i].textContent);
}
```

![](./assert/2017-12-13.png)

把链接复制到 youtube.txt 文件里面去。

# 用 bash 和 you-get 批量下载

```bash
cat youtueb.txt | while read line
do
you-get -s localhost:1086 https://www.youtuebe.com$line
done
```
