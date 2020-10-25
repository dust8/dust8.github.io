---
title: records 源码阅读
date: 2020-10-25 05:15:31
tags:
  - python
  - 源码阅读
---

[kennethreitz-archive/records](https://github.com/kennethreitz-archive/records):
Records: SQL for Humans™. 
`records` 是 `python` 的一个简单的库, 代码量比较少, 用来学习阅读笔记容易懂. 它主要是封装了 `SQLAlchemy` 和 `Tablib` 库, 一个用来处理数据库的操作, 一个用来格式化各种导出. 从中我们可以学会如何组合已有的库. 


## 源码阅读
### 找入口
通过官方的文档示例
```
import records

db = records.Database('postgres://...')
rows = db.query('select * from active_users')    # or db.query_file('sqls/active-users.sql')
```

从上面可以看出, `Database` 类是主要的入口类. 
```
class Database(object):
    """A Database. Encapsulates a url and an SQLAlchemy engine with a pool of
    connections.
    """

    def __init__(self, db_url=None, **kwargs):
        # If no db_url was provided, fallback to $DATABASE_URL.
        self.db_url = db_url or os.environ.get('DATABASE_URL')

        if not self.db_url:
            raise ValueError('You must provide a db_url.')

        # Create an engine.
        self._engine = create_engine(self.db_url, **kwargs)
        self.open = True
```



### 主要类的关系

![](/assert/2020-10-25-1.png)    
`Database` 类的数据库操都是操作 `Connection` 类, 而 `Connection` 是 `SQLAlchemy` 的连接封装.
```
class Database(object):
    ...

    def get_connection(self):
        """Get a connection to this Database. Connections are retrieved from a
        pool.
        """
        if not self.open:
            raise exc.ResourceClosedError('Database closed.')

        return Connection(self._engine.connect())

    def query(self, query, fetchall=False, **params):
        """Executes the given SQL query against the Database. Parameters can,
        optionally, be provided. Returns a RecordCollection, which can be
        iterated over to get result rows as dictionaries.
        """
        with self.get_connection() as conn:
            return conn.query(query, fetchall, **params)
```

```
class Connection(object):
    """A Database connection."""

    def __init__(self, connection):
        self._conn = connection
        self.open = not connection.closed

    def query(self, query, fetchall=False, **params):
        """Executes the given SQL query against the connected Database.
        Parameters can, optionally, be provided. Returns a RecordCollection,
        which can be iterated over to get result rows as dictionaries.
        """

        # Execute the given query.
        cursor = self._conn.execute(text(query), **params) # TODO: PARAMS GO HERE

        # Row-by-row Record generator.
        row_gen = (Record(cursor.keys(), row) for row in cursor)

        # Convert psycopg2 results to RecordCollection.
        results = RecordCollection(row_gen)

        # Fetch all results if desired.
        if fetchall:
            results.all()

        return results
```

## 从源码学使用
因为没有文档, 有些使用需要看源码才会, 比如使用事务
```
class Database(object):
    ...

    @contextmanager
    def transaction(self):
        """A context manager for executing a transaction on this Database."""

        conn = self.get_connection()
        tx = conn.transaction()
        try:
            yield conn
            tx.commit()
        except:
            tx.rollback()
        finally:
            conn.close()
```

可以看到返回的是一个支持上下文管理的连接对象

```
with db.transaction() as tx:
    user = {"name": "yuze9", "age": 20}
    tx.query('INSERT INTO lemon_user(name,age) values (:name, :age)', **user)
    # 下面是错误的 sql 语句，有错误，则上面的 sql 语句不会成功执行。
    tx.query('sof')
```


## 参考链接

- [如何使用python records 库优雅的操作数据库](https://www.cnblogs.com/wagyuze/p/11398484.html)
- [contextlib --- 为 with语句上下文提供的工具](https://docs.python.org/zh-cn/3/library/contextlib.html)