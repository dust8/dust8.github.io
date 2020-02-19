---
title: 使用tesseract识别支付二维码
date: 2020-02-19 05:15:31
tags:
  - Github Ations
  - tesseract
  - dpayocr
  - ocr
  - pypi
---

最近在写个人收款项目，需要识别收款码里面的信息，例如支付类型，金额，链接，就想到了用 `tesseract` 来识别金额，用 `pyzbar` 来识别链接，在根据链接判断支付类型。事实证明是可行的，并做成了个包发布到 `pypi` 上.

[dpayocr](https://github.com/dust8/dpayocr) : 可以识别支付宝微信收款码信息。例如支付类型，支付金额，支付链接。

## 开发问题

### tesseract

- 主要是虚拟环境找不到命令，需要指定路径。
- 支付宝和微信用同样的命令识别不了金额，对它们分别用不同的参数。前者用 `--psm 6`, 后者用 `--psm 11`

### 上传 pypi

现在可以用 `Github Actions` 来自动发布到 `pypi` 了。需要先去 `pypi` 生成 `api-token` 放到仓库的设置里面。当收到推送后，检测到发布标签就上传到 `pypi`。这里要注意在本地打了标签后，还需要 `Sharing Tags` ，例如 `git push origin v1.5` 这样才能检测到。具体的可以看下面的参考链接。

## 安装

```bash
$ pip install dpayocr
```

## 使用

### 命令行使用

```
dpayocr 66.JPG
```

输出

```
PayQrCode(pay_type=1, price=66.0, url='https://qr.alipay.com/xxx')
```

### 接口

```python
>>> from dpayocr import parse_pay_qr_code
>>> fp = '66.JPG' # 可以传入文件名或者二进制IO
>>> parse_pay_qr_code(fp)
PayQrCode(pay_type=1, price=66.0, url='https://qr.alipay.com/xxx')
```

## 问题

### 在虚拟环境找不到 Tesseract 命令

```
# If you don't have tesseract executable in your PATH, include the following:
pytesseract.pytesseract.tesseract_cmd = r'<full_path_to_your_tesseract_executable>'
# Example tesseract_cmd = r'C:\Program Files (x86)\Tesseract-OCR\tesseract'
```

## 参考链接

- [tesseract](https://github.com/tesseract-ocr/tesseract)
- [pytesseract](https://pypi.org/project/pytesseract/)
- [pyzbar](https://pypi.org/project/pyzbar/)
- [A comprehensive guide to OCR with Tesseract, OpenCV and Python](https://nanonets.com/blog/ocr-with-tesseract/)
- [Publishing package distribution releases using GitHub Actions CI/CD workflows](https://packaging.python.org/guides/publishing-package-distribution-releases-using-github-actions-ci-cd-workflows/)
- [gh-action-pypi-publish](https://github.com/pypa/gh-action-pypi-publish)
- [Git-Basics-Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
