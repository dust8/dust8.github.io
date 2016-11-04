---
title: ubuntu去除more-suggestions
date: '2013/10/08 20:00:00'
tags:
  - ubuntu
  - suggestions
---

ubuntu应该是最易用的桌面linux系统。<br>
而现在unity是ubuntu默认的桌面。<br>
unity里有个 **more suggestions** ，非常让人讨厌。就是这样的

![unity-more-suggestions](/static/img/unity-more-suggestions.png)

想要去掉它，可以用 **dconf Editor** 修改设置。<br>
**dconf Editor** 一般随系统就安装了，如果没有安装，就去软件中心安装。

修改的地方是 **desktop >> unity >> lenses >> applications**。<br>
把**applications** 里的 **display-available-apps**的勾去掉就ok了。<br>
如图：

![display-available-apps](/static/img/display-available-apps.png)

最后的效果如下：

![unity-no-more-suggestions](/static/img/unity-no-more-suggestions.png)

强迫症，没办法。^_^。
