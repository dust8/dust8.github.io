---
title: django restful framework 之序列化使用
date: 2020-05-02 05:15:31
tags:
  - django
  - restful
---

常见的我就不说了, 我下面说一些我用到的, 又不容易搜索到的知识. 其实官网文档有说明, 可能会漏看,或者看过不知道用在什么场景.

## 嵌套序列化过滤

在一对多模型中, 如果通过一来查出多的模型,会把多的模型数据全部返回. 但是我现在只需要返回多的模型里面一部分数据,例如没有逻辑删的数据. 这时候可以指定多模型的 `list_serializer_class` 并重写 `listserializer` 的 `to_representation` 方法.
`.to_representation() - Override this to support serialization, for read operations.`

```python
class CustomListSerializer(serializers.ListSerializer):
    ...
    def to_representation(self, instance):
        instance = instance.filter(is_deleted=False)
        ret = super().to_representation(instance)

        return ret

class CustomSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = CustomListSerializer


```

## 增加额外的字段

还是一对多的模型里面,想要在一的返回数据里面增加一个多的数量字段.还是可以重写 `to_representation` 方法.同理也可以减少或者修改字段和值.

```python
def to_representation(self, instance):
    instance = instance.filter(is_deleted=False)
    ret = super().to_representation(instance)

    ret['many_num'] = Many.objects.filter(one=instance).count()

    return ret

```

## 越级筛选数据

这个是因为多层嵌套模型,在序列化中传值问题,例如把请求的参数传入到后面几层的序列化里面过滤.

```python
def to_representation(self, instance):
    request = self.context['request']

    instance = instance.filter(user=request.user)
    ret = super().to_representation(instance)

    return ret

```

## 参考链接

- [listserializer](https://www.django-rest-framework.org/api-guide/serializers/#listserializer)
- [overriding-serialization-and-deserialization-behavior](https://www.django-rest-framework.org/api-guide/serializers/#overriding-serialization-and-deserialization-behavior)
- [Including extra context](https://www.django-rest-framework.org/api-guide/serializers/#including-extra-context)
- [Django-REST-framework 嵌套序列化过滤](http://program.dengshilong.org/2018/08/09/Django-REST-framework%E5%B5%8C%E5%A5%97%E5%BA%8F%E5%88%97%E5%8C%96%E8%BF%87%E6%BB%A4/)
