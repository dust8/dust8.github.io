---
title: 我的js格式化时间戳库timejs
date: '2015/02/03 20:00:00'
tags:
  - js
  - timejs
  - 时间
---

网上很多地方都有格式化时间，例如：<br>
`刚刚`， `10分钟前`， `3天前`，`2个月前` 之类的。

当然，网上有很多处理时间的库，像 `momentjs`。

不过我喜欢造轮子，造轮子也是应该提倡的；<br>
不然总是用别人造的轮子，万一人家不造了，不是惨了。

github：[timejs](https://github.com/dust8/timejs)

# 1\. timejs 源码

这是我的第一个 js 库: `timejs`。它是一个格式化时间戳的库。代码如下：

```
(function() {
    var Time = Time || {};

    Time.getTime = function(){
        var d = new Date();
        return d.getTime()
    };

    Time.fromNow = function(time){
        // time必须是毫秒
        var curTime, diff;

        curTime = Time.getTime();
        diff = curTime - time;

        if (0 > diff){
            return "出错了";
        }else if (1000 * 60 > diff) {
            return "刚刚";
        } else if (1000 * 60 <= diff && 1000 * 60 * 60 > diff) {
            return parseInt(diff / (1000 * 60)) + "分钟前";
        } else if (1000 * 60 * 60 <= diff && 1000 * 60 * 60 * 24 > diff) {
            return parseInt(diff / (1000 * 60 * 60)) + "小时前";
        } else if (1000 * 60 * 60 * 24 <= diff && 1000 * 60 * 60 * 24 * 30 > diff) {
            return parseInt(diff / (1000 * 60 * 60 * 24)) + "天前";
        } else if (1000 * 60 * 60 * 24 * 30 <= diff && 1000 * 60 * 60 * 24 * 30 * 12 > diff) {
            return parseInt(diff / (1000 * 60 * 60 * 24 * 30)) + "月前";
        } else {
            return parseInt(diff / (1000 * 60 * 60 * 24 * 30 * 12)) + "年前";
        }

    };

    // 注册命名空间到window对象上
    window["Time"] = Time;
})();    
```

# 2\. timejs 用法

## 2.1 代码

```
<!DOCTYPE html>
<html lang="zh">
  <head>
    <meta charset="UTF-8">
    <title>time.js用法</title>
  </head>
  <body>

    <script src="time.js"></script>
    <script type="text/javascript">
        var time = 1422875526971;
        document.write("Time.fromNow(time) <br/>")
        document.write("参数 time: 必须是毫秒.<br/>")
        document.write("1422875526971 => " + Time.fromNow(time) );
    </script>
  </body>
</html>    
```

## 2.2 输出

```
Time.fromNow(time)
参数 time: 必须是毫秒.
1422875526971 => 1天前   
```

参考资料：

1. [JS---创建自己的"JavaScript库"，原来如此简单](http://blog.csdn.net/mazhaojuan/article/details/7659906)
2. [binnng/time.js](https://github.com/binnng/time.js)
