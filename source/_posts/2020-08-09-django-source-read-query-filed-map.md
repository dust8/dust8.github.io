---
title: django源码阅读-查询字段映射
date: 2020-08-09 05:15:31
tags:
  - django
  - 源码阅读
---

在 [django 执行原生查询](https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/#performing-raw-queries) 文档里面的 `Manager.raw(raw_query, params=None, translations=None)` 里面有说 [将查询字段映射为模型字段](https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/#mapping-query-fields-to-model-fields) 就是根据 `translations` 参数来映射.

```python
>>> name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
>>> Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)
```

通过 `IDE` 的跟踪可以看到它的实现方式

```python
class RawQuerySet:
    ...
    @cached_property
    def columns(self):
        """
        A list of model field names in the order they'll appear in the
        query results.
        """
        columns = self.query.get_columns()
        # Adjust any column names which don't match field names
        for (query_name, model_name) in self.translations.items():
            # Ignore translations for nonexistent column names
            try:
                index = columns.index(query_name)
            except ValueError:
                pass
            else:
                columns[index] = model_name
        return columns

```

```python
class RawQuery:
    ...

    def get_columns(self):
        if self.cursor is None:
            self._execute_query()
        converter = connections[self.using].introspection.identifier_converter
        return [converter(column_meta[0])
                for column_meta in self.cursor.description]
```

可以看到关键点是 **`self.cursor.description`**

在 [直接执行自定义 SQL](https://docs.djangoproject.com/zh-hans/3.1/topics/db/sql/#executing-custom-sql-directly) 里面有把查询结果变成字典类型的方法

```python
def dictfetchall(cursor):
    "Return all rows from a cursor as a dict"
    columns = [col[0] for col in cursor.description]
    return [
        dict(zip(columns, row))
        for row in cursor.fetchall()
    ]
```

在 `python` 官方的 `sqlite3` 模块里面的 [description](https://docs.python.org/zh-cn/3/library/sqlite3.html?highlight=sqlite#sqlite3.Cursor.description)字段描述可以看到

>

    description
    这个只读属性将提供上一次查询的列名称。 为了与 Python DB API 保持兼容，它会为每个列返回一个 7 元组，每个元组的最后六个条目均为 None。
    对于没有任何匹配行的 SELECT 语句同样会设置该属性。

虽然知道它的实现, 还是感觉在 `django` 里面多此一举, 因为正常 `queryset` 没法用, 用原生的直接用数据库的 `AS` 关键字就可以了.
