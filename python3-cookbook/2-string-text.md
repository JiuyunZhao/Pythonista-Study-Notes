### 2.1 使用多个界定符分割字符串

一般字符串的分割用str.split足以胜任，但是在复杂的文本中查找分割字符串，正则表达式是无疑是首选的工具，re模块也有一个分割字符串的函数split，需要注意的是正则表达式中如果有括号分组的话，分组的结果也会在结果列表中。

```py
>>> import re
>>> line = 'asdf fjdk; afed, fjek,asdf, foo'
>>> re.split(r'[;,\s]\s*', line)
['asdf', 'fjdk', 'afed', 'fjek', 'asdf', 'foo']
>>> fields = re.split(r'(;|,|\s)\s*', line)  # 分组的内容也会出现在结果里
>>> fields
['asdf', ' ', 'fjdk', ';', 'afed', ',', 'fjek', ',', 'asdf', ',', 'foo']
>>>
```



### 2.3 用Shell通配符匹配字符串

当字符串的匹配一般方法不能满足，但又不想用正则表达式那么复杂，可以考虑使用fnmatch.fnmatch或fnmatch.fnmatchcase，两者都可以使用Unix Shell中常用的通配符匹配字符串，区别在于前者使用的是操作系统的大小写敏感规则，后者则完全按照你写的内容去匹配。

```py
>>> from fnmatch import fnmatch, fnmatchcase
>>> fnmatch('foo.txt', '*.txt')
True
>>> fnmatch('foo.txt', '?oo.txt')
True
>>> fnmatch('Dat45.csv', 'Dat[0-9]*')
True
>>>
```



### 2.13 字符串对齐

字符串对齐是字符串格式化的一部分，对于普通的左对齐，右对齐和居中对齐可以使用字符串的ljust、rjust和center方法，也可以使用内置的format函数和字符串的format方法，文中推荐使用format，因为后者在字符串的格式化功能上更加的丰富和强大。

字符串的对齐工作中似乎并不常用，但我遇到过一个使用场景，就是使用字符串表示的二进制数时，需要用0或1来将字符串补齐为8位或者16位的字符串，这时字符串的对齐功能就排上用场了。

```py
>>> text = 'Hello World'
>>> text.ljust(20)
'Hello World         '
>>> text.rjust(20)
'         Hello World'
>>> text.center(20)
'    Hello World     '
>>> text.rjust(20, '=')
'=========Hello World'
>>> text.center(20, '*')
'****Hello World*****'
>>>
```

```py
>>> # 格式化字符串
>>> format(text, '>20')
'         Hello World'
>>> format(text, '<20')
'Hello World         '
>>> format(text, '^20')
'    Hello World     '
>>> format(text, '=<20s')
'Hello World========='
>>> format(text, '*^20s')
'****Hello World*****'
>>> # 格式化数字
>>> x = 1.2345
>>> format(x, '^10.2f')
'   1.23   '
>>> # 字符串的format方法
>>> '{:>10s} {:>10s}'.format('Hello', 'World')
'     Hello      World'
```



### 2.16 以指定列宽格式化字符串

这个问题在打印信息或者在终端展示信息的时候可能会遇到，此时可以使用textwrap来指定输出列宽。

```py
>>> import textwrap
>>> s = "Look into my eyes, look into my eyes, the eyes, the eyes, the eyes, not around the eyes, don't look around the eyes, look into my eyes, you're under."
>>> print(textwrap.fill(s, 70))
Look into my eyes, look into my eyes, the eyes, the eyes, the eyes,
not around the eyes, don't look around the eyes, look into my eyes,
you're under.
>>> print(textwrap.fill(s, 40))
Look into my eyes, look into my eyes,
the eyes, the eyes, the eyes, not around
the eyes, don't look around the eyes,
look into my eyes, you're under.
>>> print(textwrap.fill(s, 40, initial_indent='    '))
    Look into my eyes, look into my
eyes, the eyes, the eyes, the eyes, not
around the eyes, don't look around the
eyes, look into my eyes, you're under.
>>> print(textwrap.fill(s, 40, subsequent_indent='    '))
Look into my eyes, look into my eyes,
    the eyes, the eyes, the eyes, not
    around the eyes, don't look around
    the eyes, look into my eyes, you're
    under.
>>>
```



### 2.17 在字符串中处理html和xml

在处理HTML或XML文本的时候，想要将如&entity;或&\#code;替换为对应的文本，或者反过来操作，只需要使用对应解析器的工具函数即可，当然，如果你比较熟悉对应的解析器的话或许有更好的方法。

```py
>>> import html
>>> s = 'Elements are written as "<tag>text</tag>".'
>>> print(s)
Elements are written as "<tag>text</tag>".
>>> print(html.escape(s))
Elements are written as &quot;&lt;tag&gt;text&lt;/tag&gt;&quot;.
>>> print(html.escape(s, quote=False))
Elements are written as "&lt;tag&gt;text&lt;/tag&gt;".
>>> 
>>> from html.parser import HTMLParser
>>> s = 'Spicy &quot;Jalape&#241;o&quot.'
>>> p = HTMLParser()
>>> p.unescape(s)
'Spicy "Jalapeño".'
>>> 
>>> from xml.sax.saxutils import unescape
>>> t = 'The prompt is &gt;&gt;&gt;'
>>> unescape(t)
'The prompt is >>>'
>>>
```



### 2.18 字符串令牌解析

令牌化字符串可以使用正则表达式的命名捕获分组来进行，语法为“\(?P&lt;group\_name&gt;\)”，这个问题在解析用户自定义的计算公式字符串时会很有用。

解决这个问题时，可以考虑使用模式对象的scanner方法，并打包到一个生成器中使用。

```py
import re

NAME = r'(?P<NAME>[a-zA-Z_][a-zA-Z_0-9]*)'
NUM = r'(?P<NUM>\d+)'
PLUS = r'(?P<PLUS>\+)'
TIMES = r'(?P<TIMES>\*)'
EQ = r'(?P<EQ>=)'
WS = r'(?P<WS>\s+)'

master_pat = re.compile('|'.join([NAME, NUM, PLUS, TIMES, EQ, WS]))
scanner = master_pat.scanner('foo = 42')
for m in iter(scanner.match, None):
    print(m.lastgroup, m.group())
```

```
NAME foo
WS  
EQ =
WS  
NUM 42
```



