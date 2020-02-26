###  9.2 创建装饰器时保留函数元信息

通常，在定义装饰器函数时，总是应该记得使用functools.wraps来注解被包装的函数，虽然在平时不加这个wraps装饰器感觉也工作的很好，但其实不加的话被装饰函数的元信息是被丢失了的，比如\_\_name\_\_、\_\_doc\_\_等信息，为了被装饰函数的元信息不被丢失，就需要加上这个wraps装饰器。

需要注意的是wraps应该放在包装原始函数的那个函数定义之上，即参数为func的函数的返回值函数定义之上，特别是带参数的装饰器定义之中更要注意。

```py
import time
from functools import wraps


def timethis(func):
    """普通的装饰器"""

    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end - start)
        return result

    return wrapper


def timethis_wraps(func):
    """有wraps的装饰器"""

    @wraps(func)
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        end = time.time()
        print(func.__name__, end - start)
        return result

    return wrapper


@timethis
def countdown(n):
    """timethis装饰函数"""
    while n > 0:
        n -= 1


@timethis_wraps
def countdown_wraps(n):
    """timethis_wraps装饰函数"""
    while n > 0:
        n -= 1


countdown(1000)
print(countdown.__name__)
print(countdown.__doc__)

countdown_wraps(1000)
print(countdown_wraps.__name__)
print(countdown_wraps.__doc__)
```

```
countdown 0.0
wrapper
None
countdown_wraps 0.0
countdown_wraps
timethis_wraps装饰函数
```



### 9.6 带可选参数的装饰器

你想定义一个装饰器，但使用的时候既可以传递参数给它，也可以不传任何参数给它，那么可以参考以下示例，利用functools.partial来实现。

```py
import logging
from functools import wraps, partial


# 星号*的作用是迫使其后面的参数，即level，name，message三个参数，都必须使用关键字参数的形式传参
def logged(func=None, *, level=logging.DEBUG, name=None, message=None):
    if func is None:
        # 此处partial的作用为返回一个函数，但它的level，name，message三个参数是已经指定好了的
        # 使用的时候只需要传入剩下的参数即可，相当于：def logged(func=None):...
        return partial(logged, level=level, name=name, message=message)

    logname = name if name else func.__module__
    log = logging.getLogger(logname)
    logmsg = message if message else func.__name__

    # 使用wraps装饰器保证func函数的元信息是正确完整的
    @wraps(func)
    def wrapper(*args, **kwargs):
        log.log(level, logmsg)
        return func(*args, **kwargs)

    return wrapper


# 不传参示例
@logged
def add(x, y):
    return x + y


# 传参示例
@logged(level=logging.CRITICAL, name='example')
def spam():
    print('Spam!')


print(add(2, 3))  # 输出：5
spam()  # 输出：Spam!
```



### 9.8 将装饰器定义为类的一部分

如果需要在装饰器中记录信息或绑定信息时，那么可以考虑将装饰器定义在类中，需要注意的是装饰器方法为类方法和实例方法时使用的也是对应的类或者实例，而且functools.wraps包装的函数是不用传递额外的cls或self参数的。

```py
from functools import wraps


class A:
    # 实例方法
    def decorator1(self, func):
        # wrapper函数不用定义参数self
        @wraps(func)
        def wrapper(*args, **kwargs):
            print('Decorator 1')
            return func(*args, **kwargs)

        return wrapper

    # 类方法
    @classmethod
    def decorator2(cls, func):
        # wrapper函数不用定义参数cls
        @wraps(func)
        def wrapper(*args, **kwargs):
            print('Decorator 2')
            return func(*args, **kwargs)

        return wrapper


a = A()


# 使用实例进行调用
@a.decorator1
def spam():
    pass


# 使用类进行调用
@A.decorator2
def grok():
    pass
```



### 9.21 避免重复的属性方法

如果在类中定义许多重复的对于属性的逻辑代码，可以考虑如下示例进行简化代码：需要检查name属性是否为str类型，检查age属性是否为int类型。

**普通方法定义属性的检查：**

```py
class Person:
    def __init__(self, name ,age):
        self.name = name
        self.age = age

    @property
    def name(self):
        return self._name

    @name.setter
    def name(self, value):
        if not isinstance(value, str):
            raise TypeError('name must be a string')
        self._name = value

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if not isinstance(value, int):
            raise TypeError('age must be an int')
        self._age = value
```

