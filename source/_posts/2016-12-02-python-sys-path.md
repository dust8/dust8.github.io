---
title: python sys.path 的妙用
date: 2016-12-02 05:15:31
tags:
  - python
  - sys.path
---

## sys.path是什么
`[sys.path](https://docs.python.org/3/library/sys.html#sys.path)` 是一个字符串列表,用来指定搜索模块的路径.每次调用解析器都会初始化它,所以不用担心改变它会污染环境.

## sphinx里面的用法
`sphinx` 里使用 `autodoc` 或者 `sphinx-apidoc`, 必须要引入包.
一种方法就是先安装包,另外一种就是在 `conf.py` 文件里面写入下面的内容:    

    import os
    import sys
    sys.path.insert(0, os.path.abspath('..'))

这样就可以引入包了,而退出之后又不会影响环境.    
这里为啥是 `..`,因为我的项目目录是这样的:    

    .
    ├── bencoding
    │   ├── __init__.py
    │   ├── decoder.py
    │   ├── encoder.py
    │   └── tokens.py
    ├── docs
    │   ├── Makefile
    │   ├── conf.py
    │   ├── index.rst
    │   ├── install.rst
    │   ├── make.bat
    │   ├── module.rst
    │   └── quickstart.rst
    ├── setup.py
    └── tests
        ├── __init__.py
        ├── context.py
        ├── test_decoder.py
        └── test_encoder.py

## 单元测试的用法
在 《The Hitchhiker’s Guide to Python》里的 [Test-Suite](http://docs.python-guide.org/en/latest/writing/structure/#test-suite) 有提到怎么优雅的引入包.
先新建文件 `tests/context.py` :    

    import os
    import sys
    sys.path.insert(0, os.path.abspath('..'))

    import bencoding # bencoding 就是你的包

然后就可以在测试文件里引入     

    from .context import bencoding

如果你的包层级太多可以这样    

    from .context import bencoding
    from bencoding.decoder import Decoder

这样就完美了.
