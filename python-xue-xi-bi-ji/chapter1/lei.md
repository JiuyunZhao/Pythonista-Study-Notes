# 类

* **类定义：**Python3中，如果新建的类没有继承任何其他类，默认继承基础类object。Python2中如果没有显式继承object类就是经典类，而显式继承了object类就是新式类，Python2推荐使用新式类。
* **类变量：**类变量就是直接在类中，但是在方法外定义的变量。类变量是所有该类的实例所共有的，且类的每个实例都可以修改类变量。
* **成员变量：**成员变量就是在定义时加了self前缀的变量，一般在\_\_init\_\_方法就定义了，成员变量会跟着实例“一起走”。
* **类的继承：**直接在类后括号里加上要继承的类就行，继承多个类时，类之间用逗号分隔即可。
* **成员方法：**新建方法时默认为新建一个成员方法，方法的第一个参数必须为self，self代表着实例本身，相当于每次都会将实例本身当参数传进去。
* **类方法：**方法“头上”加上装饰器@classmethod，方法的第一个参数必须为cls，代表着类本身。
* **静态方法：**方法“头上”加上装饰器@staticmethod，即普通函数，参数没有强制要求。

**注：**self和cls都是约定的，不是语法规定必须写成这样，但是强烈建议不要自己定义。

#### 特殊方法：

* **\_\_init\_\_：**类的初始化方法，在一个类实例化之后会自动调用这个方法，实例化时传进去的参数也是按这个初始化方法来定的。
* **\_\_new\_\_：**类的构造方法，该方法返回本类的一个实例。执行完此方法后，会自动调用类的\_\_init\_\_方法来初始化该实例，需要注意的是，如果自己重写了这个方法，一定要返回本类的实例，否则执行\_\_new\_\_后不会调用\_\_init\_\_。
* **\_\_call\_\_：**当一个实例对象以函数的形式调用时，就会执行此方法，前提是类中已经定义了此方法，没有定义的话，就会报错。例如：a = A\(\)，a\(\)就会调用类A中\_\_call\_\_方法，当然，如果没有定义\_\_call\_\_，就会报错。
* **\_\_str\_\_：**面向用户的方法，使用print打印类实例时，如果类重写了方法\_\_str\_\_，会优先打印输出\_\_str\_\_的返回值，如果没有重写这个方法就会打印\_\_repr\_\_方法的返回值。
* **\_\_repr\_\_：**面向Python本身的方法，默认返回值为类实例的内存地址，直接输出者类实例时，如在Python的IDLE中直接“&gt;&gt;&gt; a”（a为类A的实例），就会打印\_\_repr\_\_的返回值。



##### 简单示例：

```
# 推荐使用新式类，即继承object
class FirstClass(object):
    var1 = 30  # 定义类变量 
    var2 = 50

    # 初始化方法
    def __init__(self, para1, para2):
        # 定义成员变量
        self.para1 = para1
        self.para2 = para2

    # 成员方法，默认带参数self
    def func1(self):
        print('hello, guys!')

    # 也可以不在初始化方法定义成员变量，不推荐！！！
    def func2(self, para3):
        self.para3 = para3

    # 静态方法，不用带self参数
    @staticmethod
    def func3():
        return 'static method'

    # 类方法，需要带cls参数，表示类本身
    @classmethod
    def func4(cls):
        cls.var1 = 'var1'

examp1 = FirstClass('hi', 'hello')
examp2 = FirstClass('python', 'java')
examp1.func1()
examp2.func1()
examp1.func2('hei!')
print(examp1.var1)
print(examp2.var2)
print(examp2.para1)
print(examp1.para2)
print(examp1.para3)

输出：
hello, guys!
hello, guys!
50
python
hello
hei!
```



#### get方法和set方法

定义类属性的get方法和set方法使用装饰器@property，在一个方法上使用@property，该方法名就会成为外部访问对应属性的属性名，并且会自动生成一个xxx.setter的装饰器，用于定义对应的set方法，注意get方法和set方法的方法名都是一样的。@property一般用于保护属性或者私有属性，因为公共属性在哪都可以访问和修改，没有必要再去定义@property方法。



##### 简单示例：

```
class MyClass:
    def __init__(self):
        # 前面加两个下划线表示私有属性，外部不可访问
        # 这里为了方便测试，看到运行后的效果，定义为私有属性
        # 需要用到@property的属性一般定义为保护或者私有，
        # 不然公共的属性，谁都可以访问和修改，就没必要使用@property
        self.__var = 0

    # 加了@property后，函数名my_var自动成为外部访问的属性名，即get方法，而外部就不能访问直接访问__var属性了
    # 并且自动创建另一个装饰器@my_var.setter，用于设置该属性值，即set方法
    @property
    def my_var(self):
        return self.__var

    # set方法这里要注意函数名需要与get方法一致
    @my_var.setter
    def my_var(self, value):
        self.__var = value


# 实例化，定义一个私有属性
myclass = MyClass()
# 访问私有属性，自动调用get方法，即@property修饰的函数my_var
print(myclass.my_var)
# 设置私有属性，自动调用set方法，即my_var.setter修饰的函数my_var
myclass.my_var = 33
print(myclass.my_var)

# 以下可自行测试
# 私有属性不可访问，会报错
# print(myclass.__var)
# 私有属性设置无效
# myclass.__var = 44
# print(myclass.my_var)

输出：
33
```



