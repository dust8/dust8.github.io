---
title: Windows Terminal 使用及美化
date: 2020-07-03 05:15:31
tags:
  - Windows Terminal
---

## 安装软件

[PowerShell Core](https://github.com/PowerShell/PowerShell)

[oh-my-posh](https://github.com/JanDeDobbeleer/oh-my-posh)

## 修改配置

notepad.exe \$Profile

```
Import-Module posh-git
Import-Module oh-my-posh
Set-Theme Paradox

Set-PSReadlineKeyHandler -Key Tab -Function Complete # 设置 Tab 键补全
Set-PSReadLineKeyHandler -Key "Ctrl+d" -Function MenuComplete # 设置 Ctrl+d 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo # 设置 Ctrl+z 为撤销
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward # 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward # 设置向下键为前向搜索历史纪录
```

## 参考链接

- [Windows Terminal 使用及美化](https://www.antmoe.com/posts/614360dd/index.html)
