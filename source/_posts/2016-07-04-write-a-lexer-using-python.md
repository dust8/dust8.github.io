---
title: 用python写个词法解析器
date: 2016-07-04 05:15:31
tags:
  - Lexer
  - 解析器
---

想了解编译器原理的朋友, 不要在看龙书或者其他的书了, 强烈推荐先看《编程语言实现模式》.    
英文名叫 Language Implementation Patterns - Create Your Own Domain-Specific
and General Programming Languages. 它实战性非常强, 在也不会被催眠了.    

来实现一个简单的词法解析器, 解析简单的数字运算, 只支持整数和加减.    
原理我不多说, 想了解的就去看我推荐 《编程语言实现模式》.    

```python
class Token:
    def __init__(self, typ, value):
        self.typ = typ
        self.value = value

    def __str__(self):
        return '<type="%s", value="%s">' % (self.typ, self.value)


class Lexer:
    def __init__(self, text):
        '''
        text:输入字符

        p:当前输入字符的下标
        c:当前字符
        eof:结束标志
        '''
        self.text = text
        self.text_len = len(text)
        self.p = 0
        self.c = self.text[self.p]
        self.eof = 'eof'

    def consume(self):
        '''向前移动一个字符, 检测输入是否结束'''
        self.p += 1
        if self.p >= self.text_len:
            self.c = self.eof
        else:
            self.c = self.text[self.p]

    def match(self, x):
        '''确保x是输入流中的下一个字符'''
        if self.c == x:
            self.consume()
        else:
            raise ValueError('expecting %s ; found %s' % (x, self.c))

    def next_token(self):
        raise NotImplementedError()


class ListLexer(Lexer):
    def __init__(self, text):
        super().__init__(text)

    def next_token(self):
        while self.c != self.eof:
            if (self.c == ' ' or self.c == '\t' or self.c == '\n' or
                    self.c == '\r'):
                self.r_ws()
                continue
            elif self.c == '+':
                self.consume()
                return Token('OP', '+')
            elif self.c == '-':
                self.consume()
                return Token('OP', '-')
            else:
                if self.is_number():
                    return self.r_number()
                raise ValueError('invalid character: %s' % (self.c))

        return Token('EOF', 'eof')

    def r_number(self):
        buf = []
        while self.is_number():
            buf.append(self.c)
            self.consume()
        return Token('NUMBER', ''.join(buf))

    def r_ws(self):
        while (self.c == ' ' or self.c == '\t' or self.c == '\n' or
               self.c == '\r'):
            self.consume()

    def is_number(self):
        return '0' <= self.c <= '9'


if __name__ == '__main__':
    text = '''01+
    22'''
    lexer = ListLexer(text)
    t = lexer.next_token()
    while t.typ != 'EOF':
        print(t)
        t = lexer.next_token()
    print(t)
```
