---
title: chrome与xpath

date: 2017-03-07 05:15:31
tags:
  - xpath
---

## 前言

平常喜欢写爬虫，写爬虫肯定要提取信息，如果用 **正则表达式** 就太麻烦了，所以很多库都
集成了 **xpath** 选择器。如果不太熟练或者想测试 **xpath** 的话就有点不方便，
网上没有比较好的插件或工具。其实 **chrome** 里面的开发工具就可以很好的导出和测试 **xpath**。

## chrome 开发工具

打开 chrome 开发工具有 3 种方式

- 在页面右键 > inspect
- 右上角设置 > More Tools > Developer Tools
- 快捷键
  - macos: option(alt)+command+i

## 提取 xpath

在 **Elements** 里面选择要提取的元素，右键 Copy > Copy Xpath。这种方式虽然方便，还是有很多不足。
例如层级太深，只能选一个元素，这样兼容性不太好，网站稍微改一下就不行了，需要自己改动下，通过类名或者 id 等等来提高。

![export-xpath](/blog/assert/2017-03-07-export-xpath.gif)

## 测试 xpath

在 **Console** 里面用 **\$x(path)** 来测试是否可以提取到元素。  
[\$x(path)](https://developers.google.com/web/tools/chrome-devtools/console/command-line-reference?utm_source=dcc&utm_medium=redirect&utm_campaign=2016q3#xpath)  
返回一个与给定 XPath 表达式匹配的 DOM 元素数组。
记得这里只能提取元素，不能提取元素的属性。

![test-xpath](/blog/assert/2017-03-07-test-xpath.png)
