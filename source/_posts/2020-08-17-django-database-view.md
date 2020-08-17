---
title: django与数据库视图
date: 2020-08-17 05:15:31
tags:
  - django
---

项目里面使用了视图, 使用的时候报错 `Unknown column 'xxx.id' in 'field list'`.  
原因是 `django` 模型里面必须有主键, 看哪个字段适合做主键就设置为主键就可以了.

参考链接:

- [django 调用 MySQL 视图时报错 "Unknown column 'project_staff.id' in 'field list'"](https://blog.csdn.net/qq_42303913/article/details/103737924)
