---
title: flask模版重载
date: 2016-11-13 05:15:31
tags:
  - flask
  - template
  - reload
---

以前只要设置 `app.run(debug=True)` 就可以重载模版文件了.    
`flask 0.11` 改动了设置, 这样只能重载 `python` 文件, 不能连模版文件一起重载.    
要想模版文件一起重载需要设置 `TEMPLATES_AUTO_RELOAD` 这个配置([Flask Changelog](http://flask.pocoo.org/docs/0.11/changelog/#version-0-11)).    
例如:    

    app.config['TEMPLATES_AUTO_RELOAD'] = True


但是使用后还发现如果是 `include` 进来的模版改动后还是不能重载.   
必须要这样设置才可以:    

    app.config['DEBUG'] = True

所以推荐使用配置文件的形式引入配置:    

    config.py
    DEBUG = True
    TEMPLATES_AUTO_RELOADT = True


    app.py
    app.config.from_pyfile('config.py')
    app.run()
