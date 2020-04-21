# re模块\(正则表达式\)

本文是部分内容参考自：[http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html](http://www.cnblogs.com/huxi/archive/2010/07/04/1771073.html)，虽然这篇博客是基于Python2.4的老版本，但是基础的Python正则表达式还是非常全的。

**通用性：**正则表达式是用于处理字符串的功能强大的工具，但它并不是Python所独有的，许多编程语言都支持正则表达式，用法也都区别不大。

**贪婪量词：**Python中的数量词默认是贪婪的，总是尝试匹配尽可能多的字符；非贪婪的则相反，总是尝试匹配尽可能少的字符（例如：正则表达式"ab\_"如果用于查找"abbbc"，将找到"abbb"；而如果使用非贪婪的数量词"ab\_?"，将找到"a"）。

**原生字符串：**与大多数编程语言相同，正则表达式里使用"\"作为转义字符，这就可能造成反斜杠困扰。假如你需要匹配文本中的字符"\"，那么使用编程语言表示的正则表达式里将需要4个反斜杠"\"：前两个和后两个分别用于在编程语言里转义成反斜杠，转换成两个反斜杠后再在正则表达式里转义成一个反斜杠。Python里的原生字符串很好地解决了这个问题，这个例子中的正则表达式可以使用r"\"表示。

**re模块中的常用方法：**Python中正则表达式的应用在re模块中，re模块中的方法使用正则表达式来匹配字符串（以下描述中pattern为正则表达式，string为需要匹配查找的字符串）：

* **re.compile\(pattern\)：**编译正则表达式，返回编译后的模正则表达式对象，该对象拥有match、search等方法，并且某些方法可以指定匹配字符串的开始位置和结束位置。
* **re.match\(pattern, string\)：**匹配字符串的开头，成功则返回匹配对象，否则返回None。compile返回的对象的match方法可以指定匹配字符串的开始位置和结束位置。
* **re.search\(pattern, string\)：**从字符串开头开始查找匹配，直到匹配成功，则不再往后继续查找匹配，成功返回匹配对象，否则返回None。compile返回的对象的search方法可以指定匹配字符串的开始位置和结束位置。
* **re.findall\(pattern, string\)：**查找匹配字符串中所有内容，返回查找成功的字符串的列表，如果字符串中没有匹配成功的内容，则返回空列表，如果pattern中有括号\(\)分组，则列表中只返回匹配成功后的分组中的字符串内容。compile返回的对象的findall方法可以指定匹配字符串的开始位置和结束位置。
* **re.sub\(pattern, repl, string, count=0\)：**使用正则表达式pattern在字符串string中匹配查找，匹配查找成功后使用新字符串repl替换掉匹配成功的字符串，并返回，count为替换次数，默认0不是替换0次，而是替换所有。repl也可以是函数，该函数的参数为匹配对象，且应该返回一个字符串用于替换匹配成功的字符串，具体示例见下方的代码。
* **re.split\(pattern, string, maxsplit=0\)：**使用匹配成功后的字符串作为“分割符”，返回分割后的字符串列表，maxsplit为分割的次数，默认0不是分割0次，而是分割所有。
* **re.finditer\(pattern, string\)：**返回全部查找结果的迭代器，每个迭代对象为匹配对象，可以使用group\(\)和groups\(\)获取匹配成功的结果，如果没有匹配成功的字符串，则返回一个空的迭代器（不是None）。compile返回的对象的finditer方法可以指定匹配字符串的开始位置和结束位置。
* **group\(\*args\)：**匹配对象的group\(\)默认返回匹配成功的整个字符串，如果正则表达式中有括号分组，可以指定返回第几个分组结果，指定时从1开始计数，比如group\(2\)返回匹配成功的字符串中的第二个分组结果；也可以指定返回多个分组结果，结果以元组的形式返回，比如group\(1, 2, 3\)以元组返回匹配成功的字符串中的第一、第二和第三个分组结果；如果正则表达式中给某个或某几个分组指定了别名，则可以使用别名来代替分组编号来获取匹配成功的对应分组结果。
* **groups\(\)：**匹配对象的groups\(\)以元组的形式返回匹配成功后括号中分组的内容，相当于group\(1,..., n\)，但是正则表达式中没有括号分组，则返回空元组，即使匹配成功，也是返回空元组。

##### 正则表达式常用语法：

| **语法** | **说明** | **正则表达式实例** | **参与匹配的字符串** | **匹配成功的结果字符串** |
| :--- | :--- | :--- | :--- | :--- |
| . | 匹配任意字符，除换行符“\n”外 | r'a.' | 'abc' | 'ab' |
| \ | 转义字符，可以用来匹配特殊的字符 | r'a\\*' | 'a\*b' | 'a\*' |
|  | 特殊字符也可以放在\[\]字符集中来匹配 | r'a\[\*\]' | 'a\*b' | 'a\*' |
| \[\]或\[^...\] | 字符集\[\]，匹配字符集中的一个字符，从开始匹配直到匹配成功；开头加上“^”表示取反，即只要不是字符集中列出的字符都可匹配成功；可以在\[\]列出全部想要匹配的字符，也可以列出字符的范围，如\[a-c\]或\[abc\]；所有特殊字符放在字符集中都失去在正则表达式中的原有的意义 | r'a\[efg\]' | 'afggg' | 'af' |
| \d | 匹配单个数字字符：0到9 | r'a\d' | 'a345b' | 'a3' |
| \D | 匹配非数字字符 | r'a\D' | 'ab3' | 'ab' |
|  |  | r'a\[^\d\]' | 'ab3' | 'ab' |
| \s | 匹配空白字符：空格，\t，\r，\n，\f，\v | r'a\s' | 'a\n' | 'a\n' |
| \S | 匹配非空白字符 | r'a\S' | 'ab' | 'ab' |
|  |  | r'a\[^\s\]' | 'ab' | 'ab' |
| \w | 匹配单词字符：\[a-zA-Z0-9\_\] | r'a\w' | 'a9' | 'a9' |
| \W | 匹配非单词字符 | r'a\W' | 'a+' | 'a+' |
| \* | 匹配前一个字符0到无限次 | r'ab\*' | 'a' | 'a' |
| + | 匹配前一个字符1到无限次 | r'ab+' | 'abbbcde' | 'abbb' |
| ? | 匹配前一个字符0次或1次 | r'ab?' | 'abbbcde' | 'ab' |
| {m} | 匹配前一个字符m次 | r'ab{2}' | 'abbbcde' | 'abb' |
| {m,n} | 匹配前一个字符m次至n次，m和n可省略，{m,}匹配前一个字符m次至无限次，{,n}匹配前一个字符0次至n次 | r'ab{1,2}' | 'abbbbcde' | 'abb' |
| \*？或+?或??或{m,n}? | 使\*、+、?和{m,n}的匹配变成非贪婪模式（Python默认是贪婪模式） | r'ab+?' | 'abbbcde' | 'ab' |
| ^ | 匹配字符串的开头（用在字符集中\[\]表示取反） | r'^ab' | 'abbb' | 'ab' |
| $ | 匹配字符串的结尾 | r'cd$' | 'abcd' | 'cd' |
| \(\) | 括号中的内容被作为分组，\(\)后可以接数量词 | r'a\(def\){2}b' | 'adefdefbcc' | 'adefdefb' |

##### 简单示例：

```text
>>> import re
>>> pattern = re.compile('python') # compile将字符串当做正则表达式来编译
>>> result = pattern.search('hello python!')
>>> result
<_sre.SRE_Match object; span=(6, 12), match='python'>
>>> result.group()
'python'
>>>
>>> # match方法
>>> result = re.match('a', 'abc') # match是从字符串的开头开始匹配
>>> result
<_sre.SRE_Match object; span=(0, 1), match='a'>
>>> result.group() # 并不直接返回匹配成功的字符串，需要使用group()方法
'a'
>>> result = re.match('a', 'dabc')
>>> result
>>> result.group() # 没有匹配成功
Traceback (most recent call last):
  File "<pyshell#6>", line 1, in <module>
    result.group()
AttributeError: 'NoneType' object has no attribute 'group'
>>>
>>> # search方法
>>> result = re.search('python', 'abcpythondef') # 在字符串的全文中搜索匹配一次，同样也不会直接返回匹配成功的字符串
>>> result
<_sre.SRE_Match object; span=(3, 9), match='python'>
>>> result.group()
'python'
>>>
>>> # findall方法
>>> result = re.findall('python', 'abc python def python ghi')
>>> result
['python', 'python']
>>> # sub方法
>>> result = re.sub('c', 'z', 'click', 1) # 使用匹配成功的字符串替换成指定的字符串，参数依次为正则表达式，匹配成功后要去替换的字符串，原字符串，替换次数
>>> result # 返回替换后的字符串
'zlick'
>>> def sub_no_use_match(match_obj): # 用不到模式对象match_obj，但是该函数必须有这个参数
    return '36'
>>> re.sub(r'\d+', sub_no_use_match, 'Python27')  # 以函数返回的字符串替换匹配成功的字符串
'Python36'
>>> def sub_use_match(match_obj):  # 使用模式对象match_obj来返回最终的字符串
    return match_obj.group() + 'hahahaha'
>>> re.sub(r'\d+', sub_use_match, 'Python27')
'Python27hahahaha'
>>>
>>> # split方法
>>> result = re.split('a', '1a2a3a4guyuyun') # 将匹配成功的字符串用作字符串分隔符，返回分隔后的字符串列表
>>> result
['1', '2', '3', '4guyuyun']
>>>
>>> # group和groups方法的区别
>>> result = re.search('(python)python(\d{1,3})', 'pythonpython22')
>>> result.groups() # groups方法是匹配pattern中括号里的格式，以元组的形式返回括号里匹配成功的字符串
('python', '22')
>>> result.group() # group是正常的匹配，返回匹配成功的字符串
'pythonpython22'
>>>
>>> string = 'python'
>>> import re
>>> result = re.search(r'(yt)h(o)', string)
>>> result
<_sre.SRE_Match object at 0x000000000293DE88>
>>> result.group()
'ytho'
>>> result.group(0)  # 参数0无效
'ytho'
>>> result.group(1)  # 从1开始计数
'yt'
>>> result.group(2)
'o'
>>> result.group(1, 2)
('yt', 'o')
>>>
>>> result.groups()
('yt', 'o')
>>> result.groups(0)  # 传入参数无效
('yt', 'o')
>>> result.groups(1)
('yt', 'o')
>>>
>>> # finditer方法
>>> string = 'one11python, two22, three33python '
>>> result = re.finditer(r'(\d+)(python)', string)
>>> for p in result:
    print(p.group())
11python
33python
>>> for p in result:
    print(p.group(2))
python
python
>>> for p in result:
        print(p.groups())  # 若是pattern中没有括号，则返回的是每个迭代器对应的空元组。
('11', 'python')
('33', 'python')
```



