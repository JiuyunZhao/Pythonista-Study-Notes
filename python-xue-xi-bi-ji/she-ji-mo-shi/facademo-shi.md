# facade模式

facade模式，即门面模式，也称外观模式，这个模式的核心思想是使用facade对象为外部客户端提供一个统一的访问一组子系统的接口，即客户端不会直接与各个子系统交互，而是通过facade对象与各个子系统进行通信并使用子系统的相应功能。

可以通过下面这个图来理解facade模式：

![](/assets/facade.png)

**子系统：**各个子系统原则上都是独立存在的，互不干涉的，重要的是它们都不会去关注facade对象，更不会去引用facade对象。

**facade：**facade对象负责将各个子系统组合成在一起，并为外部提供一个“舒适的外观”和访问接口。

**客户端：**客户端通过facade对象去和各个子系统进行交互，不会直接去和各个子系统打交道。



**简单示例：**

```py
class Shampoo:
    """子系统：卖各种洗发露"""
    def __init__(self):
        print('We sell all kinds of shampoo!')

    def piaorou_500ml(self, number):
        print('This is 500ml piaorou shampoo! Total: %d' % number)


class WashingPowder:
    """子系统：卖各种洗衣粉"""
    def __init__(self):
        print('We sell all kinds of washing powder!')

    def libai_3kg(self, number):
        print('This is 3kg libai washing powder! Total: %d' % number)


class Tissue:
    """子系统：卖各种抽纸"""
    def __init__(self):
        print('We sell all kinds of tissue!')

    def jierou_200sheets(self, number):
        print('This is 200 sheets tissue! Total: %d' % number)


class Salesman:
    """facade：售货员"""
    def __init__(self):
        self.shampoo = Shampoo()
        self.washing_powder = WashingPowder()
        self.tissue = Tissue()
        print('What can I help you?')

    def sale_for_family(self):
        """家庭套餐"""
        self.shampoo.piaorou_500ml(1)
        self.washing_powder.libai_3kg(2)
        self.tissue.jierou_200sheets(6)


class UncleLi:
    """客户端：李大爷"""
    def __init__(self):
        print('I want bug something!')

    def buy_for_family(self):
        """直接从售货员那里购买家庭套餐"""
        sale_man = Salesman()
        sale_man.sale_for_family()
```

其他与facade模式思想相近的编程原则也可以参考下，但需要注意的是“原则”本身需要根据具体情况来灵活应用，而不是一定要这么做：

**最少知识原则：**最少知识意味着需要尽量减少对象之间的交互，但是也需要注意以下几点：

* 在设计系统时，在创建每个对象时，都需要多考查下会与之交互的类的数量以及交互的方式。
* 避免多个对象彼此紧密耦合的情况。

**迪米特法则：**它是一个设计准则，包含以下几点：

* 每个单元对系统中其他单元知道得越少越好。
* 每个单元只与其朋友交流。
* 单元不应该知道它操作的对象的内部细节。



