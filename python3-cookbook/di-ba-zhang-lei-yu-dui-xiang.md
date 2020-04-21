### 8.4 创建大量对象时节省内存方法

创建大量的对象非常的占内存，可能有上百万的数量，这时候可以考虑使用类的\_\_slots\_\_属性，它会将类的实例当成一个固定大小的数组来表示，而不是字典。

但需要注意的是，我们应该尽量少的使用这个属性，因为这种类在定义之后就不再支持添加新属性了，甚至一些类的普通特性也不支持了，比如继承等特性。

```py
import sys


class Date:
    __slots__ = ['year', 'month', 'day']

    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day


d = Date(1, 2, 3)

try:
    # 不能直接通过打印sys.getsizeof(d)查看实例d的内存大小，可以通过打印它的__dict__属性来查看
    # 如果定义了__slots__属性后，实例就没有__dict__属性了，也就是不能动态添加新属性了
    print(sys.getsizeof(d.__dict__))
except Exception as e:
    print(e)
```



### 8.6 创建可管理的属性

当给某个对象的属性赋值时，你想给它进行一些额外的逻辑处理，比如类型检查或者合法性验证的时候，可以使用property装饰器来定义它。定义property属性时，需要注意它们的方法名（也就是属性名）需要都一样。

如果想要定义一个动态的属性时，相比于使用get/set方法的形式，使用property来定义会让Python代码更加优雅。

**property基础用法示例：**

```py
class Person:
    def __init__(self, first_name):
        # 这里的self.first_name不是重复定义，而是在property中就定义了
        # 这里是在对self.first_name属性赋值，会触发该属性的setter方法
        self.first_name = first_name

    @property
    def first_name(self):
        """getter方法：属性名称为方法名称，并且只有定义了getter方法，才能定义其他的setter等方法"""
        return self._first_name

    # 装饰器的前缀名称必须跟property装饰器修饰的方法名称一样，即属性名称
    @first_name.setter
    def first_name(self, value):
        """方法名称必须跟property装饰器修饰的方法名称一样，即属性名称"""
        if not isinstance(value, str):
            raise TypeError('Expected a string!!!')
        self._first_name = value

    @first_name.deleter
    def first_name(self):
        raise AttributeError("Can't delete attribute!!!")


a = Person('Guido')
print(a.first_name)

try:
    a.first_name = 42
except TypeError as e:
    print(e)

try:
    del a.first_name
except AttributeError as e:
    print(e)
```

```
Guido
Expected a string!!!
Can't delete attribute!!!
```

**定义动态属性：**

```py
import math


class Circle:
    def __init__(self, radius):
        self.radius = radius

    @property
    def area(self):
        return math.pi * self.radius ** 2

    @property
    def diameter(self):
        return self.radius * 2

    @property
    def perimeter(self):
        return 2 * math.pi * self.radius
```



### 8.11 简化数据结构的初始化

当需要写大量的用作数据结构的类，又不想每个类都定义一个\_\_init\_\_方法时，可以参考以下例子。

```py
import math


class Structure1:
    # 属性名称列表，在子类中覆盖即可
    _fields = []

    def __init__(self, *args):
        if len(args) != len(self._fields):
            raise TypeError('Expected {} arguments'.format(len(self._fields)))

        # 如果需要添加关键字参数，或者添加不在_fields中的关键字参数，也是用这个原理定义即可
        for name, value in zip(self._fields, args):
            setattr(self, name, value)


# 实例化示例
class Stock(Structure1):
    _fields = ['name', 'shares', 'price']


class Point(Structure1):
    _fields = ['x', 'y']


class Circle(Structure1):
    _fields = ['radius']

    def area(self):
        return math.pi * self.radius ** 2
```



### 8.16 在类中定义多个构造器

类方法classmethod的一个主要用途就是定义多个构造器。

```py
import time


class Date:
    def __init__(self, year, month, day):
        self.year = year
        self.month = month
        self.day = day

    @classmethod
    def today(cls):
        """此方法就相当于第二个构造器了"""
        t = time.localtime()
        return cls(t.tm_year, t.tm_mon, t.tm_mday)


# 使用默认的__init__方法初始化
a = Date(2012, 12, 21)
# 直接返回一个新的实例
b = Date.today()
```



### 8.25 创建缓存实例

类似单例模式，某个对象你只想创建一次，或者想要使用之前创建好的对象，就像logging模块，获取相同名称的logger永远只有一个，这个问题除了使用元类等方式，还可以借助weakref模块。但是需要保证weakref.WeakValueDictionary\(\)缓存中的对象在其他地方还在被使用，没有被删除回收掉。

```py
import weakref


class Spam:
    def __init__(self, name):
        self.name = name


# 这个变量也可以放在类定义中，用于保证每次类的实例化都返回的是同一个实例对象
# 但是需要保证缓存中的对象在其他地方还在被使用，没有被删除回收掉
_spam_cache = weakref.WeakValueDictionary()


# 当程序中其他地方想要使用某个实例时，传入对应名称即可
def get_spam(name):
    if name not in _spam_cache:
        s = Spam(name)
        _spam_cache[name] = s
    else:
        s = _spam_cache[name]
    return s


a = get_spam('foo')
b = get_spam('bar')
c = get_spam('foo')
print(a is b)  # False
print(a is c)  # True
```



