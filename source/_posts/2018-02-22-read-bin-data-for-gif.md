---
title: 解析gif头部
date: 2018-02-22 06:15:31
---

## 起因
无聊看到 《python3 cookbook》 里面的 “读取嵌套和可变长二进制数据” 又感受到了
不会学习。这本书很久很久之前就知道也翻过，就是没仔细看。
后来解析协议，解析二进制文件（比如字体文件woff2）都没想到好的方法，只会文章里面的不怎么方便的方法。
古人诚不欺我，读书百遍，其义自见（xian）。我最近也在清理以前保存的书签，也发现很多以前没注意到却很有用的东西。

## 解析 gif 头部
gif 协议：https://www.w3.org/Graphics/GIF/spec-gif89a.txt

```python
import struct


class StructField:
    def __init__(self, format_str, offset):
        self.format_str = format_str
        self.offset = offset

    def __get__(self, instance, cls):
        if instance is None:
            return self
        else:
            r = struct.unpack_from(self.format_str, instance._buffer,
                                   self.offset)
            return r[0] if len(r) == 1 else r


class StructureMeta(type):
    def __init__(self, clsname, bases, clsdict):
        fields = getattr(self, '_fields_', [])
        byte_order = ''
        offset = 0
        for format_str, fieldname in fields:
            if format_str.startswith(('<', '>', '!', '@')):
                # 这里我开始以为没有用，其实是用来缓存字节序的
                byte_order = format_str[0]
                format_str = format_str[1:]
            format_str = byte_order + format_str
            setattr(self, fieldname, StructField(format_str, offset))
            offset += struct.calcsize(format_str)
        setattr(self, 'struct_size', offset)


class Structure(metaclass=StructureMeta):
    def __init__(self, bytedata):
        self._buffer = bytedata

    @classmethod
    def from_file(cls, fh):
        return cls(fh.read(cls.struct_size))


class GifHeader(Structure):
    '''https://www.w3.org/Graphics/GIF/spec-gif89a.txt'''
    _fields_ = [('3s', 'signature'), ('3s', 'version')]


if __name__ == '__main__':
    with open('bin.gif', 'rb') as fh:
        gif_header = GifHeader.from_file(fh)
        print('signature: ', gif_header.signature.decode())  # signature:  GIF
        print('version: ', gif_header.version.decode())  # version:  89a

```