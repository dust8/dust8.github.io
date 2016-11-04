---
title: flask和mongodb建站总结
date: '2014/07/15 20:00:00'
tags:
  - flask
  - mongodb
---

最近在用flask＋mongodb做一个论坛，做了很久。有些体会和大家分享下。

1. 学东西一定要仔细看文档，多看几遍；很多东西其实在文档里已经说明了的，<br>
  不需要在去搜，而且不一定能搜到。
2. 有些事去做的时候一定要专注。
3. 有些时候人很容易钻牛角尖，可以试试放段时间再去解决。

# Flask-MongoEngine

--------------------------------------------------------------------------------

文档在<http://flask-mongoengine.readthedocs.org/en/latest/>。

使用分页的时候报 xrange 错误，是因为在python3里面已经没有这个内置函数了，<br>
可以把python里面的site－packages/flask_mongoengine/pagination.py里面的109行<br>
的xrange改为range就可以了。

# Flask-Login

--------------------------------------------------------------------------------

文档在<http://flask-login.readthedocs.org/en/latest/>，<br>
这里有个中文翻译<https://github.com/yinian1992/flask-login-docs-cn/blob/master/index.rst>。

在登录限制里和mongodb用时可以这样用，放在了models.py里

```
    @login_manager.user_loader
    def load_user(userid):
        if userid is None:
            return None
        return User.objects(id=userid).first()    
```

User是用户类，继承了flask.ext.login.UserMixin和flask.ext.mongoengine.MongoEngine的Document。<br>
在视图里

```
    from flask.ext.login import login_user
    login_user(user)
```

函数里的user就是一个User类的实例。

# jinja

--------------------------------------------------------------------------------

flask默认的模板引擎是jinja。<br>
我有个回复的list列表，要在模板里列出他的楼数。<br>
python里可以这样：

```
    l = ['a', 's']
    for i, b in enumerate(l):
        print(i, b)
```

就会输出

```
    0 s
    1 d
```

但是在模板里不方便这样。可以看jinja的文档 `for` <http://jinja.pocoo.org/docs/templates/#for>。<br>
用 `variable` 里的 `loop.index` 实现 以 `1` 开始的索引数。

# flask-mail

--------------------------------------------------------------------------------

官方文档：<http://pythonhosted.org/flask-mail/><br>
下面是用126邮箱的例子，![126的文档](http://img4.cache.netease.com/help/2011/2/1/20110201095104ff8bc.png)

```
    # email server
    MAIL_SERVER = 'smtp.126.com'
    MAIL_PORT = 465
    MAIL_USE_TLS = False
    MAIL_USE_SSL = True
    MAIL_USERNAME = 邮箱用户名
    MAIL_PASSWORD = 邮箱密码
```

是用app.config来配置，app＝Flask(**name**)<br>
文档有的就不说了，我来说说我碰到的问题。

1.说不在上下文中<br>
需要 `@copy_current_request_context` ,<br>
从 `from flask import copy_current_request_context` 引入。 而且还必须在路由视图里才有效，不然还会报其他错误。

2.smtplib报encode('ascii')错<br>
把764行的 `msg = _fix_eols(msg).encode('ascii')` 改为<br>
`msg = _fix_eols(msg).encode()` , 他在python安装目录里面。<br>
具体看他报错的信息吧。
