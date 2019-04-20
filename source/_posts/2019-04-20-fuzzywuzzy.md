---
title: fuzzywuzzy字符串模糊匹配
date: 2019-04-20 05:15:31
tags:
  - fuzzywuzzy
  - 字符串匹配
---

前段时间碰到根据法院名称到数据库找省市县的问题。数据库里面的法院名字是
标准的格式，但给的搜索名字是不标准的，会少一些字，少的字位置也不确定
没办法用数据库的模糊搜索。后来发现个库很好用叫
[fuzzywuzzy](https://github.com/seatgeek/fuzzywuzzy)，它可以从备选
的列表中选出最接近的一个。

## 代码

```python
from fuzzywuzzy import process

courts = ['北京市第二中级人民法院','北京市第三中级人民法院','北京市石景山区人民法院']

print(process.extractOne('北京第三中院', courts)[0])
print(process.extractOne('石景山区法院', courts)[0])

```

## 输出

```
北京市第三中级人民法院
北京市石景山区人民法院
```
