---
title: chrome 扩展开发问题
date: 2020-02-17 06:15:31
updated: 2020-02-28 06:15:31
tags:
  - chrome
  - 扩展
---

最近开发浏览器扩展碰到的问题记录下, 会不断更新. 详细的开发资料可以看下面的参考链接, 里面已经总结的很好了, 这里只记录下在实践开发中遇到的问题和解决办法.

建了个 QQ 群: 742072171 , 提供一个互相交流学习的地方.

## Content Scripts

### jsonp 请求回调未定义

出现这个问题是浏览器扩展安全限制, 目标页面和`content-script`不能共享 js, 所以我们只需要将回调函数注入到目标页面就可以了. 原理是 `content-script` 能操作目标页面的 `DOM`, 我们就可以通过 `js` 动态的插入 `script` 链接资源.

[web_accessible_resources 介绍](http://open.chrome.360.cn/extension_dev/manifest.html#web_accessible_resources).

```
# manifest.json

"web_accessible_resources": ["js/inject.js"]
```

```
# content.js

// 向页面注入JS文件
// <script src="js/inject.js"></script>
function injectCustomJs(jsPath)
{
    jsPath = jsPath || 'js/inject.js';
    var temp = document.createElement('script');
    temp.setAttribute('type', 'text/javascript');
    // 获得的地址类似：chrome-extension://ihcokhadfjfchaeagdoclpnjdiokfakg/js/inject.js
    temp.src = chrome.extension.getURL(jsPath);
    temp.onload = function()
    {
        // 放在页面不好看，执行完后移除掉
        this.parentNode.removeChild(this);
    };
    document.head.appendChild(temp);
}


injectCustomJs();

// jsonp 请求
$.ajax({
  url: "http://xxx.com/api",
  type: "post",
  dataType: "jsonp",
  jsonpCallback: "cb",
  data: {
    signType: "v3"
  },
  success: function(data) {
    console.log("success");
  }
});

```

```
# inject.js

// 回调函数
function cb(){
    console.log('jsonp 回调')
}
```

### content script css 文件引入图片资源

```
# content.css
background-image: url("chrome-extension://__MSG_@@extension_id__/images/rotate.png");
```

```
# mainfest.json
"web_accessible_resources": ["images/rotate.png"],
```

### 多个 content 分别在文档开始和结束注入

```
"content_scripts": [
    {
      "matches": ["http://www.google.com/*"],
      "js": ["start.js"],
      "run_at": "document_start"
    },
    {
      "matches": ["http://www.google.com/*"],
      "js": ["end.js"],
      "run_at": "document_end"
    }
 ]
```

## 存储相关

### chrome.storage 获取不到数据

原因是这些接口是异步的, 它有时还没取到数据就执行后面的脚本, 导致有时获取不到数据.  
方法一:回调函数

```
function read(key, callback) {
    if(key != null) {
        chrome.storage.local.get(key, function (obj) {
            callback(obj);
        });
    }
}

// Usage
read("test", function(val) {
  // val...
})
```

方法二:承诺

```
function read(key) {
    return new Promise((resolve, reject) => {
        if (key != null) {
            chrome.storage.local.get(key, function (obj) {
                resolve(obj);
            });
        } else {
            reject(null);
        }
    });
}

// 1. Classic usage
read('test')
    .then(function (val) {
        // val...
    })
    .catch(function () {
        // looks like key is null
    });

// 2. Use async/await
var val = await read(test);
console.log(val);
```

## chrome.tabs

### 扩展里面打开新标签页

除了 `windows.open` 还可以用扩展的接口

```
chrome.tabs.create(
  {
    url: "http://xxx.com",
    // 先不激活
    active: false
  },
  function(tab) {
    console.log("tab was created", tab);
    // 激活标签
    chrome.tabs.update(tab.id, { active: true });
  }
);
```

### 查询和更新关闭标签页

因为这些接口都是异步, 如果需要先判断在执行后面的操作, 最好用承诺来处理, 下面是截取的一部分代码, 不要照搬, 领会原理就好了

```
function query(urlWd) {
  return new Promise((resolve, reject) => {
    chrome.tabs.query({ url: "https://www.baidu.com/s?wd=*" }, function(tabs) {
      resolve(tabs, urlWd);

      //   else {
      //     reject(null);
      //   }
    });
  });
}

function update(tabs, urlWd) {
  return new Promise((resolve, reject) => {
    // chrome.tabs.update(tabs[0].id, { url: urlWd, active: true }, function(tab) {
    //   resolve(tab);
    // });
    [...tabs].forEach(tab => {
      chrome.tabs.remove(tab.id);
    });
  });
}

query(urlWd)
.then(tabs => {
  update(tabs);
})
.then(e => {
  gotoBaidu(info, tab);
});
```

## 其他

### 页面是 https 的，自己的接口是 http 的会报不安全错误

可以通过通信传给 `background` 调用接口

### 使用 eval

[Content Security Policy (CSP)](https://developer.chrome.com/extensions/contentSecurityPolicy)

```
# mainfest.json

"content_security_policy": "script-src 'self' https://example.com; object-src 'self'"
```

## 参考链接

- [What are extensions?(google 官方文档)](https://developer.chrome.com/extensions)
- [开发文档(360 的, 国内比较方便,但是不全)](http://open.chrome.360.cn/extension_dev/overview.html)
- [【干货】Chrome 插件(扩展)开发全攻略](https://www.cnblogs.com/liuxianan/p/chrome-plugin-develop.html)
- [Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
