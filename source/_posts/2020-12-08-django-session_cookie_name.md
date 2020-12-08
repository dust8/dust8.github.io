---
title: django与SESSION_COOKIE_NAME
date: 2020-12-08 05:15:31
tags:
  - django
---

在服务器用端口部署了多个 `django` 后台, 导致只能登录一个后台, 登录信息被覆盖了.    
查了下可以更改配置文件里面的 `SESSION_COOKIE_NAME` 来区分登录信息.    
默认的值是 `sessionid`, 可以改成不一样的, 这样就区分开来了.    
设置好后 `打开浏览器开发控制台 -> Application -> Cookies` 里面看到是否设置成功.

```python
# settings.py
SESSION_COOKIE_NAME = "sessionid_xxx"
```


## 参考链接
- [SESSION_COOKIE_NAME](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#session-cookie-name)
