---
title: django查询的坑
date: 2020-08-24 05:15:31
tags:
  - django
---

这些坑都是官方文档有说明的, 但是二手资料太多, 误人啊.

## F 表达式的坑

为了避免条件竞争, 多个地方操作同一条记录,造成数据错误, 就使用了 `F` 表达式. 而在后面又使用了该记录的实例, 造成 `F()` 执行了多遍, 数据错误, 需要保存后使用 `refresh_from_db()` 来重载模型对象才能避免出错.

```
# 错误, 会执行 2 次 F 表达式
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()

reporter.name = 'Tintin Jr.'
reporter.save()
```

```
# 正确, F 表达式只执行了 1 次
reporter = Reporters.objects.get(name='Tintin')
reporter.stories_filed = F('stories_filed') + 1
reporter.save()
reporter.refresh_from_db()

reporter.name = 'Tintin Jr.'
reporter.save()
```

## annotate 表达式的坑

下面是取 `user_id` 最大的一个权重记录查询, 要注意使用 `order_by()` 不然的话会包含模型里面的 `ordering` 排序

```
hitlogs = (
        HitLog.objects.filter(user_id__in=user_ids)
        .values("user_id")
        .annotate(weight=Max("weight"))
        .order_by()
    )
```

> 2.2 版后已移除:
> 从 Django 3.1 中开始，模型的 Meta.ordering 中的排序不会使用在 GROUP BY 查询，
> 比如 .annotate().values() 。从 Django 2.2 开始，这些查询发出一个弃用警告，
> 指示要在查询集中添加一个显式的 order_by() 来静默警告。

## 参考链接

- [F() assignments persist after Model.save()](https://docs.djangoproject.com/zh-hans/2.2/ref/models/expressions/#f-assignments-persist-after-model-save)
- [annotate() 和 values() 的顺序-与默认排序或 order_by() 交互](https://docs.djangoproject.com/zh-hans/2.2/topics/db/aggregation/#order-of-annotate-and-values-clauses)
- [django group by 的坑](https://blog.csdn.net/qq_38534144/article/details/105468424)
