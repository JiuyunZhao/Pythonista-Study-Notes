# random模块（随机数）

random模块用于生成各种随机数，常用的方法有：

* **random.random\(\)：**产生0-1之间的随机小数。
* **random.randint\(a, b\)：**产生\[a, b\]之间的随机整数, a和b都是int类型。
* **random.randrange\(start, stop=None, step=1\)：**其实就是产生range\(start, stop=None, step=1\)中的随机整数，即如果stop没有指定，则默认产生\[0, start\)之间的随机整数，若指定了stop，则产生\[start, stop\)之间的随机整数，step用于指定步长。start、stop和step都是int类型。
* **random.uniform\(a, b\)：**返回\[a, b\]之间的一个随机浮点数， a和b可以是int类型，也可以是float类型。
* **random.choice\(seq\)：**从传入的非空序列seq中随机返回一个元素。
* **random.sample\(population, k\)：**随机返回序列或集合中的k个元素的列表。



