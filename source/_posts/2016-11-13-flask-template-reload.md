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


还有就是静态文件被缓存了,例如 `css`, `js` 这些文件改动后，刷新网页也看不到改动后的效果.   
可以在文件名后面加入随机字符, 可以看看 [static url cache buster](http://flask.pocoo.org/snippets/40/).    

    @app.context_processor
    def override_url_for():
        return dict(url_for=dated_url_for)

    def dated_url_for(endpoint, **values):
        if endpoint == 'static':
            filename = values.get('filename', None)
            if filename:
                file_path = os.path.join(app.root_path,
                                         endpoint, filename)
                values['q'] = int(os.stat(file_path).st_mtime)
        return url_for(endpoint, **values)
