---
title: 正则顶级域名
date: '2013/12/17 20:00:00'
tags:
  - 正则
---

一直想从网页中提取所有的 **顶级域名**。<br>
我想要的 **顶级域名** 像这样的 "[http://www.dust8.com"](http://www.dust8.com”),<br>
而 "<http://www.dust8.com/archive>" 这类不要。

最近找到一个正则表达式还行。就是

```
http://(\w+\.){1,3}\w+    
```

效果的话可以去找一些在线的正则表达式网站查看。

```
http://  就只是代表这几个字符
\w       是匹配任何字母数字字符；它相当于类 [a-zA-Z0-9]。
\.       是匹配 "." 这个字符； "\"是用来取消 "."的特殊意义的，
         因为 "."用来匹配任意除换行符"\n"外的所有字符的    
｛1，3｝  是表示匹配前面的字符1到3次
\w+      表示匹配 [a-zA-Z0-9] 1次或无限次    
```

现在https比较多了，听说可以防止偷窥，传输数据更安全。<br>
所有前面的正则表达式可以改为：

```
(http|https)://(\w+\.){1,3}\w+    
| 代表它左右的表达式任意匹配一个    
```

这样还不行，还是会把类似 "<http://www.dust8.com/archive>" 这样的匹配出来， 改为这样：

```
(http|https)://(\w+\.){1,3}\w+$
$ 匹配字符串末尾    
```

已经不错了。解决了上面这个问题。但是有的网页的网址是这样的：<br>
"<http://www.dust8.com/>", 这类的匹配不了，再改：

```
    (http|https)://(\w+\.){1,3}(\w+|\w+/)$    
```

现在基本已经没发现什么问题，发现了在改。

下面是我用 python3 写的一个获取一个网址内所有的顶级域名代码：

```
import re
import urllib.request

from html.parser import HTMLParser


class YHTMLParser(HTMLParser):
    def __init__(self):
        HTMLParser.__init__(self)
        self.urls = []
        self.flag = False
        self.pattern = re.compile("(http|https)://(\w+\.){1,3}(\w+|\w+/)$")

    def handle_starttag(self, tag, attrs):
        if tag == "a":
            for attr in attrs:
                if attr[0] == "href":
                    m = self.pattern.match(attr[1])
                    if m:
                        u = attr[1]
                        if u[-1] == "/":
                            u = attr[1][:-1]
                        if len(re.findall("\.", u)) == 1:
                            s = u.split("//")
                            u = s[0] + "//www." + s[1]

                        self.urls.append(u)

    def getUrls(self):
        return list(set(self.urls))


class UrlParser():
    def __init__(self):
        self.html = ""
        self.urls = []

    def getHtml(self, url):
        req = urllib.request.Request(url)
        req.add_header('User-Agent', 'Mozilla/5.0 (Windows NT 6.3; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/31.0.1650.63 Safari/537.36')
        try:
            f = urllib.request.urlopen(req)
            self.html = f.read().decode()
        except:
            self.html = ""

        self.parserHtml()

    def parserHtml(self):
        yhtmlparser = YHTMLParser()
        yhtmlparser.feed(self.html)
        self.urls = yhtmlparser.getUrls()

    def getUrls(self):
        return self.urls


if __name__ == "__main__":
    url = "http://www.the5fire.com/links/"
    urlparser = UrlParser()
    urlparser.getHtml(url)
    u = urlparser.getUrls()
    print("find", len(u), "links\n", u)
```
