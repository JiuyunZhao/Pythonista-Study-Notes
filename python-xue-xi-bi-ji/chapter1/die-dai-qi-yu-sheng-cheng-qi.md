# 迭代器与生成器

#### 迭代器

* **特点：** 迭代器用于访问集合（此集合非彼集合set\(\)）中的元素，使用迭代器不需要提前准备好要迭代的所有元素，只有迭代到某个元素时才会计算该元素，这个元素之前或之后都是没有的，因为迭代器这个特点，它遍历集合元素占用的内存很少，适用于访问一些特别大的甚至是无限的集合。
* **访问迭代器：**访问迭代器元素时会使用自身的\_\_next\_\_\(\)方法（Python2为next\(\)）不断地去访问下一个元素，且迭代器只能“前进”，不能“后退”，也不能通过下标等去访问某个特定的元素。当访问完迭代器后，这个迭代器就“完了”，要想再遍历这个集合，就要为这个集合再新建一个迭代器了。
* **\_\_iter\_\_方法和\_\_next\_\_方法：**当将一个类作为迭代器时，即自定义迭代器时，需要实现\_\_iter\_\_和\_\_next\_\_这两个方法（Python2为next\(\)），\_\_iter\_\_用于返回一个定义了\_\_next\_\_方法的对象，\_\_next\_\_方法用于返回每次迭代时的对象，并且需要定义StopIteration异常来标识迭代的完成。
* **iter\(\)和next\(\)：**这是用于迭代器的两个内建函数，iter\(\)用于为一个可迭代对象创建一个迭代器，next\(\)则用于访问迭代器的元素。
* **注：**Python2中迭代器自身的迭代方法是next\(\)，Python3中为\_\_next\_\_\(\)。

##### 简单示例：

```
>>> # 创建迭代器
>>> class MyIter:
    def __init__(self):
        self.a = 1

    def __iter__(self):
        return self

    def next(self):
        a_ = self.a
        if a_ < 5:
            self.a += 1
            return a_
        else:
            raise StopIteration


>>> it = MyIter()
>>> next(it)
>>> for item in it:
    print(item)
3
>>> next(it)

Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    next(it)
StopIteration
>>> 
>>> # iter()和next()方法的使用
>>> lst = ['a', 'b', 'c']
>>> new_it = iter(lst) 
>>> next(new_it)
'a'
>>> new_it.next()
'b'
>>> for item in new_it:
    print(item)


c
>>>
```

#### 

#### **生成器：**

* **定义：**如果一个函数被调用时返回了一个迭代器，那么这个函数就叫做生成器；如果一个函数中含有yield语法，那么这个函数也是生成器。
* **yield语法的特点：**当含有yield语法的函数被调用后，执行yield后，就会中断这个函数，继续执行这个函数之后的代码，当下一次这个函数再被调用时，会从这个函数的上次执行的yield之后的代码开始执行。

##### 简单示例：

```
>>> def generator():
    n = 3
    while n > 0:
        yield 'hello python!' # 含有yield，这个函数即为生成器，每次生成一个字符串“hello python!”
        n -= 1


>>> iterator = generator() # 返回一个迭代器
>>> iterator.__next__()
'hello python!'
>>> iterator.__next__()
'hello python!'
>>> iterator.__next__()
'hello python!'
>>> iterator.__next__() # 访问完后就“没有了”，这个迭代器也不能再用了
Traceback (most recent call last):
  File "<pyshell#13>", line 1, in <module>
    iterator.__next__()
StopIteration
>>>
```



