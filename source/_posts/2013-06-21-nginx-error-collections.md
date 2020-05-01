---
title: nginx错误收集
date: "2013/6/21 20:00:00"
tags:
  - nginx
---

# 错误 1\. unknown directive "if(\$query_string)"

在 vps 上部署 Tornado 是按照官网的[介绍](http://www.tornadoweb.org/en/stable/overview.html#running-tornado-in-production)部署的，可是提示出错了。

出错信息为：_[emerg]: unknown directive "if(\$query_string)"_。

google 了半天，在[狂奔的鹿](http://dynamiclu.iteye.com/1341469)这里找到了原因。

原来是在 _if_ 和 _(_ 之间一定要有空格。
