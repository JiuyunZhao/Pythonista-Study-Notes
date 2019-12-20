# 观察者模式

观察者模式中的主题对象一般存在着一个其他服务依赖的核心服务，并且维护着其他依赖此核心服务的对象列表（即观察者或监视者列表），当主题对象发生变化时，观察者应该改变自己的状态或者进行某些操作



#### 观察者模式中的三个角色：

* **主题：**即观察者观察的对象，一般是需要有注册和注销方法，用来添加观察者和删除观察者。
* **观察者基类：**这个类主要是需要定义一个接口，以便主题发生变化时可以得到对应的通知信息。
* **观察者：**这个类需要具体实现基类中的“通知”接口，以便和主题的变化保持同步。



#### 主题的两种通知方式：

* **拉模型：**这个方式重心在观察者上，当主题发生变化时，会广播所有的观察者，然后由观察者来获取相应的数据。
* **推模型：**这个方式重心在主题上，当主题发生变化时，主题将根据观察者的需要将自身的变化推送给需要的观察者。



#### 观察者模式的优点：

* 观察者模式中彼此交互的对象都是保持松耦合的。主题对观察者唯一的了解就是观察者实现的“通知”接口，除此之外它们之间都是互不影响且独立存在的，可以根据需要对自身作出修改。
* 可以随时添加或删除观察者。
* 这种模式下，可以在很少甚至不修改主题或观察者的情况下进行对象之间高效的数据发送。



#### 其他注意点：

* 观察者模式中是可以有多个主题和多个观察者之间的对应关系的，但是一定要弄清楚它们之间的关系以及变化，不然就会变得非常复杂。
* 一般情况是由主题来触发“通知”方法的，但是在特殊情况下也可以由观察者来触发“通知”方法。



**简单示例：**

```py
from abc import ABCMeta, abstractmethod


class Publisher:
    """被观察者：发布/订阅关系中的发布对象"""
    def __init__(self):
        self.subscribers = []
        self.latest_content = None

    def set_content(self, content):
        """有新消息时，发布新的消息"""
        self.latest_content = content
        self.publish()

    def get_latest_content(self):
        """获取最新的消息"""
        return self.latest_content

    def register(self, subscriber):
        """注册一个新的订阅者"""
        self.subscribers.append(subscriber)

    def publish(self):
        """发布消息并通知订阅的用户"""
        for subscriber in self.subscribers:
            subscriber.notify()


class Subscriber(metaclass=ABCMeta):
    """观察者的抽象类：需要定义一个通知接口，用于发布对象通知订阅的用户"""
    @abstractmethod
    def notify(self):
        pass


class SubscriberA(Subscriber):
    """观察者A：发布/订阅关系中的订阅者，当订阅的发布者有新的变化或动态的时候能及时收到通知"""
    def __init__(self):
        self.my_publisher = None

    def subscribe(self, publisher):
        """订阅并进行注册"""
        self.my_publisher = publisher
        self.my_publisher.register(self)

    def notify(self):
        """获取最新消息"""
        latest_content = self.my_publisher.get_latest_content()
        print(self, latest_content)


class SubscriberB(Subscriber):
    """观察者B：发布/订阅关系中的订阅者，当订阅的发布者有新的变化或动态的时候能及时收到通知"""
    def __init__(self):
        self.my_publisher = None

    def subscribe(self, publisher):
        """订阅并进行注册"""
        self.my_publisher = publisher
        self.my_publisher.register(self)

    def notify(self):
        """获取最新消息"""
        latest_content = self.my_publisher.get_latest_content()
        print(self, latest_content)


if __name__ == '__main__':
    new_publisher = Publisher()
    subscriber_a = SubscriberA()
    subscriber_a.subscribe(new_publisher)
    subscriber_b = SubscriberB()
    subscriber_b.subscribe(new_publisher)
    new_publisher.set_content('This is a new message!')
```



