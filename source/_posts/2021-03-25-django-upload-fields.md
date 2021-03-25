---
title: django之DATA_UPLOAD_MAX_NUMBER_FIELDS
date: 2021-03-25 05:15:31
tags:
  - django
---

[DATA_UPLOAD_MAX_NUMBER_FIELDS](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-DATA_UPLOAD_MAX_NUMBER_FIELDS) 可以用来设置通过 GET 或 POST 接收到的参数的最大数量.默认是 1000.

后台管理页面有个 `TabularInline`和`StackedInline` ,里面的内容条数太多导致报错.可以通过在配置文件里面设置 `DATA_UPLOAD_MAX_NUMBER_FIELDS` 的值来解决, 例如设置为
```python
DATA_UPLOAD_MAX_NUMBER_FIELDS = 1000*10
```

## 参考链接
- [DATA_UPLOAD_MAX_NUMBER_FIELDS](https://docs.djangoproject.com/zh-hans/3.1/ref/settings/#std:setting-DATA_UPLOAD_MAX_NUMBER_FIELDS)