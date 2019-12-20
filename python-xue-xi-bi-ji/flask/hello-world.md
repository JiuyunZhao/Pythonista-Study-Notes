# **Hello World**

在PyCharm中新建一个Flask项目即可（此功能只有专业版的PyCharm才有，社区版的没有此功能，但Hello World足够简单，只有一个py文件，因此不用PyCharm也可以，不用PyCharm时一定注意运行py文件需要用虚拟环境中的Python解释器），需要注意：①项目名称的路径名最好全英文；②解释器选择的时候选择虚拟环境中的“Scripts”目录下的“python.exe”。如图：

![](/assets/Flask-2.png)

运行以下代码，并打开浏览器访问http://127.0.0.1:5000/可以看到视图函数return的字符串“Hello World!”：

```py
# -*- coding: utf-8 -*-
from flask import Flask

# 实例化一个Flask对象，使用__name__作为参数是，以后Flask的插件出现错误，可以方便定位问题
app = Flask(__name__)


# 此装饰器的作用是形成一个URL与视图函数的映射，app即前面的Flask实例对象
@app.route('/')
def hello_world():
    """视图函数：返回指定URL下的视图"""
    return 'Hello World!'


if __name__ == '__main__':
    app.run()  # 启动一个应用服务器，接受用户请求
```

![](/assets/Flask-3.png)

run\(\)表示启动一个测试应用服务器，用来接收用户的请求，真正部署到正式用的服务器上时就不能使用这个语句了。以下是它的一些参数的使用：

* **debug：**使用“app.run\(debug=True\)”或者在配置文件中设置“DEBUG=True”开启debug模式（默认是关闭的）。项目的debug模式主要有两个优点：一，当代码中发生错误时，只能在Python控制台看到错误信息，但是在网页上就会显示“Internal Server Error”，不会显示具体的错误信息，当设置了debug模式后，网页上就会显示出对应错误的Traceback信息，方便开发人员定位问题；二，设置debug模式后，当py文件的代码中有改变时，只需“Ctrl+S”，程序便会重新加载被改变的文件，并自动重启服务器，不要开发人员每次都去手动运行程序。
* **port：**设置访问的端口号时，传入port等于自定义的端口号。
* **host：**设置在局域网中别的计算机可以访问本计算机上的项目时，传入host等于0.0.0.0。



