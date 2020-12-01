---
title: django与coalesce
date: 2020-12-01 05:15:31
tags:
  - django
  - mysql
---

在 `django` 使用统计 `annotate` 和 `aggregate` 时由于没有数据导致统计是 `null` , 而不是 `0`. 
这对返回结果产生很大影响, 可以使用 `Coalesce` 方法来设置默认值.

```python
from django.db.models import Sum, Value as V
from django.db.models.functions import Coalesce

>>> # Prevent an aggregate Sum() from returning None
>>> aggregated = Author.objects.aggregate(
...    combined_age=Coalesce(Sum('age'), V(0)),
...    combined_age_default=Sum('age'))
>>> print(aggregated['combined_age'])
0
>>> print(aggregated['combined_age_default'])
None
```


## 参考链接
- [Coalesce](https://docs.djangoproject.com/zh-hans/3.1/ref/models/database-functions/#coalesce)
- [coalesce 百科](https://baike.baidu.com/item/coalesce/10348978)
