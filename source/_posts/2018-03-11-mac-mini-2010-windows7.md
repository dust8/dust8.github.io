---
title: mac mini 2010 款安装 windows 7
date: 2018-03-11 05:15:31
---

## 问题: Boot Camp 助手 里面提升 "找不到安装器光盘"
2010款的只支持到 windows7,并且默认只支持光盘安装.   
所以需要修改系统配置,使他支持 u 盘安装.    

应用程序 => 实用工具 => Boot Camp 助理,右键 => 显示包内容 => 打开Contents文件夹, 找到 Info.plist 文件.复制到桌面,多复制几个.
把其中一个的后缀改为 txt 文件,在修改里面的文件内容.  

1. 在 DARequiredROMVersions 字段添加 Boot ROM 版本号
2. 在 PreUSBBootSupportedModels 字段添加型号标识符
3. 修改 PreUSBBootSupportedModels 为 USBBootSupportedModels,保存并替换回原位
4. 打开终端，输入 sudo codesign -fs - /Applications/Utilities/Boot\ Camp\ Assistant.app/, 输入密码,完成


现在再打开 Boot Camp就可以了 ,如果没有第 4 步 打开会报错的. 

## 问题: 分不了区
开机按住 command+s, 等到可以输入命令后, 输入 `fsck -fy` , 
等它执行完毕, 然后输入 `reboot` 重启就好了.