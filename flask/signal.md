# 信号机制

Flask中有内置的一些信号，也可以通过三方库blinker自定义信号，其实Flask内置的信号也是优先使用的blinker库，如果没有安装blinker才会使用自定义的信号机制。可以通过点击任意导入的内置信号查看源码，同时也可以看到具体有哪些内置的信号。

一般安装Flask时会自动安装blinker，如果没有安装通过pip安装即可：pip install blinker。

为了更好的理解信号的具体操作，下面先看下怎么通过blinker自定义信号，再列出内置的信号。



#### blinker自定义信号

其实除非是使用内置的信号，否则使用自定义的信号感觉上作用类似公共函数，在需要的时候调用就行。以下为blinker自定义信号步骤：

* **创建命名空间：**my\_namespace=Namesapce\(\)，可以创建多个命名空间。
* **创建信号：**my\_signal = my\_namespace.signal\('my\_signal'\)，同一个命名空间中可以创建多个信号。
* **监听信号：**my\_signal.connect\(my\_func\)，当监听到信号后，会去执行指定的函数my\_func，my\_func函数定义的时候需要注意，必须指定参数sender（无论在函数中是否用到）。
* **发送信号：**my\_signal.send\(\)，发送信号，此时会执行“监听信号”中指定的函数my\_func，如果send方法中没有指定sender参数，那么my\_func函数中的sender参数默认为None。

**简单示例：**

```py
from flask import Flask, request, g

from signals import login_signal

app = Flask(__name__)


@app.route('/login/')
def login():
    # 以查询字符串的方式获取用户名：http://127.0.0.1:5000/login/?username=xiaoming
    username = request.args.get('username')
    if username:
        # 可以将信息存储在g对象中，然后在监听函数中直接使用g对象获取信息即可
        g.username = username
        
        # 发送信号
        login_signal.send()
        
        # 也可以通过指定参数名的方式传入，这种方式需要在监听函数中也定义好对应的参数
        # login_signal.send(username=username)
        
        return '登录成功'
    else:
        return '请输入用户名！'


if __name__ == '__main__':
    app.run()
```

**signals.py**

```py
from flask import g
from blinker import Namespace

# 创建命令空间
login_namespace = Namespace()

# 创建信号，如果已存在同名的信号，可以放在不同的命名空间中
login_signal = login_namespace.signal('login')


# def print_login_info(sender, username):
#     # 通过定义参数的方式获取信息
#     print('[{}]登录成功!'.format(username))


# 定以监听函数，此函数必须要定义sender参数，无论此参数在函数中是否被用到
def print_login_info(sender):
    # 通过g对象的方式获取信息
    print('[{}]登录成功!'.format(g.username))


# 监听信号，并指定监听函数，此函数会在login_signal.send()方法调用后执行
login_signal.connect(print_login_info)
```

#### Flask内置信号

因为内置信号已经定义好了信号本身和对应的发送操作，所以在代码中只需要定义对应的监听操作即可，包括调用信号的\`connect\`方法和定义对应的监听函数。

如果想要看各个内置信号的监听函数的参数，可以通过将监听函数的参数定义为\`\*args\`和\`\*\*kwargs\`查看具体传入了哪些参数。

* **template\_rendered：**模板渲染完成后会发送的信号。
* **before\_render\_template：**模板渲染之前会发送的信号。
* **request\_started：**请求开始后会发送的信号。
* **request\_finished：**请求完成后会发送的信号。
* **request\_tearing\_down：**request对象被销毁后会发送的信号。
* **got\_request\_exception：**请求发生异常时（包括视图函数内发生的异常）会发送的信号。经常会监听这个信号，用来记录网站的异常信息。
* **appcontext\_tearing\_down：**app上下文对象被销毁后会发送的信号。
* **appcontext\_pushed：**app上下文对象被推入栈中后会发送的信号。
* **appcontext\_popped：**app上下文对象被移出栈中后会发送的信号。
* **message\_flashed：**调用了Flask的\`flasked\`方法后会发送的信号。



