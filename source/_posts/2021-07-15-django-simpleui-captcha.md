---
title: 从问题到开源之django-simpleui-captcha项目
date: 2021-07-15 05:15:31
tags:
  - django
  - django-admin
  - django-simpleui-captcha
---


## 问题
`django` 自带的后台管理比较丑, 就使用了 [simpleui](https://github.com/newpanjing/simpleui) 的主题. 为了后台的安全, 需要增加登录验证码.网上有些零散的教程,实际用起来非常不方便.就想把该功能做出一个可复用的`django app`, 方便后续使用.开源出来,也方便大家使用.

## 成果
[django-simpleui-captcha](https://github.com/dust8/django-simpleui-captcha)  是一个 `django` 后台管理登录验证码插件.

### 界面
![screenshoot1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2243a404d47a44b5ac073ea7dae90707~tplv-k3u1fbpfcp-watermark.image)

### 安装
```bash
pip install django-simpleui-captcha
```

### 快速开始
#### 1. 添加 "simpleui_captcha" 到 INSTALLED_APPS 设置, 注意要放在最前面
```py
INSTALLED_APPS = [
    "simpleui_captcha",
    "simpleui",
    ... 
]
```

#### 2. 添加 `simpleui_captcha` 的 `url` 到你的项目 `urls.py` ::
```py
path('simpleui_captcha/', include('simpleui_captcha.urls')),
```

#### 3. 运行 `python manage.py migrate` 迁移验证码模型

## 开源过程
### 起名
基于`django` web框架, 用的是`simpleui` 主题, 实现验证码功能, 故而叫`django-simpleui-captcha`, 让人一看名字就知道是做什么的,简单明了.

### 实现思路
站在巨人的肩膀上,直接集成 [django-simple-captcha](https://github.com/mbi/django-simple-captcha),添加验证码到登录表单,在修改成`simpleui`的样式.

#### 登录表单
后台的登录表单是`AdminAuthenticationForm`
```python
# django.contrib.sites.py
class AdminSite:
    def get_urls(self):
        # Admin-site-wide views.
        urlpatterns = [
            path('login/', self.login, name='login'),
            path('logout/', wrap(self.logout), name='logout'),
            ...
        ]
    def login(self, request, extra_context=None):
        defaults = {
                'extra_context': context,
                'authentication_form': self.login_form or AdminAuthenticationForm,
                'template_name': self.login_template or 'admin/login.html',
            }
```
所以我们需要继承该表单,添加验证码字段

```python
# simpleui_captcha.forms.py
from captcha.fields import CaptchaField
from django.contrib.admin.forms import AdminAuthenticationForm


class MultiCaptchaAdminAuthenticationForm(AdminAuthenticationForm):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.fields['captcha'] = CaptchaField()
```

然后赋值给站点的登录表单变量
```python
# simpleui_captcha.admin.py
from django.contrib import admin

from .forms import MultiCaptchaAdminAuthenticationForm

admin.AdminSite.login_form = MultiCaptchaAdminAuthenticationForm
```

#### 验证码相关url
用来刷新验证码
```python
# simpleui_captcha.urls.py
from django.urls import path, include

urlpatterns = [
    path('captcha/', include('captcha.urls')),
]
```

#### 模板修改
扩展后台登录模板文件,把验证码字段加入表单,并加入刷新验证码功能
```html
# simpleui_captcha/templates/admin/login.html
{% extends "admin/login.html" %}

<script>
    window.onload = function () {
        let captchaEle = document.querySelector("img.captcha");
        captchaEle.onclick = function () {
            $.getJSON("/simpleui_captcha/captcha/refresh/", function (result) {
                $('img.captcha').attr('src', result['image_url']);
                $('#id_captcha_0').val(result['key'])
            });
        };
        ...
</script>

{% block form %}
{{ block.super }}
<div hidden="hidden">
    {{ form.captcha }}
</div>
{% endblock %}
```

### 发布
按照`django`官方文档-[进阶指南：如何编写可重用程序](https://docs.djangoproject.com/zh-hans/3.2/intro/reusable-apps/) 的介绍, 完成项目,打包应用,发布应用.最后还可以提交到`github`上,让大家来贡献代码. 点击 [django-simpleui-captcha](https://github.com/dust8/django-simpleui-captcha) 给个`Star` 吧.
