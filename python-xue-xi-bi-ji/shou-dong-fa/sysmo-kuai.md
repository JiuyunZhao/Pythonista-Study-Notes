# sys模块

**sys.argv：**参数字符串列表（动态对象），第一个参数为当前程序主文件的绝对路径或空字符串，如果在命令提示符界面给Python文件传了参数（不同的参数以空格分隔，无论传入的时候写的是什么类型，最终都会转成字符串），可以在这里面获取（从第二个位置开始），比如命令提示符中运行“python main.py 111 aaa”，那sys.argv就有三个元素，第二个和第三个元素分别为“111”和“aaa”。

**sys.path：**搜索模块路径字符串列表（动态对象），搜索查找模块时会优先到这里面去搜索，第一个参数为主文件所在目录的路径或空字符串。

**sys.modules：**已经加载的模块信息字典，key为模块名称，value为模块对象，在使用\_\_import\_\_导入模块时，可以先判断下是否有同名模块已经在sys.modules中加载了，如果已经存在了，可以先删除或者不再导入了。

**sys.getsizeof\(object\)：**获取一个对象的内存占用字节数大小。

**sys.getdefaultencoding\(\)：**返回Python默认的字符串编码格式。

**sys.exit\(\[status\]\)：**退出Python解释器，并抛出一个SystemExit异常，status默认为0，即“成功”，如果status是一个整数，则被用作一个系统退出状态，如果status是其他对象，则它将会被print并系统退出状态为1，即“失败”。所以使用这个方法的话，一般是需要进行异常处理的，运行完这条语句后如果有异常捕获和处理的，会去运行后面的异常处理代码的（而os.\_exit\(\)方法则不会，它不会抛出异常）。



##### 简单示例：

```
>>> sys.argv
['']
>>> sys.path
['', 'C:\\Python27\\Lib\\idlelib',...]  # 元素太多，省略了
>>> sys.modules
{'heapq': <module 'heapq' from 'C:\Python27\lib\heapq.pyc'>,...}  # 元素太多，省略了
>>> sys.getdefaultencoding()
'ascii'
```



