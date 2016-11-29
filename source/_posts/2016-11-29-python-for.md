---
title: python for 语句的重新认识
date: 2016-11-29 05:15:31
tags:
  - python
  - for
---

## 起因
今天看到董伟明的《python web 开发实战》里面的 `善用for...else语句` 才发现原来还有这种用法.
我一直不知道 `for` 后面还有语句.他的例子是寻找符合条件的序列的索引值:    

    常见方法
    def find(seq, target):
        found = False
        for i, value in enumerate(seq):
            if value == target:
                found = True
                break
        if not found:
            return -1
        return i

    更好的方法
    def find(seq, target):
        for i, value in enumerate(seq):
            if value == target:
                break
        else:
            return -1
        return i

## 查看官方文档
### Tutorial -- for-statements
[for Statements](https://docs.python.org/3/tutorial/controlflow.html#for-statements)
从 `tutorial` 里面的控制语句看起,才发现用了这么久的 `python`, 才用了最简单的用法--`iter`.    

    >>> # Measure some strings:
    ... words = ['cat', 'window', 'defenestrate']
    >>> for w in words:
    ...     print(w, len(w))
    ...
    cat 3
    window 6
    defenestrate 12

这个用法就是我们最经常见到的用法了,在来看下另外一个魔法用法:    

    >>> for w in words[:]:  # Loop over a slice copy of the entire list.
    ...     if len(w) > 6:
    ...         words.insert(0, w)
    ...
    >>> words
    ['defenestrate', 'cat', 'window', 'defenestrate']

这里利用了切片的魔法 `words[:]`,它把 `words` 复制了一下, 所以 `for` 枚举的是`words`
的副本, 然后才可以在子句里面操作 `words`.这里又学到了一招,但是还是没有发现有`for...else`
的用法.

### Language Reference -- The for statement
[The for statement](https://docs.python.org/3/reference/compound_stmts.html#the-for-statement)
在这里我们可以看到 `for` 语句的 `BNF`:    

    for_stmt ::=  "for" target_list "in" expression_list ":" suite
              ["else" ":" suite]

`for` 语句是用来枚举序列(字符串,元组,列表)和可枚举对象的元素.当元素耗尽,如果有`else`子句,就会执行 `else` 子句里面的语句.如果在第一个 `suite` 里面执行了 `break` 语句就不会在执行 `else` 子句.

## 总结
- 官方文档才是最全的书.看过不少书和文章,这个 `for...else` 只见过这一次提及到.
- 学精知识就需要追根究底.比如要学精`python`,就需要深入它的本身,而不仅仅只学它的用法.就像前面的 `Tutorial` 文档就是它的用法, 而 `Language Reference` 文档才是它的本身.
