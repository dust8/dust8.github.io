---
title: django-import-export使用
date: 2020-12-24 05:15:31
tags:
  - django
  - django-import-export
---

[django-import-export/django-import-export](https://github.com/django-import-export/django-import-export) 是 `django` 后台管理页面导入导出插件.

## 安装
```bash
pip install django-import-export
```

```python
# settings.py
INSTALLED_APPS = (
    ...
    'import_export',
)
```

## 使用
### 创建导入导出资源类
和 `drf` 的序列化类很类似. 多看文档, 有各种需求的例子, 例如自定义字段, 字段排序, 关联字段等.
```python
# app/admin.py

from import_export import resources
from core.models import Book

class BookResource(resources.ModelResource):

    class Meta:
        model = Book
```

### 和 admin 整合
这里要注意看它的 `Admin` 接口, 有很多种方式, 例如     
- `ExportActionMixin`
- `ExportActionModelAdmin`
- `ExportMixin`
- `ImportMixin`
- ...

看名字就知道, 其实就是把导入,导出各种组合.  `Mixin` 和 `admin.ModelAdmin` 组合变成 `ModelAdmin`.    
含有 `Action` 就是直接在列表页显示导出选项, 不需要进入下一层页面选择.

```python
from .models import Book
from import_export.admin import ImportExportModelAdmin

class BookAdmin(ImportExportModelAdmin):
    resource_class = BookResource

admin.site.register(Book, BookAdmin)
```

### 常见使用
```python
# 定义导出字段 fields
class BookResource(resources.ModelResource):

    class Meta:
        model = Book
        fields = ('id', 'name', 'price', 'author__name', )
```
```python
# 排除导出字段 exclude
class BookResource(resources.ModelResource):

    class Meta:
        model = Book
        exclude = ('imported', )
```
```python
# 排序导出字段 export_order
class BookResource(resources.ModelResource):

    class Meta:
        model = Book
        fields = ('id', 'name', 'author', 'price',)
        export_order = ('id', 'price', 'author', 'name')
```

### 导出自定义字段
命名方式 `dehydrate_<fieldname>`
```python
from import_export.fields import Field

class BookResource(resources.ModelResource):
    full_title = Field()

    class Meta:
        model = Book

    def dehydrate_full_title(self, book):
        return '%s by %s' % (book.name, book.author.name)
```

## 问题
### 1. 如果有自定义的  `actions`, 导入导出会不显示
查看 `django-import-export` 源码
```python
# import_exprt/admin.py
class ExportActionMixin(ExportMixin):
    ...

    actions = admin.ModelAdmin.actions + [export_admin_action]

class ExportActionModelAdmin(ExportActionMixin, admin.ModelAdmin):
    """
    Subclass of ModelAdmin with export functionality implemented as an
    admin action.
    """
```

看起来是没问题, 实际上这个 `actions` 已经被我们自定义的 `actions` 覆盖了. 在加上就可以了.
```python
class BookAdmin(ImportExportModelAdmin):
    resource_class = BookResource
    actions= ["xxxx", ExportActionMixin.export_admin_action]

admin.site.register(Book, BookAdmin)
```

### 2.导出变成了导出全部, 多选功能没有用
目前只能用 `ExportActionMixin` 这类带有 `Action` 的方法, 让它在列表页直接选择, 不要进入下一层就可以实现多选导出.


## 参考链接
- [django-import-export 文档](https://django-import-export.readthedocs.io/en/stable/)