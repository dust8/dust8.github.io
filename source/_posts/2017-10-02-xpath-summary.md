---
title: xpath总结
date: 2017-10-02 05:15:31
tags:
  - xpath
---

# 怎么调试 xpath 的

不喜欢用正则，感觉用有时用 xpath 简单很多。  
![](/assert/2017-10-02.png)

# 用到的一些 xpath 表达式

- 'a/text()' 获取 a 元素的内容
- '//tag[@attr0][@attr1]'

## 语法

### 选取节点

- '//a/@href' 获取所有 a 元素的 href 属性值
- '//a/..' 所有 a 元素的父元素

### 谓语

- '//tr/td[1]' 获取所有 tr 里面的第一个 td 元素，索引从 1 开始
- '//li[@class="p2"]' 选择所有类名为 p2 的 li 元素
- '//tr[td[@style]]' 包含有 style 属性的 td 元素的所有 tr 元素
- 'li[a]' li 元素下有 a 元素的 li 元素

### 选取若干路径

- 'a|h2' a 元素或者 h2 元素

### 轴

- '//a/following-sibling::div[1]' a 元素的第一个 div 兄弟元素
- '//br/preceding-sibling::text()' br 元素之前的所有同级的 text 节点.有些页面全是 br 元素换行,用它就可以取到和它同级的内容
- '//a/ancestor::div[1]' a 元素的第一个 div 父元素

### 运算符

- 'li[position() mod 2 = 0]' 所有偶数位的 li 元素
- 'li[position() mod 2 = 1]' 所有基数位的 li 元素
- '//tr[len(td)>1]' 获取有 1 个以上 td 元素的所有 tr 元素
- 'li[a or h2]' li 元素下有 a 元素或者 h2 元素的 li 元素
- 'a[text()="下一页"]' text 为下一页的 a 元素

### 函数

#### 有关字符串的函数

- 'string(.)' 获取当前元素下面所有的内容
- 'string(//a)' 获取 a 元素的所有文本(只能获取第一个元素的所有文本)
- '//tr[contains(@class,"tr")]' 选择所有类名包含 tr 的 tr 元素
- '//div[@id="pagination"]/span[contains(text(),"共")]' 获取包含 '共' 的 span 元素(用 starts-with 更好).我用来获取总页数(共 666 页)
- '//span[starts-with(@class,"title")]' css 类名以 title 开头的 span 元素
- '//p[starts-with(a,"亲")]' 包含以 '亲' 开头的 a 元素的所有 p 元素
- '//span[ends-with(@class,"title")]' css 类名以 title 结尾的 span 元素
- 'substring(//span[@class="bugs-time"]/text(),6)'
- 'normalize-space(//td[starts-with(text(),"手")]/following-sibling::td[1])'
- '//div[normalize-space(string(.))="工作经历"]'

#### 关于布尔值的函数

- '//ul[not(@class)]' 选择所有没有类属性的 ul 元素

#### 合计函数

- 'count(//a)' 计算 a 元素的数量.可以用来统计一页有多少条数据
- '//tr[count(td)=4]' 只有只有 4ge 个 td 元素的 tr 元素

#### 上下文函数

- '//tr[position() > 1]' 获取除了第一个 tr 元素之外的所有 tr 元素.有时第一个 tr 元素是抬头标签,不是数据
- '//a[last()]' 最后一个 a 元素.可以用来获取分页里面的最后一页,或者下一页
