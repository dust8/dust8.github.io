---
title: mysql unicode 编码内容的查询
date: 2020-11-21 05:15:31
tags:
  - unicode
  - mysql
---

`django` 升级后 `django_admin_log` 表里面的 `change_message` 字段里面的内容变成了中文, 估计是记录字段的 `verbose_name` 了.类型下面    
```json
[{"changed": {"fields": ["\u5e10\u53f7\u603b\u989d", "\u53ef\u7528\u4f59\u989d"]}}]
```


```sql
SELECT * FROM `django_admin_log` where `change_message` LIKE CONCAT('%','_u5e10_u53f7_u603b_u989d','%')
```