# 装饰器

* **特点：**装饰器的作用就是为已存在的对象添加额外的功能，特点在于不用改变原先的代码即可扩展功能。
* **使用：**装饰器其实也是一个函数，加上@符号后放在另一个函数“头上”就实现了装饰的功能，执行被装饰的函数时，其实相当于func\(\*args, \*\*kwargs\) = decorator\(func\)\(\*args, \*\*kwargs\)。
* **基本原理：**在加载装饰器时，也就是加载到@符时，会运行一次装饰器（也就是被@后的函数），它的返回值会替代被装饰的函数地址，而被装饰的函数的地址以装饰器函数参数的形式传进了装饰器。如下示例中：加载到@decorator时，运行了一次decorator\(\)函数，这时函数hello\(\)把函数名（也就是函数地址hello）作为装饰器参数func传了进去，decorator\(\)执行完后返回函数inner\(\)的函数地址，这个函数地址替代了函数hello\(\)的函数地址，当执行hello\('Jom'\)时，就会执行替换后的函数，即inner\('Jom'\)，然后就会去执行inner\(\)函数内的内容。

### **装饰器简单示例：**

```text
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

### **装饰器传参示例：**

```text
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

## 装饰器安全防护

被装饰的函数的\_\_name\_\_属性（只测试了这个属性）会被修改为装饰器函数返回的函数名（虽然本质上已经不是在执行原函数了，因为它被装饰了，但是“表面上”我们代码还是执行的是这个函数，所以看起来就是原先的函数的属性被修改了，这样明显就是不安全的），但是为了防止这种不安全的行为，可以使用“from functools import wraps”来装饰器装饰器返回的函数。

### 简单示例：

```text
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

