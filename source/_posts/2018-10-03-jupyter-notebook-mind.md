---
title: dmind 一个jupyter notebook 思维导图插件
date: 2018-10-03 05:15:31
tags:
    - dmind
    - 思维导图
    - jupyter
---


以前用过 `幕布` 来生成思维导图，觉的很方便，不过免费的还是有限制。 这段时间想做个思维导图软件，可是搜了很久都没找到好的教程。没办法从头开始做，只能用别人的库来实现。比较有名的一点的是百度脑图开源的 [kityminder-core](https://github.com/fex-team/kityminder-core)，它支持  `json`, `text`, `markdown`，很满足需求。

我就写了个 `jupyter notebook` 的扩展 [dmind](https://github.com/dust8/dmind)。


## 示例
### text 格式
```
%%dmind text

DMind
    是一个 jupyter notebook 插件
    是一个思维导图插件
```
![text 格式](/assert/2018-10-03-1.png)


### markdown 格式, 逻辑结构图
```
%%dmind markdown right

# DMind使用文档
## 安装
### pip install dmind
## 使用
### 载入插件
#### %load_ext dmind
### 载入需要的附件
#### %dmindheader
### 渲染脑图
#### %%dmind datatype template theme 换行后输入内容
```
![markdown 格式, 逻辑结构图](/assert/2018-10-03-2.png)

### json 格式 , 目录组织图, 文艺绿
```
%%dmind json filetree fresh-green

{
    "root": {
        "data": {
            "text": "Dmind参数说明"
        },
        "children": [
...
```
![json 格式 , 目录组织图, 文艺绿](/assert/2018-10-03-3.png)

## 碰到的问题
### 主题和模板设置无效
去看完整的百度脑图实现，用 `setTimeout` 解决

### 一个页面多个脑图
只能生成多个脑图对象来渲染

### 在 notebook 上显示太小
只能给 `km-view` 类设置固定大小 `400px`


## 引用
- [Hexo中使用markdown来绘制脑图（mind map）](https://qsli.github.io/2017/01/01/markdown-mindmap/)
- [Hexo 渲染思维导图](https://hargao.top/2018/07/24/hexo-mindmap.html)
- [Creating an IPython extension with custom magic commands](https://ipython-books.github.io/14-creating-an-ipython-extension-with-custom-magic-commands/)
- [IPython extensions](https://ipython.readthedocs.io/en/stable/config/extensions/index.html)