---
title: 利用bat命令来标示不同的python运行实例
date: 2018-03-18 05:15:31
---

## 问题

由于写的爬虫需要运行好几个相同的实例,并且运行的文件规定为 `main.py`,导致运行起来后都分不清谁是谁了.

## 写 bat 命令来标示

同事告诉我的,我又修改了下,代码如下,保存为 `main.bat` 就好了.

```bat
@echo off

echo 设置自定义标题用来区分不同的运行实例, 运行的文件名默认为 main.py, 目录为运行的 bat 文件目录

set default_title=test
set dafault_title_prefix=spider
set default_filename=main.py
set default_path=%~dp0
set default_cmd=python

title %dafault_title_prefix%-%default_title%
set fullname=%default_path%%default_filename%
set run_cmd=%default_cmd% %fullname%

echo 运行命令为: %run_cmd%

Rem 运行命令
%run_cmd%

pause
```

![main.bat](/assert/2018-03-18.png)

备注:

- 等号前后不要有空格
- 加个暂停可以防止出错后窗口关闭后看不到出错信息
