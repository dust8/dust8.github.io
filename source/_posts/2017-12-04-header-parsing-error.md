---
title: requests 解析头部出错
date: 2017-12-04 05:15:31
tags:
 - requests
---

今天在使用 `requests` 下载附件的时候出现 `MissingHeaderBodySeparatorDefect` 错误，感觉有点意思，记录下解决办法。


## 起因
我不知道要下载的文件名和后缀名，url 里面没有，文件名和后缀名只能从返回的响应头里找到，所以需要解析头部。

## 文件名是乱码
通过找资料知道默认的编码是  `iso-8859-1` ，所以很简单的先编码在解码就搞定了。

```python
res.headers['content-disposition'].encode('iso-8859-1').decode()
```
虽然是解决了问题，但是又碰到了一个开始说的问题

## 解析头部出错
去搜索出错信息，资料很少，最后发现一个相关的问题 [HeaderParsingError: Failed to parse headers ](https://github.com/requests/requests/issues/3098).
有人说不关 `requests` 的事，`requests` 解析头部用的是标准库里的库解析的。有人也给了解决方案，不过他说要放到 `requests` 里面去，其实只要在自己的代码里面重载就可以了。

```python
def parse_headers(fp, _class=HTTPMessage):
    """Parses only RFC2822 headers from a file pointer.

    email Parser wants to see strings rather than bytes.
    But a TextIOWrapper around self.rfile would buffer too many bytes
    from the stream, bytes which we later need to read as bytes.
    So we read the correct bytes here, as bytes, for email Parser
    to parse.

    """
    headers = []
    while True:
        line = fp.readline(_MAXLINE + 1)
        if len(line) > _MAXLINE:
            raise LineTooLong("header line")
        headers.append(line)
        if len(headers) > _MAXHEADERS:
            raise HTTPException("got more than %d headers" % _MAXHEADERS)
        if line in (b'\r\n', b'\n', b''):
            break
    # 默认 iso-8859-1 编码。避免解析头部错误，用 utf8 编码
    # hstring = b''.join(headers).decode('iso-8859-1')
    hstring = b''.join(headers).decode()
    return email.parser.Parser(_class=_class).parsestr(hstring)


http.client.parse_headers = parse_headers
```