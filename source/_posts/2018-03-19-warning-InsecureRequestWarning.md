---
title: 利用warnings忽略InsecureRequestWarning警告
date: 2018-03-19 05:15:31
tags:
  - 已码学码
  - 标准库
---

![warnings](./assert/2018-03-19.png)

今天在 `requests` 的源代码里面看到下面的写法

```python
import warnings

# urllib3's DependencyWarnings should be silenced.
from .packages.urllib3.exceptions import DependencyWarning
warnings.simplefilter('ignore', DependencyWarning)
```

`warnings` 是标准库里面的, 通过过滤器可以忽略警告.

当我用 `requests` 请求 `https` 时总是报 `InsecureRequestWarning` 错误,现在就可以这样处理了.

```python
import warnings

from requests.packages.urllib3.exceptions import InsecureRequestWarning

warnings.simplefilter('ignore', InsecureRequestWarning)
```
