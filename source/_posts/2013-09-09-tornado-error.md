---
title: tornado自定义错误页
date: '2013/09/09 20:00:00'
tags:
  - tornado
---

tornado 自定义错误页， 网上有好几种方法， 都不是很好。<br>
我发现还可以这样改， 如下：

```
import tornado.ioloop
import tornado.web

class BaseHandler(tornado.web.RequestHandler):
    def write_error(self, status_code, **kwargs):
        self.finish("this is my 404!")

class MainHandler(BaseHandler):
    def get(self):
        self.write("Hello, world")

application = tornado.web.Application([
    (r"/", MainHandler),
])

if __name__ == "__main__":
    application.listen(8888)
    tornado.ioloop.IOLoop.instance().start()  
```

就是重载 `tornado.web.RequestHandler`, 重写 `write_error` 方法。<br>
然后根据 `status_code` 的状态码给出不同的响应。<br>
不过 `finish` 有效一点， 那些 `render` 和 `redirect` 都有些问题。
