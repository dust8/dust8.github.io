---
title: nginx tornado的静态文件处理
date: '2013/10/09 20:00:00'
tags:
  - nginx
  - tornado
---

昨天把这个博客程序更新了下，居然出现静态文件未发现。<br>
本地是没问题的，在vps里就有问题了。<br>
在浏览器里查看源代码，点击css之类的会出现**ngixn**相关的<br>
字样，应该是它没分配过来。但是我关于这部分的代码都没动过，<br>
没理由更新前可以，更新后不行。<br>
早上终于发现了一篇文章解决了这个问题<br>
[Nginx Tornado 站点配置遇到的一个问题，解决后我只想说卧槽](http://www.luoyuncloud.com/forum/topic/view?id=509)。

只要把 **nginx** 里关于静态文件的指派删除就好了。<br>
就是那段

```
location ^~ /static/{}    
```

重启 **nginx**，就好了，很无语。
