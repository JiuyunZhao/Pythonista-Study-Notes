### 5.1 读写文本数据

使用open打开文本文件时，Python会默认将换行符转换为\n，如果不想Python进行默认的转换，可以使用参数newline=''以保留原来的换行符。

当使用默认的编码不能正确读取数据时，open还有一个encoding参数用来指定以哪种编码打开文件。

```py
>>> f = open('hello.txt', 'rt')
>>> f.read()
'hello world!\n'
>>> g = open('hello.txt', 'rt', newline='')
>>> g.read()
'hello world!\r\n'
>>>
```



### 5.4 读写字节数据

使用open函数的rb或wb进行二进制的数据读写的时候，比如图片或音频数据，需要注意以下几点：

* 读取二进制数据时，返回的数据都是字节字符串格式的，而不是文本字符串格式。
* 写入二进制数据的时候，也需要保证是以字节形式的对象进行写入。
* 在二进制格式的文件中读取或写入文本数据时，需要进行相应的解码和编码操作。
* 对于字节字符串，通过下标索引或迭代的时候，返回的是对应的字节值，即一个数字，而不是字节字符串。

```py
with open('somefile.bin', 'rb') as f:
    # 读取出来的是字节字符串
    data = f.read()
    # 解码为文本字符串
    text = data.decode('utf-8')


with open('somefile.bin', 'wb') as f:
    # 以字节对象的形式写入
    f.write(b'Hello World')
    # 编码后写入文件
    text = 'Hello World'
    f.write(text.encode('utf-8'))
```

```py
>>> t = 'hello world'
>>> t[0]
'h'
>>> for c in t:
    print(c)

    
h
e
l
l
o
 
w
o
r
l
d
>>> b = b'hello world'
>>> b[0]
104
>>> for c in b:
    print(c)

    
104
101
108
108
111
32
119
111
114
108
100
>>> 
```



### 5.7 读写压缩文件

读写压缩文件还是比较常用的，Python对于gzip和bz2格式的压缩文件操作还是有很好的支持的，即gzip模块和bz2模块，这两个模块有和内置open函数一样的函数，包括参数的使用也是一样的，但是需要注意的是这两个模块默认是以二进制来打开的，所以你想要读写文本数据时就需要指定对应的rt和wt模式了。

在写入压缩数据时，这两个模块还有一个压缩比参数compresslevel，默认为最高级别9，等级越低性能越好，但是数据压缩程度也越低。

```py
import gzip

# gzip和bz2的open函数使用方法一样，就只贴gzip的实例代码了

# 默认一个二进制模式打开，文本文件需要指定rt模式
with gzip.open('somefile.gz', 'rt') as f:
    text = f.read()

with gzip.open('somefile.gz', 'wt') as f:
    f.write(text)
```



