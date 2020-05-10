---
title: Django 使用旧有的数据库
date: 2020-05-10 05:15:31
tags:
  - django
---

有个项目是用其他语言写的接口, 需要快速开发出后台管理页面, 就选用了 `django`. 按照下面的方法就可以在几分钟内开发完成.

## 生成模型文件

这个官网有教程, 可以点击下面的参考链接看. 只需要运行一条命令就可以生成模型文件

```bash
python manage.py inspectdb > models.py
```

## 生成后台管理文件

```python
# lib/gen_admin.py
import inspect
import os
from pathlib import Path
import sys

import django


def get_classes(arg):
    """找出模块里所有的类"""
    classes = []

    clsmembers = inspect.getmembers(arg, inspect.isclass)
    for (_, value) in clsmembers:
        classes.append(value)

    return classes


def gen_admin(model):
    """根据模型生成 admin.py 的内容"""
    temp = f"""
@admin.register({model.__name__})
class {model.__name__}Admin(admin.ModelAdmin):
    list_display = ('{"','".join((field.name for field in model._meta.fields))}')
    """
    return temp


def gen_admins(models):
    result = ""
    for model in models:
        result += gen_admin(model)
    return result


def main():
    BASE_DIR = Path(__file__).resolve(strict=True).parent.parent
    sys.path.append(str(BASE_DIR))

    # 改成你的配置
    os.environ.setdefault("DJANGO_SETTINGS_MODULE", "dazhong.settings")
    django.setup()

    # 要生成的模型
    import areas

    classes = get_classes(areas.models)
    result = gen_admins(classes)
    print(result)


if __name__ == "__main__":
    main()


```

## 参考链接

- [Django 使用旧有的数据库](https://docs.djangoproject.com/zh-hans/3.0/howto/legacy-databases/)
- [外部脚本调用 Django 环境](https://huoyingwhw.com/2019/05/01/%E5%A4%96%E9%83%A8%E8%84%9A%E6%9C%AC%E8%B0%83%E7%94%A8Django%E7%8E%AF%E5%A2%83/)
- [Use Pathlib in Your Django Settings File](https://adamj.eu/tech/2020/03/16/use-pathlib-in-your-django-project/)
