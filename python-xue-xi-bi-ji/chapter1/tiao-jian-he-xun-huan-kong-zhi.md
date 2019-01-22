# **条件和循环控制**

#### if条件控制

基本结构为：if-elif-else，if和elif后面跟一个布尔表达式，若值为真，则执行对应子语句块，若值为假，则跳过此条件块。

#### while循环控制

while后面跟一个布尔表达式，若值为真，则执行其中的语句块，执行完后再一次判断布尔表达式的值和执行语句块，然后无限重复判断和执行，直到布尔表达式的值为假。可以使用continue结束本层循环的本次循环，而使用break则结束本层的整个循环。

#### for循环

for用于遍历一个可迭代对象，使用continue可以结束本层循环的本次循环，而使用break则结束本层的整个循环。

#### for和while的else

当for或while正常执行完后接着执行else中的内容，当forwhile循环中使用break终止跳出循环后就不会执行else中的内容了。

#### 简单语句组

if、while和for语句中的语句块如果只有一条简单的语句，这时候条件判断和字句块可以写在一行，比如：if a &gt; 1: print\(a\)。



##### 简单示例：

```
>>> for i in range(5):
    print(i)
    for j in range(20, 25):
        if j == 21:
            continue
        if j == 23:
            break
        print(i, j)
0 20 #21项没有是因为continue结束了"j == 21"这次循环
22 #23项和24项是因为break结束了本层for循环
1 20
22
     
20
22
3 20
22
     #break并没有作用于外层的for循环，所以会循环到底
20
22
>>> 
 

>>> for i in range(5):  # 正常执行完毕
    print(i)
else:
    print('end')
1
3
end
>>> for i in range(5):  # break中断循环后就不会执行else中的内容了
    if i % 2 == 1:
        break
    print(i)
else:
    print('end')
>>>  # 简单语句组
>>> if 3 > 1: print('ok')
ok
>>> a = 3
>>> while a > 0: a -= 1
>>> for i in range(2): print(i)
1
>>>
```



