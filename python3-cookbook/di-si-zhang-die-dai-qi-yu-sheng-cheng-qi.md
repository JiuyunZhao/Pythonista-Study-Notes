### 4.2 代理迭代

如果想要迭代一个不可迭代对象，只需要为这个对象定义一个\_\_iter\_\_\(\)方法即可，\_\_iter\_\_\(\)方法必须返回一个实现了\_\_next\_\_\(\)方法的迭代器对象。

```py
class Node:
    """Node类似一个树节点"""
    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        # iter(s)只是简单的通过调用s.__iter__()方法来返回对应的迭代器对象，就跟len(s)会调用s.__len__()原理是一样的
        return iter(self._children)


if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    # 输出当前节点下其他节点的打印值
    for ch in root:
        print(ch)
```

```
Node(1)
Node(2)
```



### 4.4 实现迭代器协议

想在迭代某个对象时按照自己的方式来迭代，最简单的方法就是使用yield定义一个生成器函数，但是需要注意的是，在迭代操作时，如果不是使用for循环，就需要先使用iter\(\)函数转换一下，再去迭代它。比如以下示例代码中在树形结构中定义一个深度优先的生成器函数：

```py
class Node:
    """Node类似一个树节点"""

    def __init__(self, value):
        self._value = value
        self._children = []

    def __repr__(self):
        return 'Node({!r})'.format(self._value)

    def add_child(self, node):
        self._children.append(node)

    def __iter__(self):
        # 返回一个可以迭代子节点的迭代器
        return iter(self._children)

    def depth_first(self):
        """深度优先遍历节点"""
        # 使用yield定义一个生成器
        yield self
        for c in self:
            # 注意这里是yield from
            yield from c.depth_first()


if __name__ == '__main__':
    root = Node(0)
    child1 = Node(1)
    child2 = Node(2)
    root.add_child(child1)
    root.add_child(child2)
    child1.add_child(Node(3))
    child1.add_child(Node(4))
    child2.add_child(Node(5))
    # 以深度优先原则遍历节点
    for ch in root.depth_first():
        print(ch)
```

```
Node(0)
Node(1)
Node(3)
Node(4)
Node(2)
Node(5)
```



### 4.7 迭代器切片

想要对迭代对象切片，或者说只想要其中某一段，可以使用itertools.islice，但是需要注意的是这样会消耗掉这个迭代器，之后就不能使用了，因为迭代器是不可逆的。

```py
>>> def count(n):
    while True:
        yield n
        n += 1

        
>>> c = count(0)
>>> c[10:20]
Traceback (most recent call last):
  File "<pyshell#105>", line 1, in <module>
    c[10:20]
TypeError: 'generator' object is not subscriptable
>>> import itertools
>>> for x in itertools.islice(c, 10, 20):
    print(x)
11
13
15
17
19
>>>
```



### 4.8 跳过可迭代对象的开始部分

在遍历一个可迭代对象时，想要跳过开始的某些元素，可以使用itertools.dropwhile，为它传入一个函数和可迭代对象，如果知道确切的索引位置，也可以使用itertools.islice。

```py
>>> from itertools import dropwhile, islice
>>> items = ['a', 'b', 'c', 1, 4, 10, 15]
>>> for x in dropwhile(lambda i: isinstance(i, str), items):
    print(x)
4
15
>>> for x in islice(items, 3, None):
    print(x)
4
15
>>>
```



### 4.11 同时迭代多个序列

内置函数zip的使用有时候很方便，但是它只会遍历到最短的那个序列完就结束了，如果想要遍历完最长的那个序列，可以使用itertools.zip\_longest\(\)。

```py
>>> a = [1, 2, 3]
>>> b = ['w', 'x', 'y', 'z']
>>> for i in zip(a,b):
    print(i)

    
(1, 'w')
(2, 'x')
(3, 'y')
>>> from itertools import zip_longest
>>> for i in zip_longest(a, b):
    print(i)

    
(1, 'w')
(2, 'x')
(3, 'y')
(None, 'z')
>>>
```



### 4.12 不同集合上元素的迭代

想要遍历多个可迭代对象中的元素，但又不想单独遍历每个对象，或者把它们都整合在一个对象中再遍历，此时可以使用itertools.chain\(\)。

```py
>>> from itertools import chain
>>> a = [1, 2, 3, 4]
>>> b = ['x', 'y', 'z']
>>> for x in chain(a, b):
    print(x)
2
4
x
y
z
>>>
```



### 4.14 展开嵌套的序列

展开嵌套的序列，这个问题或许有其他的解决方式，但文中使用递归生成器的方式还是很很不错的。

```py
from collections import Iterable


def flatten(items, ignore_types=(str, bytes)):
    for x in items:
        if isinstance(x, Iterable) and not isinstance(x, ignore_types):
            yield from flatten(x)
        else:
            yield x


items = [1, 2, [3, 4, [5, 6], 7], 8]
for x in flatten(items):
    print(x)
```

```
1
2
3
4
5
6
7
8
```



### 4.15 顺序迭代合并后的排序迭代对象

你有多个可迭代对象，想要将它们合并排序后遍历里面的元素，那么可以使用heapq.merge\(\*iterables, key=None, reverse=False\)，但是需要注意，使用这个函数前每个可迭代对象都要预先排序好，因为这个函数只是每次从多个序列的第一个元素中选出最小或最大的元素。并且因为它是可迭代的，意味着它可以处理非常长的序列而不用担心内存消耗。

```py
>>> import heapq
>>> a = [1, 4, 7, 10]  # 预先排好序的序列
>>> b = [2, 5, 6, 11]
>>> for c in heapq.merge(a, b):
    print(c)
2
5
7
11
>>>
```



### 4.16 迭代器代替while无限循环

某些情况下可以使用iter创建一个迭代器来替换while循环，iter函数它接受一个可选的 callable 对象和一个标记\(结尾\)值作为输入参数。当以这种方式使用iter的时候，它会创建一个迭代器， 这个迭代器会不断调用 callable 对象直到返回值和标记值相等为止。虽然文中并没有说这两种方式在性能上有什么差别，但是从代码编写上看，iter的方式会更加优雅些。

```py
CHUNKSIZE = 8192

def reader(s):
    while True:
        # 接收数据
        data = s.recv(CHUNKSIZE)
        if data == b'':
            break
        # 处理数据
        process_data(data)
```

```py
def reader2(s):
    for data in iter(lambda: s.recv(CHUNKSIZE), b''):
        # 处理数据
        process_data(data)
```



