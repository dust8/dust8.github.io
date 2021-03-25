---
title: django之save_model
date: 2021-03-26 05:15:31
tags:
  - django
  - django-admin
---


```python
def save_model(self, request, obj, form, change):
    # form.initial 表单里面的初始值保存着原来的数据, clean_data 里面就是更改后的数据
    user_info = form.initial["user_info"]
    try:
        user_instance = obj.user_classmets
        # 这是保存改动
        super().save_model(request, obj, form, change)
        # 特殊的目的,因为需要保存后才能处理
        xxx(user_instance)
    except Exception as ex:
        # 这里在外键后面直接加_id,来设置数字,不然必须是实例
        # 出错了还原数据
        obj.user_info_id = user_info
        obj.save()
        # 只发送错误信息, 更改的信息不显示, 不然会叠加显示2条信息,导致错误信息被遮罩
        messages.error(request, f"{ex}")
        messages.set_level(request, messages.ERROR)
```

## 参考链接
- [ModelAdmin.save_model](https://docs.djangoproject.com/zh-hans/3.1/ref/contrib/admin/#django.contrib.admin.ModelAdmin.save_model)
- [django-admin只显示自定义的提示信息](https://blog.csdn.net/Keep_on_Growing/article/details/100929122)