# 高阶函数

通常我们说的Python高阶函数指的是函数的参数类型为函数，或者函数的返回值类型为函数，Python中常用的高阶函数有map、filter、reduce、partial。

### map

map是一个内置的高阶函数，需要传入一个函数和一个可迭代对象，然后将每个迭代元素作为参数传入到这个函数中，函数的返回值就是这个元素对应的最终结果，具体效果见示例。

```py
>>> # 将列表中的元素全部转换为浮点类型
>>> str_lst = ['1', '2', '3', '4', '5']
>>> float_lst = list(map(float, str_lst))
>>> print(float_lst)
[1.0, 2.0, 3.0, 4.0, 5.0]
```

### filter

filter也是一个内置的高阶函数，同样需要传入一个函数和一个可迭代对象，然后将每个迭代元素作为参数传入到这个函数中，和map不同的是，filter会过滤掉函数返回结果为False的元素，只留下返回结果为True的元素，具体效果见示例。

```py
>>> # 获取列表中的偶数
>>> def func(ele):
...     if ele % 2 == 0:
...         return True
...     return False
... 
>>> num_lst = [1, 2, 3, 4, 5, 6]
>>> even_nums = list(filter(func, num_lst))
>>> even_nums
[2, 4, 6]
```

### reduce

reduce在Python2的版本中是作为内置的高阶函数，但是在Python3中已经移动到内置库functools中了。它需要传入一个函数和一个序列，首先会将序列的第一个元素和第二个元素作为参数传入函数中，然后将这个函数的返回值和第三个元素再次传入此函数，以此类推，最后的函数返回值就是reduce函数的结果了，具体效果见示例。

```py
>>> # 计算1到5的阶乘，即1*2*3*4*5=120
>>> from functools import reduce
>>> def factorial(ele1, ele2):
...     return ele1*ele2
... 
>>> fac_res = reduce(factorial, list(range(1, 6)))
>>> fac_res
120
```

### partial

partial也是在内置库functools中的一个高阶函数，主要用于“包装”另一个函数，给另一个函数的参数定义默认值，返回一个新的函数。考虑一种场景，当一个函数需要多次使用，但是每次使用时某个参数或某几个参数传入的值都是相同的，此时就可以使用partial给此函数的这几个参数的传值指定为默认值。

```py
>>> from functools import partial
>>> def func(a, b, c):
...     return a*b*c
... 
>>> func(2, 3, 4)
24
>>> func(2, 3)  # 如果最后一个参数没有指定默认值，那么使用时就必须传入对应的参数，不然会报错
Traceback (most recent call last):
  File "<input>", line 1, in <module>
TypeError: func() missing 1 required positional argument: 'c'
>>> part_c_func = partial(func, c=4)  # 给参数c指定默认值
>>> part_c_func(2, 3)
24
>>> part_b_func = partial(func, b=3)  # 给参数b指定默认值
>>> part_b_func(2, c=4)  # 注意这里需要显式指定参数c，不然会报错
24
>>> part_b_func(2, c=7)
42
```



