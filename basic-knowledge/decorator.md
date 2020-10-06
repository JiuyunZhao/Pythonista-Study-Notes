# 装饰器

**1、特点：**装饰器的作用就是为已存在的对象添加额外的功能，特点在于不用改变原先的代码即可扩展功能。

**2、使用：**装饰器其实也是一个函数，加上@符号后放在另一个函数“头上”就实现了装饰的功能，执行被装饰的函数时，其实相当于func\(\*args, \*\*kwargs\) = decorator\(func\)\(\*args, \*\*kwargs\)。

**3、应用场景：**常用的有以下一些场景：

* 附加功能。
* 请求拦截：比如登录验证、权限验证等。
* 修改返回值。
* 函数注册。

**4、普通装饰器的基本原理：**在加载装饰器时，也就是加载到@符时，会运行一次装饰器（也就是被@后的函数），它的返回值会替代被装饰的函数地址，而被装饰的函数的地址以装饰器函数参数的形式传进了装饰器。如下示例中：加载到@decorator时，运行了一次decorator\(\)函数，这时函数hello\(\)把函数名（也就是函数地址hello）作为装饰器参数func传了进去，decorator\(\)执行完后返回函数inner\(\)的函数地址，这个函数地址替代了函数hello\(\)的函数地址，当执行hello\('Jom'\)时，就会执行替换后的函数，即inner\('Jom'\)，然后就会去执行inner\(\)函数内的内容。

**普通装饰器示例：**

```py
def decorator(func):
    print('hello python!')  # 相当于给hello添加的额外功能
    def inner(name):
        print('hello, my friend')  # 相当于给hello添加的额外功能
        func(name) # func即为hello()函数

    return inner

@decorator # 装饰器用@表示，函数decorator()装饰函数hello()
def hello(name):
    print('hello %s!' % name)

hello('Jom') # 执行函数

# 输出--------------------------------------------------------------
# hello python!
# hello, my friend
# hello Jom!
```

**5、装饰器传参：**此时定义装饰器时，需要在普通装饰器的外面再定义一个函数，而最外面的这个函数的参数就对应于装饰器使用时传入的参数，具体见示例。

**装饰器传参示例：**

```py
def decorator_var(var1, var2):
    print('decorator_var')

    # 装饰器的传参，就是在简单装饰器外面再套一层“壳子”
    # 并且加载到装饰器的时候，会自动执行decorator_var和decorator两个函数
    def decorator(func):
        print('decorator')

        def inner(name):
            print(var1, var2)
            func(name)

        return inner

    return decorator


@decorator_var('Python', '36')
def hello(name):
    print('hello %s!' % name)


hello('Jom')

# 输出--------------------------------------------------
# decorator_var
# decorator
# Python 36
# hello Jom!
```

**6、安全防护：**被装饰的函数的一些元信息，比如\_\_name\_\_属性，会被修改为装饰器函数返回的函数名（虽然本质上已经不是在执行原函数了，因为它被装饰了，但是“表面上”我们代码还是执行的是这个函数，所以看起来就是原先的函数的属性被修改了，这样“看起来”就是不安全的），但是为了防止这种不安全的行为，可以使用“from functools import wraps”来装饰器装饰器返回的函数，此装饰器会将原函数的一些元信息（如\_\_module\_\_、\_\_name\_\_、\_\_doc\_\_等）拷贝至返回的内函数中。

```py
from functools import wraps


# 装饰器__name__属性防护
def decorator(func):
    @wraps(func)  # 这个装饰器可以防止被装饰的函数的__name__属性值被修改
    def inner(*args, **kwargs):  # 可以使用*args和**kwargs为“万能”的传参方式，当然这里也可以写成inner(a, b)
        func(*args, **kwargs)

    return inner


# 加载到这个装饰器时，相当于：test_func = decorator(test_func)，即test_func = inner
# 如果没有对__name__的保护，加载装饰器后，如：函数test_func的__name__属性就会由“test_func”变为“inner”了
@decorator
def test_func(a, b):
    print('sum of a+b: %s' % (a + b))
```

**7、**多个装饰器的执行顺序：如果某个函数使用了多个装饰器，那么执行的顺序相对于原函数的位置按照从下到上（从近到远），然后再从上到下（从远到近）的顺序执行，具体见示例。

```py
def decorator1(func):
    print('decorator1...')

    def inner(*args, **kwargs):
        print('decorator1 inner...')
        func()
    return inner


def decorator2(func):
    print('decorator2...')

    def inner(*args, **kwargs):
        print('decorator2 inner...')
        func()
    return inner


def decorator3(func):
    print('decorator3...')

    def inner(*args, **kwargs):
        print('decorator3 inner...')
        func()
    return inner


# 多个装饰器，对于装饰器函数中、内函数外的部分按照从下到上的顺序执行，
# 对于内函数中的部分则按照从上到下的顺序执行。
@decorator3
@decorator2
@decorator1
def test_func():
    print('test_func...')


if __name__ == '__main__':
    test_func()

# 输出--------------------------------
# decorator1...
# decorator2...
# decorator3...
# decorator3 inner...
# decorator2 inner...
# decorator1 inner...
# test_func...
```

**8、类装饰器：**如果想要把一个装饰器定义为一个类，其实原理都是一样的，将装饰器当成一个可调用的对象即可，第一次加载到@Decorator时，会执行Decorator\(func\)语句，而执行原函数时，就是相当于执行Decorator\(func\)\(\*args, \*\*kwargs\)，具体见示例。

**普通类装饰器：**

```py
class Decorator:
    def __init__(self, func):
        print('__init__...')
        self._func = func

    def __call__(self, *args, **kwargs):
        print('__call__...')
        self._func(*args, **kwargs)


@Decorator
def test_func():
    print('test_func...')


if __name__ == '__main__':
    test_func()

# 输出------------------------
# __init__...
# __call__...
# test_func...
```

**类装饰器传参示例：**

```py
class Decorator:
    def __init__(self, string):
        self._func = None
        print('__init__...')
        print(string)

    def decorator_func(self):
        print('decorator_func...')
        self._func()

    def __call__(self, *args, **kwargs):
        print('__call__...')
        self._func = args[0]
        return self.decorator_func


@Decorator('hello')
def test_func():
    print('test_func...')


if __name__ == '__main__':
    test_func()

# 输出------------------------
# __init__...
# hello
# __call__...
# decorator_func...
# test_func...
```

**总结：**由上面的示例其实可以看出，在定义装饰器时，无论是定义为函数的形式还是类的形式，使用时只需要简单记为decorator\(func\)\(\*args, \*\*kwargs\)就可以了，如果需要给装饰器传参，不过是在前面多了一个执行步骤而已，就变为了decorator\(\*dec\_args, \*\*dec\_kwargs\)\(func\)\(\*args, \*\*kwargs\)，dec\_args和dec\_kwargs是给装饰器传入的参数。

