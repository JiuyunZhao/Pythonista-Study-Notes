### 10.2 控制模块被全部导入的内容

尽管强烈反对使用from xxx import \*的形式来进行导入，但是当你真的需要用到这种形式时，需要注意以下几点：

* 这种形式不会导入以下划线开头的变量。
* 如果模块中（包括\_\_init\_\_.py）定义了\_\_all\_\_变量（列表类型），则只会导入\_\_all\_\_中定义的内容，即便\_\_all\_\_是一个空列表，而且列表中的模块名需要是字符串类型。
* 如果\_\_all\_\_中包含了未定义的名称，则会引起报错。



### 10.4 将模块分割成多个文件

当某个包中的模块太多，或者一个文件中的类、函数等太多之后，可以将它们在逻辑上统一成一个模块，以减小使用者的负担。

如有一个mymodule.py：

```py
class A:
    def spam(self):
        print('A.spam')

class B(A):
    def bar(self):
        print('B.bar')
```

新建一个包，将A和B分别定义在a.py和b.py两个文件：

```
mymodule/
    __init__.py
    a.py
    b.py
```

```py
# a.py
class A:
    def spam(self):
        print('A.spam')
```

```py
# b.py
from .a import A
class B(A):
    def bar(self):
        print('B.bar')
```

```py
# __init__.py
from .a import A
from .b import B
```

使用示例：

```py
>>> import mymodule
>>> a = mymodule.A()
>>> a.spam()
A.spam
>>> b = mymodule.B()
>>> b.bar()
B.bar
>>>
```

以上\_\_init\_\_.py中的写法有一个缺点，就是会在导入时会一次性将全部模块都加载完成，可以考虑如下延迟加载的方案，但是也需要注意延迟加载这种方案在继承和类型检查时可能会被中断：

```py
# __init__.py
def A():
    from .a import A
    return A()

def B():
    from .b import B
    return B()
```



### 10.5 利用命名空间导入目录分散的代码

当你想给程序添加插件或者附加包时，可以考虑如下方案：

如有两个插件的代码目录结构如下，需要注意的是，这个目录结构中任何一层目录都是不能有\_\_init\_\_.py的：

```
foo-package/
    spam/
        blah.py

bar-package/
    spam/
        grok.py
```

```py
>>> import sys
>>> sys.path.extend(['foo-package', 'bar-package'])
>>> import spam.blah
>>> import spam.grok
>>>
>>> import spam
>>> spam.__path__
_NamespacePath(['foo-package/spam', 'bar-package/spam'])
>>>
```



### 10.7 运行目录或压缩文件

当使用Python去运行一个目录或者压缩文件时，如果目录或者压缩文件中含有\_\_main\_\_.py，则Python会将这个文件作为程序主文件来运行，通常，遇到这种情形，建议写一个简单的shell脚本来运行程序。

程序文件目录结构：

```
myapplication/
    spam.py
    bar.py
    grok.py
    __main__.py
```

运行程序目录：

```
bash % python3 myapplication
```

运行压缩文件：

```
bash % ls
spam.py bar.py grok.py __main__.py
bash % zip -r myapp.zip *.py
bash % python3 myapp.zip
... output from __main__.py ...
```

shell脚本：

```
#!/usr/bin/env python3 /usr/local/bin/myapp.zip
```



### 10.10 通过字符串名导入模块

当你想通过字符串来导入某个模块时，可以使用importlib.import\_module\(\)来进行导入，它返回的是生成的模块对象，你只需要将它保存在一个变量中，就可以像正常的模块一样使用它。

如果字符串中包含相对导入，需要给一个额外的参数，如\_\_package\_\_。

相对于使用内置的\_\_import\_\_\(\)来达到同样的目的，import\_module更加通用，所以推荐使用后者。

```py
>>> import importlib
>>> math = importlib.import_module('math')
>>> math.sin(2)
0.9092974268256817
>>> mod = importlib.import_module('urllib.request')
>>> u = mod.urlopen('http://www.python.org')
>>>
```

```py
import importlib
# Same as 'from . import b'
b = importlib.import_module('.b', __package__)
```



