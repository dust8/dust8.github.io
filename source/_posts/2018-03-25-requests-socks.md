---
title: requests与socks
date: 2018-03-25 05:15:31
tags:
  - 已码学码
---

![socks](./assert/2018-03-25.png)

## socks

[SOCKS](http://docs.python-requests.org/en/master/user/advanced/#socks)  
`requests` 从 `2.10.0` 版本支持 `socks` 协议.  
安装:

```bash
pip install requests[socks]
```

使用:

```python
proxies = {
    'http': 'socks5://user:pass@host:port',
    'https': 'socks5://user:pass@host:port'
}
```

## requests 怎么使用 socks 的

首先是要安装额外库

```bash
pip install requests[socks]
```

这种安装方法来自 [setuptools](http://setuptools.readthedocs.io/en/latest/setuptools.html) 的 `extras_require` . 可以在 `requests` 的 `setup.py` 里面看到.

```python
extras_require={
        'security': ['pyOpenSSL>=0.14', 'cryptography>=1.3.4', 'idna>=2.0.0'],
        'socks': ['PySocks>=1.5.6, !=1.5.7'],
        'socks:sys_platform == "win32" and (python_version == "2.7" or python_version == "2.6")': ['win_inet_pton'],
    }
```

可以看到会安装 [PySocks](https://github.com/Anorov/PySocks) 库.

`PySocks` 的引入方式是 `import socks` . 在 `requests` 里面只在 `requests/adapters.py` 看到

```python
from urllib3.contrib.socks import SOCKSProxyManager

DEFAULT_POOLSIZE = 10
```

使用的是 [urllib3](https://github.com/shazow/urllib3/) 库.

```python
class SOCKSProxyManager(PoolManager):
    """
    A version of the urllib3 ProxyManager that routes connections via the
    defined SOCKS proxy.
    """
    pool_classes_by_scheme = {
        'http': SOCKSHTTPConnectionPool,
        'https': SOCKSHTTPSConnectionPool,
    }

    def __init__(self, proxy_url, username=None, password=None,
                 num_pools=10, headers=None, **connection_pool_kw):
        parsed = parse_url(proxy_url)

        if parsed.scheme == 'socks5':
            socks_version = socks.PROXY_TYPE_SOCKS5
            rdns = False
        elif parsed.scheme == 'socks5h':
            socks_version = socks.PROXY_TYPE_SOCKS5
            rdns = True
        elif parsed.scheme == 'socks4':
            socks_version = socks.PROXY_TYPE_SOCKS4
            rdns = False
```

可以看到 `requests` 和 `urllib3` 设置的默认连接池大小都是 10.
