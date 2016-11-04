---
title: 中文分词
date: '2016/4/3 20:00:00'
tags:
  - 分词
  - 数学之美
---

英文分词大体用空格就可以分开了，而中文分词无法使用该办法。<br>
中文分词最简单的办法就是查字典。它其实就是把一个句子从左往右扫描一遍， 遇到字典里有的词就标识出来，最后取长度最长的那个词，如果都不在字典里， 就分割成单字词。

```python
dictionary = ['中', '中国', '航天', '官员', '应邀', '到', '美国', '与', '太空', '总署', '开会']
text1 = '中国航天官员应邀到美国与太空总署官员开会'
text2 = '中国航天官员应邀到美国与太空总署官员开会去了'
```

```
def look_up_dictionary(text, dictionary):
    cut_point = 0
    words = []
    text_length = len(text)
    while cut_point < text_length:
        step = 0
        i = 0
        text_remain_length = len(text[cut_point:])
        while i < text_remain_length:
            i += 1
            word = text[cut_point:cut_point+i]
            if word in dictionary:
                step = i

        # 字典里没有或者就是单字词
        if step == 0:
            step = 1

        words.append(text[cut_point:cut_point+step])
        cut_point += step
    return words    

print('/'.join(look_up_dictionary(text1, dictionary)))
print('/'.join(look_up_dictionary(text2, dictionary)))
```

输出为

```
中国/航天/官员/应邀/到/美国/与/太空/总署/官员/开会
中国/航天/官员/应邀/到/美国/与/太空/总署/官员/开会/去/了
```

字典里有 `中` , `中国` , 就取长度最长的 `中国` ;<br>
字典里没有 `去了`, 就取单字词 `去` 和 `了` 。<br>
这样就可以解决大半的分词问题。

那么字典去哪里找，这里分享一个网站，<http://www.mdbg.net/。><br>
这里有个项目叫 `CC-CEDICT`， 可以下载到字典。

```python
def get_dictionary(filename):
    dictionary = []
    with open(filename) as f:
        for line in f:
            if line.startswith('#') or line.startswith('%'):
                continue
            word = line.split()[1]
            dictionary.append(word)
    return dictionary      


dictionary = get_dictionary('cedict_ts.u8')
print(len(dictionary))
print('/'.join(look_up_dictionary('上海大学城书店', dictionary)))
```

输出为

```
114350
上海大学/城/书店
```

`CC-CEDICT` 目前大约有 `11万` 的字典， 里面有繁体，简体， 上面代码只要了简体。
