---
title: 第一次编程工作面试

date: 2017-03-15 05:15:31
---

## 前言
想转行做编程工作，平时自学python，玩过不少东西，但掌握的少之又少，看来还是要专一门
学精一些。网上投了不少简历，有一半都根本看都没看过简历，剩下的都是回复不合适，只有
昨天有一家给了面试机会，自己掌握不精，平时没怎么记，习惯用自动补全，有点丢人啊。还被
高德坑了下，说是一个半小时的路程，光在车上就花费了两个半小时。

## 面试题
### 问怎么知道python安装没有，还有就是怎么看python的版本号
这个简直是送分题
```bash
which python
python --version
```

### 给‘xxx xx xxxx’让提取数字并从大到小排序
```python
line = 'dust dust8  1234 45 90'

def filter_nums(line):
    str_nums = line.split()
    nums = [int(num) for num in str_nums if num.isnumeric()]
    nums.sort(reverse=True)
    return nums

filter_nums(line)
```

这里我 **isnumeric** 没写出来，只知道有这么个函数可以判断是否是数字，还有就是漏了
**int** 会导致排序不正确。早知道用正则好了。

### 就是考察django的数据库查询，有2个限制条件
我没怎么看django，上午在吊水，下午复习了下python基本知识，晚上只看了几章django。
导致没写出来。回来看了下有3种方法。

- `filter(quantity__gt=5, price__lt=10)`
- `filter(quantity__gt=5).filter(price__lt=10)`
- `filter(Q(quantity__gt=5)&Q(price__lt=10))`

### 叫用django的内置tags来写个表格
django的内置tags有哪些我还没看过，就用html写了下
```html
<table>
  <tr><th rowspan="2">订单</th><th colspan="3">流程</th></tr>
  <tr><th>A</th><th>B</th><th>C</th></tr>
</table>
```
这里我知道有个属性可以控制它的占用的空间，也没写对，就是**rowspan** **colspan**。
还有就是django内置的tags，我就写了
```jinja
{% for x in xx %}
    <tr><td>{{ x.x }}</td><td>{{ x.y }}</td></tr>
{% endfor %}
```
其实 **for** 就是django内置的一种tag，可惜我有点紧张，变量好像还忘记变量加双大括号。

### 有个思考题，说怎么测量路上的通车数量
回答的不是很好，写了射线类测量，人类计数，图像识别，还要思考他们的准确性和成本。

## 总结
不知道这几道题目是否是为我特意出的，感觉很简单，只是平时只是图好玩，没怎么记，只是记住了
流程，可以这样做，做的时候查下文档就可以弄出来了。看来平时要多记啊。

学历不高，年纪稍大，没有经验，总是追求新技术，导致什么都知道一些，做的时候要复习下，
又没有专精哪一门，不知道谁能收留下，我应该可以很快进入工作状态。