**简化后的代码：**

```py
# 此函数返回一个属性对象，用于检查赋的值是否为指定的类型
# 在类中使用这个函数时，跟把这个函数中的代码放到类中定义是一样的，作用就只是把重复使用的代码提出来了
def typed_property(name, expected_type):
    storage_name = '_' + name

    @property
    def prop(self):
        return getattr(self, storage_name)

    @prop.setter
    def prop(self, value):
        if not isinstance(value, expected_type):
            raise TypeError('{} must be a {}'.format(name, expected_type))
        setattr(self, storage_name, value)

    return prop


class Person:
    name = typed_property('name', str)
    age = typed_property('age', int)

    def __init__(self, name, age):
        self.name = name
        self.age = age
```

**使用functools.partial改良的代码：**

```py
from functools import partial

String = partial(typed_property, expected_type=str)
Integer = partial(typed_property, expected_type=int)


class Person:
    name = String('name')
    age = Integer('age')

    def __init__(self, name, age):
        self.name = name
        self.age = age
```



### 9.22 定义上下文管理器的简单方法

自定义上下文管理器有两种方式，一种是使用contextlib.contextmanager装饰器，但是这种方式应该只用来定义函数的上下文管理器，如果是对象，比如文件、网络连接或者锁等，就应该使用另一种方式，即给对应的类实现\_\_enter\_\_\(\)方法和\_\_exit\_\_\(\)方法，在进入with语句时会调用\_\_enter\_\_\(\)方法，退出时调用\_\_exit\_\_\(\)方法，对象的方式容易懂一点，所以这里就只给出函数方式的示例。

```py
from contextlib import contextmanager


@contextmanager
def list_transaction(orig_list):
    working = list(orig_list)
    yield working
    # 如果没有报错，对列表orig_list的任何修改才会生效
    orig_list[:] = working


items = [1, 2, 3]
with list_transaction(items) as working:
    working.append(4)
    working.append(5)
print(items)

items = [1, 2, 3]
with list_transaction(items) as working:
    working.append(4)
    working.append(5)
    raise RuntimeError('oops！')
print(items)
```

```
[1, 2, 3, 4, 5]
Traceback (most recent call last):
  File "Z:/Projects/Daily Test/test.py", line 120, in <module>
    raise RuntimeError('oops！')
RuntimeError: oops！
```



### 9.23 在局部变量域中执行代码

在使用exec\(\)时，都应该想一下是否有更好的方案或者可以替代的方案。当然，如果是必须要使用exec\(\)，那么需要注意exec\(\)执行时，它使用的局部变量域是拷贝自真实局部变量域，而不是直接使用的真实局部变量域，如果需要获取exec局部变量域中的值，可以使用locals\(\)，locals\(\)返回一个真实局部变量域的拷贝，也是指向的exec\(\)拷贝的局部变量域。

**参考以下测试代码和示例：**

```py
>>> a = 13
>>> exec('b = a + 1')
>>> b
14
```

```py
>>> def test():
    a = 13
    exec('b = a + 1')
    print(b)

    
>>> # b并没有在真实局部变量域中，所以会报错
>>> test()
Traceback (most recent call last):
  File "<pyshell#3>", line 1, in <module>
    test()
  File "<pyshell#1>", line 4, in test
    print(b)
NameError: name 'b' is not defined
>>> 
```

```py
>>> def test1():
    x = 0
    exec('x += 1')
    print(x)

    
>>> test1()  # x的值并没有被修改
0
>>> 
```

```py
>>> def test2():
    x = 0
    loc = locals()
    print('before:', loc)
    exec('x += 1')
    print('after:', loc)
    print('x =', x)
    # 想要获取修改后的值，可以从locals()中获取
    x = loc['x']
    print('x =', x)
    # 注意，再次运行locals()时，原来的值会被覆盖掉
    x = 444
    loc = locals()
    print('new:', loc)

    
>>> test2()
before: {'x': 0}
after: {'x': 1, 'loc': {...}}
x = 0
x = 1
new: {'x': 444, 'loc': {...}}
>>> 
```



