---
title: ubuntu13.04解决亮度无法保存
date: '2013/08/22 20:00:00'
tags:
  - ubuntu
  - 亮度
---

最近又把笔记本搞成ubuntu13.04了，除了安装amd官网的显卡驱动解决发热问题， 还把亮度不能保存问题给解决了。

1.按ctrl+alt+t可以唤出终端，在终端里输入

```
sudo gedit /etc/rc.local   
```

2.在 `exit 0` 前面输入下面的东东：

```
chmod a+w /sys/class/backlight/acpi_video0/brightness    
echo 80 > /sys/class/backlight/acpi_video0/brightness   
```

备注：

```
echo 80
```

这个 `80` 是根据

```
/sys/class/backlight/acpi_video0/max_brightness
```

这个最大亮度决定的，因为我的是100.
