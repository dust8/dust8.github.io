---
title: 我的静态博客生成器--lychee
date: '2015/08/29 20:00:00'
tags:
  - lychee
---

前段时间搞了个静态博客生成器.<br>
`lychee`, 源码: [github](https://github.com/dust8/lychee).<br>
代码很简陋,不过勉强可以用.

# 问题: 现在这类工具很多,为啥还要自己弄

我用python的,什么都想用python解决,可以少装一些软件

# 问题:也有python写得,为啥不用

用得库太多,兼容性差,和其他语言一样,安装时出现各种问题

--------------------------------------------------------------------------------

最主要的原因是,自己写一个可以学些东西,就算比较差,也很有成就感.

它是我的第一个发布到 `pypi` 得工具,可以通过 `pip` 安装.

以下是我的一些经验:

# 打包的时候, templates等文件没有打包进去

```
MANIFEST.in
recursive-include lychee/templates *
recursive-include lychee/posts *
recursive-include lychee/static *

setup.py
include_package_data=True    
```

# 如何复制包里的数据到其他地方

```
os.path.abspath(__file__)
```

# 如何获取现在命令行的路径

```
os.getcwd()
```

# 如何改变运行的路径

```
os.chdir()
```
