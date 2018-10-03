---
title: 布隆过滤器
date: '2016/3/16 20:00:00'
tags:
  - Filter
  - BloomFilter
---

`Bloom Filter` , 中文译作布隆过滤器。<br>
从 `filter` 和过滤器可以看出它的用途，筛选东西，判断它是否满足要求。 它的优点是空间效率和时间效率，缺点是有一定的误判率和撤销困难。

# 1\. bit 和 byte 的关系

```
1 byte = 8 bit    
```

一个字节 (`byte`) 有八位 (`bit`)。

# 2\. 优点

## 2.1 空间效率

```
s1 = 'http://www.dust8.com'
s2 = 'http://www.the5fire.com'
>>> len(s1), len(s2)
(20, 23)
```

字符串s1的长度为20，字符串s2的长度为23。也就是说s1占20个字节，s2占23个字节。<br>
有人觉得它们太占地方了，想了个办法减少它们占的空间。 用它们的长度来代替它们，这样每个字符串都可以只用一个字节表示，大大减少空间。慢慢的字符串多了起来，有了

```
s3 = 'http://www.github.com'
s4 = 'http://www.google.com'    
>>> len(s3), len(s4)
(21, 21)
```

字符串s3和s4的长度都为21。要表示这4个字符串就要4个字节。有人又想了个办法，用一个bit来表示一个字符串。比如表示s1，就用第20个bit来表示，s2就用第23个bit来表示。怎么表示了，常见的就是把该bit设置为1，跟默认值不一样。这4个字符串长度最大的为23，所以用3个字节就可以表示它们了。<br>
85个字节用3个字节就可以表示了，所以说它空间效率高。

## 2.2 时间效率

它在内存操作，所以速度快。<br>
它只要一次查询就可以完成，所以速度快。比如要看是否有s1，只要看第20个bit是不是1，是1就有，如果是0则没有。

# 3.缺点

## 3.1 误判

s3和s4的长度都为21，都用第21个bit来表示，这就会产生误判。分不清表示的是s3，还是s4，或者2个都表示。

## 3.2 撤销困难

同样是s3和s4，如果表示了s3和s4，那么第21个bit就是1。现在不想表示s3了，却不能把第21个bit设置为0，设置为0就会影响s4的表示。

# 4\. 设计Bloom Filter

如何降低误判率是关键，这需要

- 选取区分度高的哈希函数。比如上面的长度函数区分度就比较低，导致s3和s4分不清。
- 根据存储数组、哈希函数个数、误判率之间的关系，分配空间、个数

符号 |     说明
-- | :--------:
m  |   bit的数量
n  |   要表示个数量
k  | 使用的哈稀函数的数量
f  |    误判率

m是总共开辟的存储空间位数，n是待存储字符串的个数，k是哈希函数的个数，f是误判率。<br>
大致可以认为误判率跟 `m／n` 和 `k` 成反比。当`k`为固定值时，`m/n` 越大，误判率越小。

# 5\. 代码

```
import math
import hashlib


class BloomFilter:
    hash_algorithms = [hashlib.md5, hashlib.sha1,
                       hashlib.sha224, hashlib.sha256]

    def __init__(self, m):
        self.m = m
        self.filter = bytearray(math.ceil(m / 8))

    def _hash(self, value):
        for hash_algorithm in self.hash_algorithms:
            digest = int(hash_algorithm(str(value).encode()).hexdigest(), 16)
            # 进行and运算来保证值在存储范围内
            yield digest & (self.m - 1)

    def add(self, value):
        for digest in self._hash(value):
            # 把对应的bit设置为1
            self.filter[int(digest / 8)] |= 2 ** (digest % 8)

    def __contains__(self, item):
        return all(self.filter[int(digest / 8)] & (2 ** (digest % 8))
                   for digest in self._hash(item))


if __name__ == '__main__':
    bf = BloomFilter(200)
    bf.add('1')
    bf.add(1)
    print(1 in bf, '20' in bf)  # True False
```
