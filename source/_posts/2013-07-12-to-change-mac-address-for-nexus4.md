---
title: nexus 4更改物理地址
date: '2013/07/12 20:00:00'
tags:
  - nexus
  - macaddress
---

刚入手几天 nexus 4，分享一点 nexus 4的使用心得。

# 1.刷回官方rom的问题

网上有些教程，没说完，导致刷完机后开在 `x` 屏动不了。这个问题是刷完机没有双wipe。步骤如下：<br>
（1）按住电源键，维持10s，将手机强制关机，然后还是按住音量下键不放，开机，双手维持大约3s后进入倒地绿色机器人界面。<br>
（2）按音量键上下切换到"Recovery mode"，然后按电源键确认，手机上会出现一个较小的倒地绿色机器人，肚子上有个叹号警示标志。<br>
（3）按住电源键不放，然后快速地按一下音量键上键松开，进入"Android system recovery <3e>"界面，有蓝色小字菜单。<br>
（4）按音量键上下选择"wipe data/factory reset"，按电源键确认，然后选择"yes -- delete all user data"，再确认，等出现黄色小字"Data wipe complete.（清除数据完成）"后，再确认"reboot system now（重启系统）"。

# 2.官方rom如何改物理地址

我以前的那款机可以按照 [更改android手机的物理地址等等](http://blog.dust8.com/p/2) 这样更改。但是现在在nexus 4的官方rom上不行，cm10.1也不行。google一圈，有的说改"WCNSS_qcom_cfg.ini"和"WCNSS_qcom_wlan_nv.bin"。我试过有的路由器可以用，有的不行，而且重启就没了，又要重新替换文件，比较麻烦。经过我 [dust8](http://www.dust8.com) 的几天摸索，终于发现一个比较简单的方法。

手机必须root，且有re文件管理器这类东西。<br>
物理地址放在 `/persist/wifi/.macaddr` 这里，只要更改这个文件，就可以改物理地址。改完在重启手机，你就发现物理地址就是你改的那个。一次更改，就可以了一直用了。
