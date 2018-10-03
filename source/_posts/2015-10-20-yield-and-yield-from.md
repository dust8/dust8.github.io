---
title: yield 和 yield from 的理解
date: '2015/10/20 20:00:00'
tags:
  - yield
  - yield from
---

`yield` 表达式文档: [Yield expressions](https://docs.python.org/3/reference/expressions.html#yield-expressions)

`yield` 表达式的作用有点像 `return`, 但是又有些不同.<br>
相同的是执行到它就会返回, 不同的是 `return` 不会再回来; `yield` 还会回来. 举个栗子:

```
def p():
    i = 0
    while True:
        print('start', i)
        yield i
        i += 1
        print('end', i)

p = p()

next(p)
print('-'*10)
next(p)
print('-'*10)
next(p)
print('-'*10)
next(p)
```

输出为:

```
start 0
----------
end 1
start 1
----------
end 2
start 2
----------
end 3
start 3
```

可以看到,第一次执行到 `yield` 就返回了; 第二次则是先执行 `yield`后面的,然后循环到 `yield` 再返回, 重要的是它还记得你上次返回时的状态(例如 `i` 的值), 而 `return` 就不会.

`yield from` 表达式文档: [yield from](https://docs.python.org/3/whatsnew/3.3.html#pep-380-syntax-for-delegating-to-a-subgenerator)

举个栗子:

```
def g(x):
    print('this first yield from')
    yield from range(x, 0, -1)
    print('this second yield from')
    yield from range(x)


for i in g(5):
    print(i)
```

输出为:

```
this first yield from
5
4
3
2
1
this second yield from
0
1
2
3
4
```

`yield from` 后面是迭代器, 并且 `yield` 完这个迭代器, 才会去执行第二个 `yield from`.
