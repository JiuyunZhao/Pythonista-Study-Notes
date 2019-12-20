# 状态模式

状态模式可以看做是在运行时改变对象行为的一种方式。状态模式允许对象在其内部状态变化时改变其行为，此时感觉就像对象本身已经改变了一样。



#### 参与者：

* **State接口：**State基类，定义不同状态共同需要执行的接口。
* **ConcreteSate对象：**State基类的子类，不同状态的可以在子类接口中实现不同的操作。
* **Context对象：**客户端需要关注的对象，此对象中维护自身的具体状态对象，当状态改变时，改变的是Context中的状态对象，而Context对象是不需要改变的。



#### 优点：

* 对象的行为是根据运行时对象的状态而定，避免了使用大量的条件判断来改变代码逻辑。
* 状态模式中，更加容易添加新的行为，只需要定义一个额外的状态即可。
* 提高了编码的聚合性，使得属于同种状态的操作被归于一个类中。



#### 缺点：

* 由于状态本身的功能过于单一，可能会创建过多的状态类，导致使用和维护都会变得非常麻烦。
* 对于Context对象，每个状态的引入，都可能需要进行更新，并且每次更新都可能会影响到原来的行为，导致Context对象的维护变得更加困难。



**简单示例：**

```py
"""
以电饭煲为例，它有三种状态或者说三种功能：煮饭、煮汤、煮粥
指定好电饭煲的状态后，它就开始以对应模式进行工作
"""
from abc import ABCMeta, abstractmethod


class CookState(metaclass=ABCMeta):
    """State接口：定义状态对象共有的接口，即需要煮什么"""
    @abstractmethod
    def cook(self):
        pass


class CookRice(CookState):
    """ConcreteSate对象：煮饭"""
    def cook(self):
        print('Cooking rice...')


class CookSoup(CookState):
    """ConcreteSate对象：煮汤"""
    def cook(self):
        print('Cooking soup...')


class CookPorridge(CookState):
    """ConcreteSate对象：煮粥"""
    def cook(self):
        print('Cooking porridge...')


class Cooker:
    """Context对象：电饭煲，根据自身状态决定煮什么"""
    def __init__(self):
        # 定义本身具有的几种状态，或者电饭煲的几种功能
        self.states = [CookRice(), CookSoup(), CookPorridge()]
        self.state_index = 0

    def switch_state(self):
        """切换电饭煲的状态"""
        if self.state_index == (len(self.states) - 1):
            self.state_index = 0
        else:
            self.state_index += 1

    def start_cook(self):
        """开始工作"""
        self.states[self.state_index].cook()


if __name__ == '__main__':
    cooker = Cooker()
    cooker.start_cook()
    cooker.switch_state()
    cooker.start_cook()
```



