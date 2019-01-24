# json模块和pickle模块（JSON）

Python中的json模块和pickle都是用于数据的序列化和反序列化，它们提供的方法也是一样的：dumps，dump，loads，load。

* **dumps\(obj\)：**将对象序列化为str。
* **dump\(obj, fp\)：**将对象序列化为str，并存入文件中。
* **loads\(s\)：**将（序列化后的）字符串反序列化为Python对象。
* **load\(fp\)：**将文件中的（序列化后的）字符串反序列化为Python对象。

json和pickle模块虽然都是用于数据的序列化和反序列化，但它们之间还是有许多区别的，或者说各有各的优点和缺点：

* **通用性：**json序列化后的字符串是通用的格式（普通的字符串）在不同的平台和语言都可以识别，而pickle序列化后的字符串只有Python可以识别（Python专用序列化模块）  。
* **处理的数据类型：**json能序列化的对象只是Python中基础数据类型，而pickle能序列化Python中所有的数据类型。
* **处理后的数据类型：**json序列化后的字符串是文本类型（记事本打开文件后或者print打印后，你也能看懂其中的内容），而pickle序列化后的字符串是二进制流数据（记事本打开后或者print打印后就完全看不懂里面的内容了）。所以在进行文件操作时注意使用的是哪个模块，是否需要以b的格式打开。
* **使用空间：**json需要的存储空间较小，pickle需要的存储空间较大。



##### 简单示例：

```
>>> import pickle
>>> dic = {'a': 111, 'b': 222, 'c': 333}
>>> f = open('D:/pk_file.pk', 'wb')
>>> lst = [1, 2, 4, 5]
>>> # 将字典对象和列表对象序列化，并存入文件，文件名后缀自定义为.pk
>>> pickle.dump(dic, f)
>>> pickle.dump(lst, f)
>>> f.close()
>>> # 将文件中的Python对象按写入顺序读取出来，且一次读取一个对象
>>> pk_f = open('D:/pk_file.pk', 'rb')
>>> result = pickle.load(pk_f)
>>> type(result)
<class 'dict'>
>>> result
{'a': 111, 'b': 222, 'c': 333}
>>> other_result = pickle.load(pk_f)
>>> type(other_result)
<class 'list'>
>>> other_result
[1, 2, 4, 5]
>>>
```



