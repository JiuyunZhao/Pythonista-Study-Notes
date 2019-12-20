# 工厂模式

“工厂”即表示一个负责创建其他类型的对象的类，通常情况下，一个工厂的对象会有一个或多个方法与之关联，这些方法用于创建不同类型的对象，工厂对象会根据客户端给方法传递的不同的参数或者客户端调用不同的方法返回不同的对象。

#### **优点**

对象的创建是可以根据需要单独创建的，但是使用工厂模式来创建对象有以下优点：

* 松耦合，对象的创建是根据工厂类来进行的，与类本身的实现是独立开来的。
* 对于客户端来说，不需要知道类的具体实现，只需要调用相应接口就可以得到需要的对象了，这其实是简化了客户端的相关实现。
* 对于对象的修改只需要在工厂里面进行即可，包括添加新的对象，客户端只需要更改少量的代码，甚至可以不修改代码就可以达到要求。
* 使用工厂接口，还可以重用已有的对象，不用去别处调用已有的对象或者重新创建一个对象。

#### **工厂模式的3种实现形式（或者说3中变体）：**

* 简单工厂模式：工厂类会提供一个接口，并根据客户端传入参数来创建相应的实例对象。（创建一个对象）
* 工厂方法模式：需要定义一个基类，不同的子类则代表着不同类型的对象。相对于简单工厂模式而言，工厂方法模式具有更强的可定制性。（创建一个对象）
* 抽象工厂模式：需要定义一个抽象工厂类，然后由不同的子类来创建不同系列的对象，一个系列即代表一组对象。（创建一组对象）

#### **简单工厂模式示例：**

```py
from abc import ABCMeta, abstractmethod


class Flower(metaclass=ABCMeta):
    @abstractmethod
    def show_price(self):
        pass


class Rose(Flower):
    def show_price(self):
        print('Rose price: $99')


class Tulip(Flower):
    def show_price(self):
        print('Tulip price: $66')


class FlowerSimpleFactory:
    def get_flower(self, flower_type):
        return eval(flower_type)()


if __name__ == '__main__':
    flower_factory = FlowerSimpleFactory()
    rose = flower_factory.get_flower('Rose')
    tulip = flower_factory.get_flower('Tulip')
    rose.show_price()
    tulip.show_price()
```

```
Rose price: $99
Tulip price: $66
```

**特点：**接口根据客户端传入的参数即可返回对应的实例对象，甚至不用返回它的对象就可以进行对应的操作（比如示例中的工厂FlowerSimpleFactory中可以直接定义一个print\_price方法来打印各种花的价格，而不是先返回对象，再由对象调用show\_price方法来打印），即不会暴露对象的创建逻辑，客户端直接使用接口即可完成对象的创建，甚至创建对象之后的一些操作。



#### 工厂方法模式示例：

```py
from abc import ABCMeta, abstractmethod


class Flower(metaclass=ABCMeta):
    @abstractmethod
    def show_price(self):
        pass


class Rose(Flower):
    def show_price(self):
        print('Rose price: $99')


class Tulip(Flower):
    def show_price(self):
        print('Tulip price: $66')


class Lily(Flower):
    def show_price(self):
        print('Lily price: $33')


class FlowerShopFactory(metaclass=ABCMeta):
    def __init__(self):
        self.flowers = []
        self.stock_flowers()

    @abstractmethod
    def stock_flowers(self):
        pass

    def get_flowers(self):
        return self.flowers

    def add_flower(self, flower):
        self.flowers.append(flower)


class FlowerShop1(FlowerShopFactory):
    def stock_flowers(self):
        self.add_flower(Rose())
        self.add_flower(Tulip())


class FlowerShop2(FlowerShopFactory):
    def stock_flowers(self):
        self.add_flower(Rose())
        self.add_flower(Tulip())
        self.add_flower(Lily())


if __name__ == '__main__':
    flower_shop1 = FlowerShop1()
    for flower in flower_shop1.get_flowers():
        flower.show_price()

    flower_shop2 = FlowerShop2()
    for flower in flower_shop2.get_flowers():
        flower.show_price()
```

```
Rose price: $99
Tulip price: $66
Rose price: $99
Tulip price: $66
Lily price: $33
```

**特点：**工厂方法可以根据基类来定义不同的子类，如示例中的FlowerShop1和FlowerShop2，每个子类则代表“工厂”可以创建的一个“产品”。即对象的创建是通过继承的子类来完成的。



#### 抽象工厂模式示例：

```py
from abc import ABCMeta, abstractmethod


class MiniCar(metaclass=ABCMeta):
    @abstractmethod
    def show_size(self):
        pass


class SedanCar(metaclass=ABCMeta):
    @abstractmethod
    def show_price(self):
        pass


# 国产车
class DomesticMiniCar(MiniCar):
    def show_size(self):
        print('Domestic mini car size: 111')


class DomesticSedanCar(SedanCar):
    def show_price(self):
        print('Domestic sedan car price: 10W')


# 英国车
class EnglishMiniCar(MiniCar):
    def show_size(self):
        print('English mini car size: 222')
        

class EnglishSedanCar(SedanCar):
    def show_price(self):
        print('English sedan car price: 30w')


# 抽象工厂类
class CarFactory(metaclass=ABCMeta):
    @abstractmethod
    def create_mini_car(self):
        pass

    @abstractmethod
    def create_sedan_car(self):
        pass


# 国产车工厂类
class DomesticCarFactory(CarFactory):
    def create_mini_car(self):
        return DomesticMiniCar()
    
    def create_sedan_car(self):
        return DomesticSedanCar()


# 英国车
class EnglishCarFactory(CarFactory):
    def create_mini_car(self):
        return EnglishMiniCar()
    
    def create_sedan_car(self):
        return EnglishSedanCar()
```

**特点：**需要定义一个接口（如示例的抽象工厂类）来创建一系列的相关对象，如示例中的两个子类分别创建两个系列的对象（国产车和英国车），即对象的创建也是由子类来完成。

