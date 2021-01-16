---
title: django-filter使用
date: 2020-12-24 05:15:31
tags:
  - django
  - django-filter
---

[django-filter](https://django-filter.readthedocs.io/en/stable/) 可以用来过滤查询. 跟 [Django REST framework](https://www.django-rest-framework.org/) 很配.

## 使用
```python
# 模型
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=255)
    price = models.DecimalField()
    description = models.TextField()
    release_date = models.DateField()
    manufacturer = models.ForeignKey(Manufacturer)
```
```python
# 过滤器
import django_filters

class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = ['price', 'release_date']
```

## 指定字段, 排除字段
```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        # 指定字段
        fields = ['username', 'last_login']
```
```python
class UserFilter(django_filters.FilterSet):
    class Meta:
        model = User
        # 排除字段
        exclude = ['username', 'last_login']
```

## 过滤大于和小于或者包含等于
```python
class ProductFilter(django_filters.FilterSet):
    price = django_filters.NumberFilter()
    price__gt = django_filters.NumberFilter(field_name='price', lookup_expr='gt')
    price__lt = django_filters.NumberFilter(field_name='price', lookup_expr='lt')
    price__lte = django_filters.NumberFilter(field_name='price', lookup_expr='lte')

    class Meta:
        model = Product

class ProductFilter(django_filters.FilterSet):
    class Meta:
        model = Product
        fields = {
            'price': ['lt', 'gt'],
            'release_date': ['exact', 'year__gt'],
        }
```

## 过滤范围
一般是名字里面包含 `Range`, 例如 `RangeFilter` , `NumericRangeFilter` ,  `DateRangeFilter` , `DateFromToRangeFilter`, 查询时 `url` 为 `/?date_after=2016-01-01&date_before=2016-02-01`
```python
class Comment(models.Model):
    date = models.DateField()
    time = models.TimeField()

class F(FilterSet):
    date = DateFromToRangeFilter()

    class Meta:
        model = Comment
        fields = ['date']

# Range: Comments added between 2016-01-01 and 2016-02-01
f = F({'date_after': '2016-01-01', 'date_before': '2016-02-01'})

# Min-Only: Comments added after 2016-01-01
f = F({'date_after': '2016-01-01'})

# Max-Only: Comments added before 2016-02-01
f = F({'date_before': '2016-02-01'})
```

## 自定义 method
### 自定义搜索多个字段
```python
# https://django-filter.readthedocs.io/en/stable/guide/usage.html#customize-filtering-with-filter-method
class F(django_filters.FilterSet):
    username = CharFilter(method='my_custom_filter')

    class Meta:
        model = User
        fields = ['username']

    def my_custom_filter(self, queryset, name, value):
        return queryset.filter(**{
            name: value,
        })

# https://django-filter.readthedocs.io/en/stable/guide/tips.html#filtering-by-relative-times
from django.utils import timezone
from datetime import timedelta
...

class DataModel(models.Model):
    time_stamp = models.DateTimeField()


class DataFilter(django_filters.FilterSet):
    hours = django_filters.NumberFilter(
        field_name='time_stamp', method='get_past_n_hours', label="Past n hours")

    def get_past_n_hours(self, queryset, field_name, value):
        time_threshold = timezone.now() - timedelta(hours=int(value))
        return queryset.filter(time_stamp__gte=time_threshold)

    class Meta:
        model = DataModel
        fields = ('hours',)
```

```python
# 同时搜索多个字段
class ClassSellInfoFilter(filters_rest.FilterSet):
    fuzzy_search = filters_rest.CharFilter(method='search_title_or_class_tag')

    def search_title_or_class_tag(self, queryset, name, value):
        return queryset.filter(Q(class_sell_title__icontains=value) | Q(class_name__class_tag__icontains=value))

    class Meta:
        model = ClassSellInfo
        fields = ['fuzzy_search']
```

## 参考链接
- [django-filter](https://django-filter.readthedocs.io/en/stable/)
- [DateFromToRangeFilter](https://django-filter.readthedocs.io/en/master/ref/filters.html#datefromtorangefilter)
- [Python系列之《Django-DRF-搜索》](https://www.dazhuanlan.com/2019/12/05/5de8e06c9fab8/)