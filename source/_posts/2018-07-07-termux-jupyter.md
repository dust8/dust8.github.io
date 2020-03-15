---
title: termux与jupyter
date: 2018-07-07 05:15:31
---

[termux](https://github.com/termux/termux-app) 是 `android` 下的 `Linux` 环境和命令行模拟器.

## termux 文档

- [Touch Keyboard](https://wiki.termux.com/wiki/Touch_Keyboard)
- [Internal and external storage](https://wiki.termux.com/wiki/Internal_and_external_storage)

## 安装 jupyter

### 基本使用

```bash
apt update
apt install python python-dev
apt install clang libzmq libzmp-dev libcrypt-dev
pip install jupyter
```

如果漏装依赖会报错

- 'Python.h' file not found
- 'crypt.h' file not found

安装成功就可以正常使用了.

### 高级使用

#### 对外使用和设置密码

具体可以看 [Running a notebook server](https://jupyter-notebook.readthedocs.io/en/stable/public_server.html)  
这样就可以在其他电脑连接手机里面的 `jupyter`, 并设置了密码，地址就不用代 `token` 了.

#### 使用魔法方法

在 `jupyter` 里面运行 `linux` 命令.  
![jupyter magic](./assert/2018-07-07.png)
