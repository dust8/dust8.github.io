---
title: csgo个人训练命令
date: 2016-11-04 05:15:31
tags:
  - csgo
  - 命令
---

看csgo的视频, 有很有趣的做法.比如可以看到弹点,看到雷的飞行轨迹,自己还可以飞.
这样练枪和练扔雷很有帮助.下面就说说这些是怎么设置的.

## 启用开发者控制台   

设置==>游戏设置==>启用开发者控制台==>是  

## 启动控制台    

在游戏中按 `~` 启动控制台,然后就可以输入下面的命令.
下面这些命令必须是在离线游戏中才能用的.

## 命令     
### 必须命令   

    sv_cheats 1 作弊

### 常用命令    

    sv_infinite_ammo 1无限弹药

    noclip 穿墙.建议使用bind t "noclip" 绑定按键,则按t键为开启穿墙模式

    sv_showimpacts 1 显示弹痕,能清晰了解弹道分布

    sv_grenade_trajectory "1" 打开手雷飞行轨迹
    sv_grenade_trajectory_time "10" 设置手榴弹飞行轨迹的保留时间

    bot_kick 踢出所有NPC
