---
title: 百度网盘搜索分析
date: '2015/10/22 20:00:00'
tags:
  - baidu
---

想搞个百度网盘搜索,今天抓了下百度网盘链接,记录下.

- <http://pan.baidu.com/pcloud/feed/getsharelist?&auth_type=1&start=0&limit=60&query_uk=3154767138>
- <http://pan.baidu.com/pcloud/user/getinfo?query_uk=3154767138>
- <http://pan.baidu.com/pcloud/friend/getfanslist?query_uk=3154767138&limit=24&start=0>
- <http://pan.baidu.com/pcloud/friend/getfollowlist?query_uk=3154767138&limit=24&start=0>

# 获取分享列表

`getsharelist` :获取分享列表<br>
`start` :分享列表起始索引<br>
`limit` :分享列表分页每页的数量<br>
`query_uk` :查询uk

返回的 `json` 数据,用 `json.loads()` 处理成 `dict`.<br>
`feed_time` 是 `毫秒`, 该用户分享该文件的时间.<br>
`path` 是 文件路径, 被url转码过, 所以可以用 `urllib.parse.unquote()` 转回来.

# 获取用户的信息

`getinfo` :获取用户的信息

同上处理成 字典.<br>
`fans_count` :粉丝数<br>
`follow_count` : 订阅数<br>
`pubshare_count` : 分享数

# 获取粉丝列表

`getfanslist` : 获取粉丝列表

# 获取订阅列表

`getfollowlist` : 获取订阅列表

通过上面4个 `url`, 就可以从一些初始的用户开始爬取他们的分享文件, 在根据他们的订阅和粉丝来获取下一步的爬取对象.

# 工具

- [json格式化](http://tool.oschina.net/codeformat/json)
- [url转码](http://tool.oschina.net/encode?type=4)
