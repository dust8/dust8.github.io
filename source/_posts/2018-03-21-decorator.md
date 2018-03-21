---
title: python装饰器
date: 2018-03-21 05:15:31
tags:
---
![decorator](/assert/2018-03-21.png)    

## 基础知识
装饰器是可调用的对象,其参数是另一个函数(被装饰的函数).

## 手写装饰器
缺点:    
- 不支持关键字参数
- 遮盖了被装饰函数的 `__name__` 和 `__doc__` 属性    


```python
import time


def clock(func):
    def wrapper(*args):
        t0 = time.perf_counter()
        result = func(*args)
        elapsed = time.perf_counter() - t0
        name = func.__name__
        arg_str = ','.join(repr(arg) for arg in args)
        print(f'[%0.8fs] {name}({arg_str}) -> {result}' % (elapsed))
        return result

    return wrapper


@clock
def snooze(seconds):
    time.sleep(seconds)


@clock
def factorial(n):
    return 1 if n < 2 else n * factorial(n - 1)


if __name__ == '__main__':
    print('*' * 40, 'Calling snooze(.123')
    snooze(.123)
    print('*' * 40, 'Calling factorial(6)')
    factorial(6)
```

输出    

```python
**************************************** Calling snooze(.123
[0.12387697s] snooze(0.123) -> None
**************************************** Calling factorial(6)
[0.00000217s] factorial(1) -> 1
[0.00005243s] factorial(2) -> 2
[0.00015136s] factorial(3) -> 6
[0.00019293s] factorial(4) -> 24
[0.00023032s] factorial(5) -> 120
[0.00027032s] factorial(6) -> 720
```

## 标准库里的装饰器
`functools.wraps` 是标准库里的装饰器,它的作用是协助构建行为良好的装饰器,解决了手写装饰器的缺点.
```python
from functools import wraps

>>> def my_decorator(f):
...     @wraps(f)
...     def wrapper(*args, **kwds):
...         print('Calling decorated function')
...         return f(*args, **kwds)
...     return wrapper
```

## 可以调用类里面方法的装饰器
只要在参数里面加上 `self`, 就可以使用类里面的方法了.
```python
from functools import wraps

>>> def my_decorator(f):
...     @wraps(f)
...     def wrapper(self, *args, **kwds):
...         print('Calling decorated function')
...         return f(*args, **kwds)
...     return wrapper
```