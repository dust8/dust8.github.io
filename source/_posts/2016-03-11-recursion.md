---
title: 递归
date: '2016/3/11 20:00:00'
tags:
  - recursion
---

一个函数能调用自身就叫做递归。通常情况下，一个递归函数将具有一个终止条件和一个或多个递归调用其自身。

# 1.1 例子：计算指数

```
def exp(x, n):
    '''计算 x 的 n 次方
    >>> exp(2, 3)
    8
    >>> exp(3, 2)
    9
    '''
    if n == 0:
        return 1
    else:
        return x * exp(x, n-1)
```

还有斐波纳契也常做例子。

# 1.2 例子： 展开嵌套的序列

将一个多层嵌套的序列展开成一个单层序列。

```
def flatten_list(l, result=None):
    '''展开嵌套的序列
    >>> flatten_list([ [1, 2, [3, 4] ], [5, 6], 7])
    [1, 2, 3, 4, 5, 6, 7]
    '''
    if result is None:
        result = []

    for x in l:
        if isinstance(x, list):
            flatten_list(x, result)
        else:
            result.append(x)

    return result
```

# 1.3 例子：JSON编码

把python对象编码成JSON字符。

```
def json_encode(data):
    '''JSON 编码
    >>> json_encode(['foo', {'bar': ('baz', None, 1.0, 2)}])
    '["foo", {"bar":["baz", null, 1.0, 2]}]'
    '''
    if isinstance(data, bool):
        return "false" if not data else "true"
    elif isinstance(data, type(None)):
        return "null"
    elif isinstance(data, (int, float)):
        return str(data)
    elif isinstance(data, str):
        return '"' + escape_string(data) + '"'
    elif isinstance(data, (list, tuple)):
        return "[" + ", ".join(json_encode(d) for d in data) + "]"
    elif isinstance(data, dict):
        return '{' + ', '.join(json_encode(k) + ':' + json_encode(v) for k, v in data.items()) + '}'
    else:
        raise TypeError('%s is not JSON serializable, is %s' %
                        (repr(data), type(data)))


def escape_string(s):
    """Escapes double-quote, tab and new line characters in a string."""
    s = s.replace('"', '\\"')
    s = s.replace("\t", "\\t")
    s = s.replace("\n", "\\n")
    return s
```

在这里给我的 [bencoding](https://github.com/dust8/bencoding) 打个广告，<br>
它是一个 `B编码` 的 `python` 库，也是基于递归写的。
