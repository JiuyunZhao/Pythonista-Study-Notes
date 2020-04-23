### 1.2 解压可迭代对象赋值给多个变量

此小节主要是对于星号表达式的探讨，当想要将一个可迭代对象的部分元素赋值给某个变量，尤其是这部分元素的数量并不确定时，星号表达式就能很好的排上用场了。

**使用场景1：**普通的分割赋，需要注意的是，星号表达式的变量为列表类型，即使该变量中的元素为0个。

```py
>>> record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212')
>>> name, email, *phone_numbers = record
>>> name
'Dave'
>>> email
'dave@example.com'
>>> phone_numbers
['773-555-1212', '847-555-1212']
>>> name, email, phone_number1, phone_number2, *others = record
>>> others
[]
>>>
```

**使用场景2：**在迭代元素为可变长序列时，星号表达式也很好用。

```py
records = [
    ('foo', 1, 2),
    ('bar', 'hello'),
    ('foo', 3, 4),
]


def do_foo(x, y):
    print('foo', x, y)


def do_bar(s):
    print('bar', s)


for tag, *args in records:
    if tag == 'foo':
        do_foo(*args)
    elif tag == 'bar':
        do_bar(*args)
```



### 1.3 保留最后 N 个元素

在迭代操作或者其他操作中，如果想要保留一个有固定元素个数的对象的时候，可以考虑使用collections.deque，它会构造一个固定长度的队列（如果指定了的话），当队列满了之后，如果继续往队列中添加元素，就会删除最老的元素，再添加新的元素进去。deque也有对应的方法在队列的两端添加和弹出元素。虽然可以使用列表等方法实现相同的功能，但deque的性能上是优于列表的插入删除等操作的，当然，具体的场景还是要看个人的使用习惯和选择了，此小节只是给出了另一个值得考虑的解决方案。

```py
>>> from collections import deque
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3], maxlen=3)
>>> q.append(4)
>>> q
deque([2, 3, 4], maxlen=3)
>>> q.appendleft(5)
>>> q
deque([5, 2, 3], maxlen=3)
>>> q.pop()
>>> q
deque([5, 2], maxlen=3)
>>> q.popleft()
>>> q
deque([2], maxlen=3)
>>>
```



### 1.4 查找最大或最小的 N 个元素

在一个元素个数比较多的集合中查找最大或最小的N个元素，并且要查找的元素个数相对于集合本身元素个数较小时，可以考虑使用heapq模块中的nlargest和nsmallest函数。需要注意的是如果要查找的元素个数和集合本身的元素个数相近的话，建议使用先排序后切割的方式，如sorted\(items\)\[:N\]。

在heapq中，由于它是堆结构的，所以第一个元素永远是最小的，当你想要多次使用集合，并且每次使用集合时可以得到集合的最小元素时，可以考虑使用heapq模块，当然，如果你仅仅是想要得到该集合中的唯一一个最大或最小的元素时，建议直接使用max或min函数。

```py
>>> import heapq
>>> nums = [1, 8, 2, 23, 7, -4, 18, 23, 42, 37, 2]
>>> heapq.nlargest(3, nums)
[42, 37, 23]
>>> heapq.nsmallest(3, nums)
[-4, 1, 2]
>>> portfolio = [
    {'name': 'IBM', 'shares': 100, 'price': 91.1},
    {'name': 'AAPL', 'shares': 50, 'price': 543.22},
    {'name': 'FB', 'shares': 200, 'price': 21.09},
    {'name': 'HPQ', 'shares': 35, 'price': 31.75},
    {'name': 'YHOO', 'shares': 45, 'price': 16.35},
    {'name': 'ACME', 'shares': 75, 'price': 115.65}
]
>>> heapq.nlargest(3, portfolio, key=lambda s: s['price'])
[{'name': 'AAPL', 'shares': 50, 'price': 543.22}, {'name': 'ACME', 'shares': 75, 'price': 115.65}, {'name': 'IBM', 'shares': 100, 'price': 91.1}]
>>> 
>>> heap = list(nums)
>>> heapq.heapify(heap)  # 将数据进行堆排序后放入列表中
>>> heap
[-4, 2, 1, 23, 7, 2, 18, 23, 42, 37, 8]
>>> heapq.heappop(heap)
-4
>>> heapq.heappop(heap)
>>> heapq.heappop(heap)
>>>
```



### 1.8 字典的运算

如果想要某个字典中键最大（最小）或值最大（最小）的键值对，可以使用zip\(\)函数将键值先反转过来。

```py
>>> prices = {
    'ACME': 45.23,
    'AAPL': 612.78,
    'IBM': 205.55,
    'HPQ': 37.20,
    'FB': 10.75
}
>>> max(zip(prices.values(), prices.keys()))
(612.78, 'AAPL')
>>> min(zip(prices.values(), prices.keys()))
(10.75, 'FB')
>>>
```



### 1.11 命名切片

