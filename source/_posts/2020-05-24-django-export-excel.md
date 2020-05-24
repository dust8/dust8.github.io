---
title: django 导出 excel 文件
date: 2020-05-24 05:15:31
tags:
  - python
  - django
  - excel
---

`django admin` 后台需要导出 `excel` 文件. 除了用插件, 也可以自己简单的实现.
`lib` 目录是一些自己写的库, 在 `settings` 文件里面把它加入了 `sys.path`.
导出的时候需要选中要导出的数据才能导出, 不然只点导出按钮是无效的. 因为模型里面有些字段是 `choice`
所以增加判断, 有 `get_xx_display` 属性的时候, 返回比较友好的显示.

```python
# lib/export.py
from django.http import HttpResponse
from openpyxl import Workbook


class ExportExcelMixin:
    '''
    通用导出 Excel 文件动作
    '''

    def export_as_excel(self, request, queryset):
        '''
        导出为 excel 文件. 文件名为 app名.模型类名.xlsx.

        :param request:
        :param queryset:
        :return:
        '''
        # 用于定义文件名, 格式为: app名.模型类名
        meta = self.model._meta
        # 模型所有字段名
        field_names = [field.name for field in meta.fields]
        field_verbose_names = [field.verbose_name for field in meta.fields]

        # 定义响应内容类型
        response = HttpResponse(content_type='application/msexcel')
        # 定义响应数据格式
        response['Content-Disposition'] = f'attachment; filename={meta}.xlsx'

        wb = Workbook()
        ws = wb.active
        ws.append(field_verbose_names)

        # 遍历选择的对象列表
        for obj in queryset:
            # 将模型属性值的文本格式组成列表
            data = []
            for field in field_names:

                if hasattr(obj, f'get_{field}_display'):
                    # 可选属性取显示的值
                    value = getattr(obj, f'get_{field}_display')()
                else:
                    value = getattr(obj, field)

                data.append(f'{value}')

            ws.append(data)

        # 将数据存入响应内容
        wb.save(response)

        return response

    # 该动作在 admin 中的显示文字
    export_as_excel.short_description = '导出Excel'

```

```python
# admin.py
from django.contrib import admin


from export import ExportExcelMixin
from .models import *

class XXAdmin(admin.ModelAdmin, ExportExcelMixin):
    ...

    actions = ['export_as_excel']
```

## 参考链接

- [Admin actions](https://docs.djangoproject.com/zh-hans/3.0/ref/contrib/admin/actions/)
- [Django Admin 中增加导出 Excel 功能](https://www.cnblogs.com/superhin/p/11454758.html)
