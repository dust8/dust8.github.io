---
title: Django 使用upload_to 自定义上传路径
date: 2020-05-12 05:15:31
tags:
  - django
---

## 简单自定义

这个官网有教程, 可以点击下面的参考链接看.

```python
class MyModel(models.Model):
    # file will be uploaded to MEDIA_ROOT/uploads
    upload = models.FileField(upload_to='uploads/')
    # or...
    # file will be saved to MEDIA_ROOT/uploads/2015/01/30
    upload = models.FileField(upload_to='uploads/%Y/%m/%d/')
```

```python
def user_directory_path(instance, filename):
    # file will be uploaded to MEDIA_ROOT/user_<id>/<filename>
    return 'user_{0}/{1}'.format(instance.user.id, filename)

class MyModel(models.Model):
    upload = models.FileField(upload_to=user_directory_path)
```

## 在复杂一些

```python
def _upload_to(attrs=None,root=''):
    def upload_to(instance, filename):
        ins = instance

        if attrs:
            for attr in attrs.split('.'):
                ins = getattr(ins,attr)

            path = f'{root}{ins}/{filename}'
        else:
            path = f'{root}{filename}'

        return path

    return upload_to


class TBusinesschain(models.Model):
    bc_id = models.AutoField(db_column='BC_Id', primary_key=True)
    bc_logo = models.FileField(upload_to=_upload_to('bc_id','business/chain/'),db_column='BC_Logo', max_length=50, verbose_name='店铺Log')


class TBusinesscoupon(models.Model):
    bc_id = models.AutoField(db_column='BC_Id', primary_key=True)
    bc_shopid = models.ForeignKey(TBusiness, on_delete=models.CASCADE,
    bc_logo = models.FileField(upload_to=_upload_to('bc_shopid.bi_id','business/'),db_column='BC_Logo', max_length=50, verbose_name='logo')
```

## 参考链接

- [FileField.upload_to](https://docs.djangoproject.com/zh-hans/3.0/ref/models/fields/#django.db.models.FileField.upload_to)
