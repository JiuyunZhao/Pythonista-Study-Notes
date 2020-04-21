### 3.2 执行精确的浮点数运算

Python的浮点数计算是存在误差的，但是在大多数领域这种误差是被允许的，并且在性能上也是最快的，但是如果你的程序需要提供非常精确的计算，并且可以容忍一定的性能损耗，比如金融领域的计算，那么可以使用Decimal模块，需要注意的是，Decimal模块需要用字符串的形式表示数字。

另外提一下，如果是进行舍入运算或其他计算操作，可以考虑使用内置函数round或者math模块中的方法。

```py
>>> a = 4.2
>>> b = 2.1
>>> a + b
6.300000000000001
>>> (a + b) == 6.3
False
>>> (a + b) > 6.3
True
>>> 
>>> from decimal import Decimal
>>> a = Decimal('4.2')
>>> b = Decimal('2.1')
>>> a + b
Decimal('6.3')
>>> print(a + b)
6.3
>>> (a + b) == Decimal('6.3')
True
>>> 
>>> # decimal的舍入运算
>>> from decimal import localcontext
>>> a = Decimal(1.3)
>>> a
Decimal('1.3000000000000000444089209850062616169452667236328125')
>>> a = Decimal('1.3')
>>> a
Decimal('1.3')
>>> b = Decimal('1.7')
>>> print(a / b)
0.7647058823529411764705882353
>>> with localcontext() as ctx:
    ctx.prec = 3
    print(a / b)

    
0.765
>>>
```



### 3.3 数字的格式化输出

数字的格式化操作，包括浮点数和Decimal数字对象，可以使用内置的format函数或字符串的format方法，指定宽度和精度的一般形式为\[&lt;&gt;^\]?width\[,\]?\(.digits\)?，?表示它前面的部分是可选的，&lt;&gt;^为对齐方式，width为宽度，逗号为千位符，.digits为精度。

```py
>>> x = 1234.56789
>>> format(x, '0.2f')
'1234.57'
>>> format(x, '>10.1f')
'    1234.6'
>>> format(x, '<10.1f')
'1234.6    '
>>> format(x, '^10.1f')
'  1234.6  '
>>> format(x, ',')
'1,234.56789'
>>> format(x, '0,.1f')
'1,234.6'
>>> format(x, 'e')
'1.234568e+03'
>>> format(x, '0.2E')
'1.23E+03'
>>> 'The value is {:0,.2f}'.format(x)
'The value is 1,234.57'
>>>
```



### 3.4 二八十六进制整数

如果想要将整数转换为二进制、八进制或十六进制的数字字符串，使用内置的bin\(\)、oct\(\)或hex\(\)函数即可，或者使用format函数进行转换，当然，反过来，想要将对应进制的数转换为十进制的整数，直接使用int\(\)即可。

```py
>>> x = 1234
>>> # 带有对应进制表示前缀：0b，0o，0x
>>> bin(x)
'0b10011010010'
>>> oct(x)
'0o2322'
>>> hex(x)
'0x4d2'
>>> # 没有对应进制表示前缀：0b，0o，0x
>>> format(x, 'b')
'10011010010'
>>> format(x, 'o')
'2322'
>>> format(x, 'x')
'4d2'
>>> # 转换为十进制
>>> int('4d2', 16)
>>> int('10011010010', 2)
>>> int('2322', 8)
>>>
```



### 3.5 字节到大整数的打包与解包

字节字符串到整数之间的转换并不常用，但是遇到了，也可以考虑这种解决方案，即使用int.from\_bytes\(\)方法和int.to\_bytes\(\)方法。

```py
>>> data = b'\x00\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'
>>> int.from_bytes(data, 'little')
>>> int.from_bytes(data, 'big')
>>> x = 94522842520747284487117727783387188
>>> x.bit_length() / 8
14.625
>>> x.to_bytes(15, 'little')
b'4\x00#\x00\x01\xef\xcd\x00\xab\x90x\x00V4\x12'
>>> x.to_bytes(15, 'big')
b'\x124V\x00x\x90\xab\x00\xcd\xef\x01\x00#\x004'
>>>
```



### 3.15 字符串转换为日期

将字符串转为日期，可以使用datetime中的strptime\(\)方法，但是专门记下这小节，是因为文中说这个方法是用纯Python实现，并且效率并不是很好，如果是已知字符串的格式，推荐自己写一套解析转换的代码，我在平时的使用中也是有这种体验，特别是转换次数多了后感受就特别明显。

```py
>>> from datetime import datetime
>>> text = '2012-09-20'
>>> datetime.strptime(text, '%Y-%m-%d')
datetime.datetime(2012, 9, 20, 0, 0)
>>> 
```



