---
title: 用 python 方便的插入原生 sql 数据
date: 2020-05-23 05:15:31
tags:
  - python
  - sql
---

使用原生 `sql` 插入数据库的时候, 如果字段太多会导致写起来太麻烦,就想了个自动生成插入语句的代码.
前提是字段名要和数据库一致, 如果不一致可以自己在加一层代码, 做一些键名映射.

```python

def insert(conn,table_name, data, sep='?'):
    cur = conn.cursor()

    # 把字典变成键值对数组
    zipped = []
    for key, value in data.items():
        zipped.append((key, value))

    # 把键值对数组变成对应的键数组和值数组
    keys, values = zip(*zipped)

    sql = f"insert into {table_name}({', '.join(keys)}) values ({', '.join([sep for _ in range(len(values))])})"
    cur.execute(sql, values)
    conn.commit()
    cur.close()
```
