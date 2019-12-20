# 单例模式

**单例设计模式：**单例模式提供这样一个机制，确保类有且只有一个特定类型的对象，并提供全局访问点。一般单例模式使用时，需要注意以下几点：

* 确保该类有且只有一个对象被创建。
* 需要为该对象提供一个全局访问点，以便程序可以全局访问该对象（必须保证这个访问点返回的都是同一个对象）。
* 需要注意控制共享资源的访问。

**单例模式优点：**如果在程序中某个对象只需要创建一次，就可以考虑使用单例模式，比如日志操作，数据库操作等，这种对象创建多个就会非常浪费资源，也不能很好的实现这些操作的同步，所以单例模式是一个非常不错的选择。

**单例模式缺点：**它的缺点也很明显，因为单例是全局变量，所以使用时它可能在某个地方已经被修改了，但是开发人员可能并不知道。而且所有使用到这个单例的类，会因为这个单例成为紧密耦合的关系，从而某个类的改变可能会影响到其他的类。

**实现方式：**一般有两种方式，一种是使用元类metaclass控制类实例化时的对象，另一种是使用类的\_\_new\_\_方法控制类返回的对象。（关于元类，这里有篇博客讲得很详细：https://www.cnblogs.com/tkqasn/p/6524879.html）



##### **元类实现单例模式（Python3.6）**

```text
class Singleton(type):
    def __init__(cls, *args, **kwargs):
        cls.__instance = None
        super().__init__(*args, **kwargs)

    def __call__(cls, *args, **kwargs):
        if cls.__instance is None:
            cls.__instance = super().__call__(*args, **kwargs)

        return cls.__instance


class MySingleton(metaclass=Singleton):
    def __init__(self, val):
        self.val = val
        print(self.val)


hello = MySingleton('hello')
hi = MySingleton('hi')
print(hello is hi)

----------输出结果----------
hello
True
```

* **metaclass：**Python3中metaclass是通过指定metaclass实现的，Python2中是通过指定类变量\_\_metaclass\_\_来实现的，但原理都是一样的。
* **type：**Python中所有的类都是type类的实例，即一个类（还未实例化）的定义，其实就是type类（Python内建元类）的实例。如下的打印可以更加直观的理解这一点：

```text
>>> int.__class__
<class 'type'>
>>> num = 3
>>> num.__class__
<class 'int'>
>>> num.__class__.__class__
<class 'type'>
>>> 
>>> 
>>> class A:
    pass

>>> A.__class__
<class 'type'>
>>> a = A()
>>> a.__class__
<class '__main__.A'>
>>> a.__class__.__class__
<class 'type'>
>>>
```

* **\_\_call\_\_：**当调用一个实例时，即执行实例加括号的形式，就会调用该实例的\_\_call\_\_方法，如果没有定义（需要自己定义），则会报错。例如a=A\(\)，a\(\)就会调用a的\_\_call\_\_方法。
* **代码执行流程：**第一步执行MySingleton时（即没加括号的部分），进行元类的实例化，即MySingleton=Singleton\(\)，Singleton的实例化和普通类一样会先执行\_\_new\_\_返回该类的实例，然后自动执行该实例的\_\_init\_\_方法进行初始化，此示例中的初始化方法给该实例赋予了一个值为None的\_\_instance变量；第二步执行MySingleton\('hello'\)时，进行类的实例化，即MySingleton\('hello'\)=Singleton\(\)\('hello'\)，这里就会调用到Singleton的\_\_call\_\_方法了，而super\(\).\_\_call\_\_即调用type的\_\_call\_\_方法，这时候就和普通类实例化一样会调用MySingleton的\_\_new\_\_和\_\_init\_\_方法了。
* **原理：**由于每次实例化MySingleton时都会先调用metaclass中的\_\_call\_\_方法，所以只有第一次实例化时才会执行MySingleton的\_\_new\_\_和\_\_init\_\_，后面的实例化都只会返回第一次实例化好的实例，所以导致的结果就是无论进行多少次实例化，都给你返回同一个实例，当然就只有单例了（所以“输出结果”中就没有打印“hi”了） 。
* **cls和self：**Singleton的编写，在eclipse中提示需要写成self，在PyCharm中提示需要写成cls，因为参数self是约定代表实例本身，但在这里type的实例就是类，所以推荐写成cls。

##### **\_\_new\_\_实现单例模式（Python3.6）**

```text
class MySingleton:
    def __init__(self, val):
        self.val = val 
        print(self.val) 
        print(self.__dict__)

    def __new__(cls, *args, **kwargs):
        if not hasattr(cls, '_instance'):
            cls._instance = super().__new__(cls)

        return cls._instance


hello = MySingleton('hello')
hi = MySingleton('hi')
print(hello is hi)


-----------输出结果--------------
hello
{'val': 'hello'}
hi
{'val': 'hi'}
True
```

* **原理：**通过给类定义一个类变量，指向本类的一个实例，每次实例化调用\_\_new\_\_的时候都返回这个类变量，可以看到数据结果打印的是True，所以自然就是单例了。
* **缺点：**每次实例化虽然都是同一个实例，但是每次实例化都会调用一次\_\_init\_\_方法，导致这个实例会随着每次初始化而改变，所以不推荐这种方式来实现单例，因为\_\_new\_\_方法一般是用来改变类结构的。
* **hasattr：**类中的私有变量，即加了双下划线的变量，在\_\_dict\_\_中会加上一个“\_classname”前缀，所以如果这里使用\_\_instance的话，hasattr\(cls, '\_\_instance'\)会一直返回False，因为这里已经不是\_\_instance了，而是\_MySingleton\_\_instance。



