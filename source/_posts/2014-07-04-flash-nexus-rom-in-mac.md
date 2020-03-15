---
title: mac下刷nexus官方包
date: "2014/07/04 20:00:00"
tags:
  - mac
  - nexus
---

最近换到 mac 下了，给 nexus 刷机记录下。

# 1.下载安装 fastboot 和 adb

[task－adb－fastboot.zip](/blog/assert/2014-07-04-task-adb-fastboot.zip)或者去<http://bbs.gfan.com/android-3945571-1-1.html>下载。

把它解压出来有 4 个文件：adb－mac，adb－ubuntu，fastboot－mac，fastboot－ubuntu。

把 fastboot-mac 复制到/usr/local/bin 下并改名为 fastboot，在 task－adb－fastboot 的目录里，运行：

```
    ＃sudo cp fastboot-mac /usr/local/bin/fastboot
```

把 adb-mac 复制到/usr/local/bin 下并改名为 adb，在 task－adb－fastboot 的目录里，运行：

```
    ＃sudo cp adb-mac /usr/local/bin/adb
```

# 2.下载 nexus 官方包

<https://developers.google.com/android/nexus/images><br>
找到对应的包，下载下来并把它解压出来。

# 3.刷机

1）按住音量下不放，同时按住电源键，进入了 bootload<br>
2）当然是连上电脑<br>
3）打开终端，进入 nexus 官方包解压出来的目录里，输入

```
    ＃./flash-all.sh
```

不用管它，等他刷完就可以了。

# 4.清空数据

1）开机会卡在开机画面，按住电源键不放，等它关机。<br>
2）按住音量下不放，同时按住电源键，进入了 bootload<br>
3）按音量键选择 Recovery mode, 在按电源一下<br>
4）会出现有感叹号的倒地小绿人，在按下音量上键，就会进入 recovery<br>
5）双清，用音量键选 wipe data／factory reset，在按电源，等它清完；<br>
在选 wipe cache partition，在按电源，等它清完；<br>
在选 reboot system now，在按电源，就会重启了。第一次比较慢，要好几分钟才进得去。
