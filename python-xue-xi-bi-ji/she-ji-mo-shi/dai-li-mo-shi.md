# 代理模式

代理通常就是一个介于寻求方和提供方之间的中介系统。其核心思想就是客户端（寻求方）没有直接和提供方（真实对象）打交道，而是通过代理对象来完成提供方提供的资源或操作。

代理其实就是封装实际服务对象的包装器或代理人。代理可以为其包装的对象提供附加功能，而无需改变此对象的代码。代理模式的主要目的是为其他对象提供一个代理者或占位符，从而控制对实际对象的访问。



#### 三种常见的不同类型或不同应用场景下的代理：

* **虚拟代理：**如果一个对象实例化后会占用大量的内存，可以先利用占位符表示，只有当客户端请求或访问这个对象时才会创建实际的对象。
* **远程代理：**给位于远程服务器或不同地址空间上的实际对象提供了本地表示。例如应用程序可能需要获取不同服务器或空间地址上的对象信息，这时候就可以通过一个本地的代理来获取相关信息，而不需要直接去和各个服务器或空间地址上的对象“打交道”。
* **保护代理：**通过代理来访问真正的对象，访问时，代理则检查和控制来自客户端的请求权限、认证、授权等，从而保护了真正的实际对象。



#### **代理模式注意点：**

* 客户端实际上可以直接访问真实对象以得到自己想要的结果，但是使用代理也会有许多优势，就如同它的名字“代理”，是可以进行代理的，但是具体的使用还是需要根据具体情况而定。
* 代理是可以根据需要在代理的接口中添加额外的操作的，但需要注意的是这些额外的操作不要变成了“累赘”。
* 由于代理相当于是给真实对象进行了一层封装，所以可能会增加一定的耗时。



**简单示例：**

```py
from abc import ABCMeta, abstractmethod


class HouseOwner(metaclass=ABCMeta):
    """房主抽象类：都可以将房子出租"""
    @abstractmethod
    def rent_house(self, rental):
        pass


class Landlord(HouseOwner):
    """真实对象：房主"""
    def __init__(self):
        self.account = 0
        self.house_key = 'house key'

    def rent_house(self, rental):
        """收取租金，并房屋钥匙给出租的人"""
        self.account += rental
        return self.house_key


class HouseAgent:
    """代理类：中介，代理房东出租他们的房子"""
    def __init__(self):
        self.account = 0
        self.house_resource = []
        # 房源肯定不只一个，这里就只简单放一个了
        self.house_resource.append(Landlord())

    # 通常而言，代理类和表示真实对象的类具有相同的接口
    # 表示此方法给真实对象某个操作进行的代理操作
    def rent_house(self, rental, agency_fee):
        """收取租金和中介费，并将房子出租给客户"""
        self.account += agency_fee
        house_key = self.house_resource[0].rent_house(rental)
        return house_key


class Renter:
    """客户端类：租户"""
    def __init__(self):
        self.account = 10000
        self.house_key = None
        self.house_agent = HouseAgent()

    def find_house(self):
        """在某一个中介（代理对象）处出租房子"""
        self.house_key = self.house_agent.rent_house(3000, 1000)
        print("You've rented a house!")



if __name__ == '__main__':
    renter = Renter()
    renter.find_house()
```



