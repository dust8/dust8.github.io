---
title: bt爬虫系统
date: 2016-06-02 11:49:36
tags:
  - dht
  - bt
  - crawler
---

# 前言

好几年前就开始接触 `dht` 了，那时还是 `小虾` 引起的潮流.
大概原理基本都了解了, 就是代码写的难看，执行效率也不高。
这几年有很多人也接触到这些，开源了不少简洁而又高效的代码。

# 技术表

## B 编码

我用的是 `python3`, 不是主流，所以要自己写编码和解码，有了 [bencoding](https://github.com/dust8/bencoding)。

### 官方的库

[bencode](https://github.com/bittorrent/bencode), 它只支持 `python2`.

### 非官方的库

[bencoder.pyx](https://github.com/whtsky/bencoder.pyx), A fast bencode implementation in Cython.

## infohash 的抓取

### 官方的 bootstrap-dht

[bootstrap-dht](https://github.com/bittorrent/bootstrap-dht), 它简单的示例了下 `dht` 启动服务器。

### 非官方的

- [simDHT](https://github.com/Fuck-You-GFW/simDHT), 它是老太太的杰作, 简单又简洁。
  它最先解决了怎么才能高效的认识其他节点，也就是改变发送的 id。id 的高位也就是最前面的影响最大，距离越远。

- [maga](https://github.com/whtsky/maga), 这是最近发现的，它使用了 `asyncio`, 非常高效，代码也
  很简洁，推荐看一下。

## 种子信息的抓取

### dht 网络抓取

这种方法不会受限于人，它根据`bittorrent`协议去下载信息。优点是全，稳, 缺点就是慢.
如果直接根据 `announce_peer` 去下载，有很多 peer 连不上，有很多 peer 自己都还没下载完成。

### 其他提供者抓取

这种方法就是去抓像迅雷这样的提供者.优点是快，缺点是不全，要看它们的脸色.

## 坑

### ip 和端口的解析

`ip` 4 个字节，怎么转成 '192.168.1.1' 这样，用 `socket.inet_ntoa`,半路出家的坑啊。

### 路由表

写爬虫完全不需要

### 效率

像 `maga` 那样，效率太高了，`cpu` 和 网络占用太多。 给它定个限制，一分钟只能发多少个包，
超过的直接就不发了，因为是爬虫，丢包没影响。

### 文件名解码

绝大多数可以用 `utf8` 解码，出错在用 `gb18030`, 剩下的就难搞了，只能丢掉了。

# 爬虫架构

![bt-crawler-system](./assert/2016-06-02-bt-crawler-system.PNG)

1. `infohash` 抓取用 `maga` 改的
2. `metadata` 抓取用 迅雷的地址
3. `infohash` 传递用 `rabbitmq` 和 `celery`
4. 数据库用 `mongodb`, 全文搜索就不好搞， 可以考虑用 `elasticsearch`.
