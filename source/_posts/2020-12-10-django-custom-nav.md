---
title: django与后台自定义导航
date: 2020-12-10 05:15:31
tags:
  - django
  - admin
  - simpleui
---

有些自定义的后台页面, 但是有权限才能操作. 用的 `simpleui` 不支持权限, `simple pro` 才支持, 不是免费的. 只能自己定制.

## 添加后台自定义导航
这样添加后是所有用户都看得到导航
```python
# 自定义导航
SIMPLEUI_CONFIG = {
    'system_keep': True,
    'dynamic': True,  # 设置是否开启动态菜单, 默认为False. 如果开启, 则会在每次用户登陆时动态展示菜单内容
    'menus': [{
        'name': '自定义导航',
        'icon': 'fa fa-file',
        'codename': 'c_app',
        'models': [{
            'name': '添加同学',
            'url': 'c_app/c_xx/',
            'icon': 'fas fa-home',
            'codename': 'c_xx',
        }, ],
    }, ],
}
```

## 自定义导航支持权限
### 数据库操作
|  表名   | 名字  | 作用 |
|  ----  | ----  | ----  |
| django_content_type  | 已经安装模型表 | 用来描述已经安装的模型  |
| auth_permission  | 权限表 | 用户记录所有模块的view,add,change,delete权限，与django_content_type关联  |

在 `django_content_type` 里面添加 `app` 和 模型
```sql
INSERT INTO  `django_content_type`(`app_label`,`model`) VALUES ('c_app','c_xx');
```

插入成功后需要记住 `id` 的值, 例如 `95`

在 `auth_permission` 表里面添加权限记录. 增删改查每种对应一条权限, 例如查看权限已 `view` 开头
```sql
INSERT INTO `auth_permission`(`name`,`content_type_id`,`codename`)VALUES('Can view xx', 95 , 'view_c_xx');
```

### 后台给用户分配权限
在后台给用户把权限加上

### 权限判断
查看了 `simpleui` 导航代码, 把他修改了下, 注释定制的就是不一样的代码.  
在 `app` 目录下面新建 `templatetags` 文件夹里面新建 `simpletags.py`  文件, 里面定制一个菜单导航 `c_menus`
```
# simpletags.py
import copy
import json

from simpleui.templatetags.simpletags import get_config, _import_reload, get_icon, unicode_to_str, LazyEncoder, register


@register.simple_tag(takes_context=True)
def c_menus(context, _get_config=None):
    data = []

    # return request.user.has_perm("%s.%s" % (opts.app_label, codename))
    if not _get_config:
        _get_config = get_config

    config = _get_config('SIMPLEUI_CONFIG')
    if not config:
        config = {}

    if config.get('dynamic', False) is True:
        config = _import_reload(_get_config('DJANGO_SETTINGS_MODULE')).SIMPLEUI_CONFIG

    app_list = context.get('app_list')
    for app in app_list:
        _models = [
            {
                'name': m.get('name'),
                'icon': get_icon(m.get('object_name'), unicode_to_str(m.get('name'))),
                'url': m.get('admin_url'),
                'addUrl': m.get('add_url'),
                'breadcrumbs': [{
                    'name': app.get('name'),
                    'icon': get_icon(app.get('app_label'), app.get('name'))
                }, {
                    'name': m.get('name'),
                    'icon': get_icon(m.get('object_name'), unicode_to_str(m.get('name')))
                }]
            }

            for m in app.get('models')
        ] if app.get('models') else []

        module = {
            'name': app.get('name'),
            'icon': get_icon(app.get('app_label'), app.get('name')),
            'models': _models
        }
        data.append(module)

    # 如果有menu 就读取，没有就调用系统的
    key = 'system_keep'

    # [定制]获取当前用户权限
    permissions = context.request.user.get_all_permissions()

    if config and 'menus' in config:
        if config.get(key, None):
            temp = copy.deepcopy(config.get('menus'))
            for i in temp:
                # 处理面包屑
                if 'models' in i:
                    # [定制]去掉没有权限的自定义菜单
                    for k in i.get('models')[:]:
                        if 'codename' in k and f"{i['codename']}.view_{k['codename']}" not in permissions:
                            if not context.request.user.is_superuser:
                                i.get('models').remove(k)
                                continue

                        k['breadcrumbs'] = [{
                            'name': i.get('name'),
                            'icon': i.get('icon')
                        }, {
                            'name': k.get('name'),
                            'icon': k.get('icon')
                        }]
                else:
                    i['breadcrumbs'] = [{
                        'name': i.get('name'),
                        'icon': i.get('icon')
                    }]

                # [定制]去掉空的一级菜单
                if 'models' in i and len(i['models']) == 0:
                    continue

                data.append(i)
        else:
            data = config.get('menus')

    # 获取侧边栏排序, 如果设置了就按照设置的内容排序, 留空则表示默认排序以及全部显示
    if config.get('menu_display') is not None:
        display_data = list()
        for _app in data:
            if _app['name'] not in config.get('menu_display'):
                continue
            _app['_weight'] = config.get('menu_display').index(_app['name'])
            display_data.append(_app)
        display_data.sort(key=lambda x: x['_weight'])
        data = display_data

    # 给每个菜单增加一个唯一标识，用于tab页判断
    eid = 1000
    for i in data:
        eid += 1
        i['eid'] = eid
        if 'models' in i:
            for k in i.get('models'):
                eid += 1
                k['eid'] = eid

    menus_string = json.dumps(data, cls=LazyEncoder)

    # 把data放入session中，其他地方可以调用
    if not isinstance(context, dict) and context.request:
        context.request.session['_menus'] = menus_string

    return '<script type="text/javascript">var menus={}</script>'.format(menus_string)

```

### 模板使用定制导航
使用自定义 `admin` 应用模板, 把 `simpleui` 的模板文件 `index.html` 里面的 `menus` 改成定制的 `c_menus`
```
{% block menus %}
    {% autoescape off %}
        {% c_menus %}
    {% endautoescape %}
{% endblock %}
```



## 参考链接
- [simplepro 自定义权限](https://simpleui.72wo.com/docs/simplepro/permissions.html)
- [The contenttypes framework](https://docs.djangoproject.com/zh-hans/3.1/ref/contrib/contenttypes/)
- [自定义你 应用的 模板](https://docs.djangoproject.com/zh-hans/3.1/intro/tutorial07/#customizing-your-application-s-templates)
- [关于simpleui菜单设置与django自带和自定义权限的控制调整](https://simpleui.72wo.com/topic/127/)