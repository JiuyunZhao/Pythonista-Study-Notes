# 输入输出

**输入函数input\(\)和raw\_input\(\)**

在Python3.x中只有input\(\)作为输入函数，会将输入内容自动转换str类型。

在Python2.x中有input\(\)和raw\_input\(\)两个输入函数，对于input\(\)函数，你输入的是什么类型，他就传入什么类型；raw\_input\(\)和3.x中的input\(\)作用一样。

```text
>>> a = input()
>>> type(a)
<type 'int'>
>>> b = input()
'3'
>>> type(b)
<type 'str'>
>>> c = raw_input()
>>> type(c)
<type 'str'>
```

**输出函数print\(\)**

在Python2.7中可以不用括号（用一个空格间隔），但是在Python3.x中统一为print\(\)，必须有括号。

print\(self, \*args, sep=' ', end='\n', file=None\)

* \*args：可以输出多个参数。
* sep：指定输出的多个参数之间的间隔符，默认是一个空格。
* end：指定最后一个参数输出之后输出的字符，默认是换行符。

```text
>>> print('a', 'b', 'c', sep='-', end='#')
a-b-c#
>>>
```

