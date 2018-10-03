---
title: 更改android手机的物理地址等等
date: '2013/06/27 20:00:00'
tags:
  - android
---

有些人总有些不一样的需求，更改android手机的物理地址相信很多人都需要。<br>
[dust8](http://www.dust8.com) 也是有这需求的。

# 1\. 环境、工具

手机root， 终端模拟器（运行脚本用）， bosybox（增强脚本用）

# 2\. 改配置的文件

文件命名为wl.sh, 放在内存卡根目录。

```
#！/system/bin/sh
# 这是改手机名，默认一般是 android_********
setprop net.hostname PC20130627

# 这是把wifi关掉。 wlan0是wifi的代号
busybox ifconfig "wlan0" down

# 这是改手机的ip地址
busybox ifconfig "wlan0" 192.168.1.55

# 这是改手机的物理地址
busybox ifconfig "wlan0" hw ether 00:00:00:00:00:00

# 这是打开wifi
busybox ifconfig "wlan0" up    
```

# 3\. 运行配置

打开wifi， 连上信号， 打开终端，输入

```
>su  
>cd sdcard  
>source wl.sh  
```

如果说source not find, 可以换成 `sh wl.sh`

## 总结：

为啥要用busybox，是因为手机内置的命令功能不强。ifconfig命令基本无法使用。网上的教程很少，并且很零碎，现在总结出来，让你们释放你们的黑暗面...
