---

title: sentry 试用
date: 2018-02-22 05:15:31
tags:
---## sentry 简介

官网: [https://sentry.io](https://sentry.io)  
sentry 是一个错误跟踪系统。可以用来记录错误日志，但不能用来替代日志，因为它主打的是跟踪错误。

## 本地安装

官方有很好的 sass 服务，但是我选本地安装。本地也有 2 种：python 和 docker。当然用官方推荐的 docker。

去 github 上克隆 [getsentry/onpremise](https://github.com/getsentry/onpremise) 下来,
要安装 github 上的来安装，不要按照 sentry 文档上的来安装，会简单很多。

docker 换 中国科学技术大学 的开源镜像会快很多：  
https://docker.mirrors.ustc.edu.cn
![](/blog/assert/2018-02-22.png)

## 使用

浏览器打开 localhost:9000 就可以用了。

### 浏览器打开不了里面的某些链接

看浏览器的 url 地址栏就发现问题了：
http://localhost:9000/organizations/sentry/members/127.0.0.1/organizations/sentry/rate-limits/

这是相对路径不对。

### raven 里面的 dsn 填什么

浏览器里面给的是：
://168b3dd55f7c47919b511d0ae5efe5f1:135f030ae86b458aa19646176f5e7a7a@127.0.0.1/
2

前面要加 http, ip 地址后面要加端口号。