程序中使用下标进行切片的情况还是比较多的，如果下标在使用时写死的话，就会使得程序后期变得难以维护，这时可以考虑将切片的下标换为内置的slice函数，所有的下标切片都可以使用这个函数。

```py
>>> items = [0, 1, 2, 3, 4, 5, 6]
>>> a = slice(2, 4)
>>> items[2:4]
[2, 3]
>>> items[a]
[2, 3]
>>> items[a] = [10, 11]
>>> items
[0, 1, 10, 11, 4, 5, 6]
>>> a.start
>>> a.stop
>>> a.step
>>> 
>>> record = '....................100 .......513.25 ..........'
>>> cost = int(record[20:23]) * float(record[31:37])  # 下标切片写死在代码里，不建议这样做
>>> cost
51325.0
>>> SHARES = slice(20, 23)
>>> PRICE = slice(31, 37)
>>> cost = int(record[SHARES]) * float(record[PRICE])
>>> cost
51325.0
>>>
```



### 1.12 序列中出现次数最多的元素

对于一个序列中每个元素的出现次数，当然可以自己手动利用字典去实现，但是最优的选择应该是collections.Counter，它就是专门为这类问题而设计的，Counter对象的底层实现其实也是一个字典，具体使用见示例代码。

```py
>>> from collections import Counter
>>> words = [
    'look', 'into', 'my', 'eyes', 'look', 'into', 'my', 'eyes',
    'the', 'eyes', 'the', 'eyes', 'the', 'eyes', 'not', 'around', 'the',
    'eyes', "don't", 'look', 'around', 'the', 'eyes', 'look', 'into',
    'my', 'eyes', "you're", 'under'
]
>>> word_counts = Counter(words)
>>> word_counts.most_common(3)  # 出现次数最多的3个单词
[('eyes', 8), ('the', 5), ('look', 4)]
>>> word_counts['eyes']
>>> morewords = ['why','are','you','not','looking','in','my','eyes']
>>> for word in morewords:word_counts[word] += 1  # 像字典一样操作Counter对象

>>> word_counts['eyes']
>>> word_counts.update(morewords)  # 或者使用update方法进行更新
>>> a = Counter(words)
>>> b = Counter(morewords)
>>> c = a + b  # 可以使用+/-运算符进行操作
>>> c
Counter({'eyes': 9, 'the': 5, 'look': 4, 'my': 4, 'into': 3, 'not': 2, 'around': 2, "don't": 1, "you're": 1, 'under': 1, 'why': 1, 'are': 1, 'you': 1, 'looking': 1, 'in': 1})
>>>
```



### 1.13 通过某个关键字排序一个字典列表

如果想要对一个元素为字典的列表进行排序操作，如sorted/max/min等函数，平常使用对应的lambda表达式即可，但是如果对性能有要求的话，建议使用operator.itemgetter函数。

如果排序的元素不是字典，而是类对象，则可以使用operator.attrgetter，这是1.14节的内容，用法与itemgetter相似，就不贴示例代码了

```py
>>> from operator import itemgetter
>>> rows = [
    {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003},
    {'fname': 'David', 'lname': 'Beazley', 'uid': 1002},
    {'fname': 'John', 'lname': 'Cleese', 'uid': 1001},
    {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}
]
>>> sorted(rows, key=itemgetter('fname'))
[{'fname': 'Big', 'lname': 'Jones', 'uid': 1004}, {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003}, {'fname': 'David', 'lname': 'Beazley', 'uid': 1002}, {'fname': 'John', 'lname': 'Cleese', 'uid': 1001}]
>>> sorted(rows, key=itemgetter('lname','fname'))
[{'fname': 'David', 'lname': 'Beazley', 'uid': 1002}, {'fname': 'John', 'lname': 'Cleese', 'uid': 1001}, {'fname': 'Big', 'lname': 'Jones', 'uid': 1004}, {'fname': 'Brian', 'lname': 'Jones', 'uid': 1003}]
>>>
```



### 1.20 合并多个字典或映射

当你想在多个字典中查找某些键或值的时候可以使用collections.ChainMap，它会将多个字典从逻辑上合并为一个字典，如果多个字典中有相同的键，则对于该键的操作总是对应的拥有该键的第一个字典。需要注意的是对于ChainMap对象的操作是会作用到对应的字典上去的。

```py
>>> from collections import ChainMap
>>> a = {'x': 1, 'z': 3 }
>>> b = {'y': 2, 'z': 4 }
>>> c = ChainMap(a,b)
>>> c['x']
>>> c['y']
>>> c['z']
>>> len(c)
>>> list(c.keys())
['x', 'z', 'y']
>>> list(c.values())
[1, 3, 2]
>>> c['z']
>>> c['z'] = 10
>>> c['z']
>>> c['w'] = 40
>>> c['w']
>>> a
{'x': 1, 'z': 10, 'w': 40}
>>>
```



