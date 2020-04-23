# 命令模式

命令模式，正如模式的名字一样，该模式中的不同操作都可以当做不同的命令来执行，可以使用队列来执行一系列的命令，也可以单独执行某个命令。该模式重点是将不同的操作封装为不同的命令对象，将操作的调用者与执行者进行解耦。

命令模式中的Command对象（即每一个命令，或者说命令对象）用于封装在完成某项操作或触发一个事件时所需的全部信息，包括需要完成此操作的对象、该对象的方法以及该方法所需的参数，即Command对象中会封装好某项操作所需的所有信息，使用时只需要调用对应的execute方法即可，即表示“这条命令”的执行。通常我们会使用到不只一个命令，因此可能会创建多个Command对象，代表多个不同操作的命令。



#### 命令模式三个角色：

* **Command：**命令对象，对特定的操作进行封装，用于创建不同的命令。
* **Receiver：**参数接受者，即具体操作的执行者。
* **Invoker：**调用命令的对象，由此对象来调用不同的命令对象（即命令队列的创建者）。



#### 命令模式核心思想：

* 将请求封装为对象（即封装为Command命令对象）。
* 可用不同的请求对客户进行参数化（根据不同的操作进行不同命令的参数传值）。
* 允许将请求保存在队列中。
* 提供面向对象的回调。



#### 命令模式优点：

* 将操作的调用者和执行者解耦，使用Command对象来作为中间的代理者。
* 可以使用队列，以便创建和管理一系列的命令。
* 添加新的命令更加容易，且无需更改现有的代码。
* 可以使用命令模式实现重做或回滚操作，以及异步任务执行，只需要执行对应的命令即可。



#### 命令模式缺点：

* 命令模式可能需要创建许多的类和对象来进行相互的协作，所以增加了实现和维护的复杂度。
* 因为每一个命令都是一个Command类，所以如果命令过多，那实现和维护起来就更加的麻烦。



**简单示例：**

```py
from abc import ABCMeta, abstractmethod


class Receiver:
    """Receiver：定义各种方法以便执行不同的操作"""
    def action1(self):
        print('Execute action1...')

    def action2(self):
        print('Execute action2...')


class Command(metaclass=ABCMeta):
    """命令对象接口：定义统一的命令执行方法"""
    @abstractmethod
    def execute(self):
        pass


class Action1(Command):
    """命令1：用于执行操作action1"""
    def __init__(self, receiver):
        self.receiver = receiver

    def execute(self):
        self.receiver.action1()


class Action2(Command):
    """命令2：用于执行操作action2"""
    def __init__(self, receiver):
        self.receiver = receiver

    def execute(self):
        self.receiver.action2()


class Invoker:
    """创建命令队列，调用并执行队列中的命令"""
    def __init__(self):
        self.actions = []

    def append_action(self, action):
        self.actions.append(action)

    def execute_actions(self):
        for action in self.actions:
            action.execute()


if __name__ == '__main__':
    receiver = Receiver()
    action1 = Action1(receiver)
    action2 = Action2(receiver)

    invoker = Invoker()
    invoker.append_action(action1)
    invoker.append_action(action2)
    invoker.execute_actions()
```



