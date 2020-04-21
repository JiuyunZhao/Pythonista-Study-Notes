### 7.2 只接受关键字参数的函数

有时候为了代码可读性或者其他原因，我们需要在使用函数的时候强制要求调用者使用关键字参数，这时候只需要在某个\*参数或单个\*后面定义关键字参数即可。

```py
def print_ax(a, x):
    print(a, x)


def print_x(a, *, x):
    print(a, x)


print_ax(111, 222)
# 如果这样写就会报错：print_x(111, 222)
print_x(111, x=222)
```



### 7.7 匿名函数捕获变量值

lambda表达式的参数值是在运行时绑定的，而不是定义时绑定的，这跟def定义参数默认值是不同的，如果需要lambda表达式在定义时就绑定值，可以使用参数默认值的写法。

```py
>>> x = 10
>>> a = lambda y: x + y
>>> x = 20
>>> b = lambda y: x + y
>>> # 下面两个执行结果是一样的
>>> a(10)
30
>>> b(10)
30
>>> x = 30
>>> a(10)
40
>>> b(10)
40
>>> # 使用参数默认值的方式
>>> a = lambda y, x=x: x + y
>>> b = lambda y, x=x: x + y
>>> a(10)
40
>>> b(10)
40
>>> 
```



### 7.8 减少可调用对象的参数个数

当调用某个函数的时候可能你并不需要传入全部参数值，或者只能传入部分参数值，就可以使用functools.partial\(\)。

**用法示例：**

```py
>>> from functools import partial
>>> def spam(a, b, c, d):
    print(a, b, c, d)

    
>>> s1 = partial(spam, 1)  # a = 1
>>> s1(2, 3, 4)
1 2 3 4
>>> s2 = partial(spam, d=42) # d = 42
>>> s2(1, 2, 3)
1 2 3 42
>>> s3 = partial(spam, 1, 2, d=42) # a = 1, b = 2, d = 42
>>> s3(3)
1 2 3 42
>>> 
```

**排序使用示例：**

```py
"""将列表中的点根据距离基点的距离进行排序"""
import math
from functools import partial

# 基点
pt = (4, 3)
points = [(1, 2), (3, 4), (5, 6), (7, 8)]


def distance(p1, p2):
    """返回两个点之间的距离"""
    x1, y1 = p1
    x2, y2 = p2
    return math.hypot(x2 - x1, y2 - y1)


points.sort(key=partial(distance, pt))
print(points)  # [(3, 4), (1, 2), (5, 6), (7, 8)]
```

**回调函数使用示例：**

```py
import logging
from multiprocessing import Pool
from functools import partial


def output_result(result, log=None):
    if log is not None:
        log.debug('Got: %r', result)


def add(x, y):
    return x + y


if __name__ == '__main__':
    # logging模块用于打印日志
    logging.basicConfig(level=logging.DEBUG)
    log = logging.getLogger('test')

    # 异步调用另一个函数，并将log对象传入进去
    p = Pool()
    p.apply_async(add, (3, 4), callback=partial(output_result, log=log))
    p.close()
    p.join()
```



