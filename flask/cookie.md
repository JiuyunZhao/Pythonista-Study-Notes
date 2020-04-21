# cookie

**在网站中，HTTP请求是无状态的：**第一次请求成功后，第二次请求时服务器依然不知道这次请求的所属用户是谁。为了解决这个问题，在第一次请求成功后，服务器会生成并返回对应的cookie信息给浏览器，而浏览器在下一次请求同一个网站的时候就会自动（不需要再输入用户名和密码了）将其cookie信息附在请求上，此时服务器根据请求中的cookie信息就知道用户是谁了。

**cookie存储的数据量是有限的：**虽然不同浏览器会有具体的大小限制，但是一般不会超过4KB（4KB已经算是很大了），因此cookie只能用来存储一些小数据。

**cookie有效期：**服务器返回cookie个浏览器时需要给cookie设置有效期，当请求中的cookie超过有效期后，此cookie就算作无效的了，如果不设置则有效期会在session结束时自动失效。

**设置cookie：**使用\`from flask import Response\`中Response的set\_cookie方法

**set\_cookie\(self, key, value='', max\_age=None, expires=None, path='/', domain=None, secure=False, httponly=False, samesite=None\):**

* **key：**cookie的key。
* **value：**cookie的value。
* **max\_age：**设置cookie保存的时间（秒数），默认为None，表示保存时间与浏览器的session一致。
* **expires：**设置cookie的有效期，应该传入Python中的\`from datetime import datetime\`数据类型，或者是一个时间戳。需要注意的是这个参数使用的是格林尼治时间，也就是说我们设置时间时需要在本地时间的基础上减8小时（北京时间与格林尼治时间相差8小时的时间差），在显示时才能显示为我们使用的正确时间，不然显示的时间总是比我们看到的时间多8小时。
* **path：**设置使用cookie的url路径，默认为该域名下的url都有效。
* **domain：**设置使用cookie的域名，默认为该请求下的域名，它的子域名是不能使用此cookie的，所以一般跨域名使用cookie时才会用到此参数。比如：主域名为“www.csdn.net”，如果想要子域名“xxx.csdn.net”也能使用此cookie，domain参数就需要设置为“.csdn.net”。
* **secure：**此参数如果设置为True，则表示此cookie只能在HTTPS请求中才能使用。
* **httponly：**此参数如果设置为True，则表示不允许JavaScript使用此cookie，注意，此设置可能不是所有的浏览器都支持。
* **samesite：**限制cookie的使用范围，使其只能在“同一站点”才能使用此cookie。

**注：**如果同时设置了max\_age和expires参数，则以max\_age为准，会忽略expires参数。如果这两个参数多没有设置，则和浏览器的session保持一致，即关掉整个浏览器时，session自动结束，cookie也自动失效。

**删除cookie：**使用Response对象的delete\_cookie方法删除对应key值的cookie即可，这个方法其他参数请参考set\_cookie方法。

**获取cookie：**使用request.cookies.get方法即可获取对应的cookie信息。

**简单示例：**

```py
from datetime import datetime

from flask import Flask, Response

from views import bp

app = Flask(__name__)
app.register_blueprint(bp)
# 主域名和子域名因为没有注册，所以需要在C:\Windows\System32\drivers\etc\hosts文件中配置一下
# 127.0.0.1 hai.com
# 127.0.0.1 hi.hai.com
app.config['SERVER_NAME'] = 'hai.com:5000'


@app.route('/')
def hello_world():
    resp = Response('Hello World! -- set cookie!')
    # 有效期截止到2019-08-31 00:00:00
    expire_time = datetime(year=2019, month=8, day=31, hour=16, minute=0, second=0)
    # 有效期截止到31天之后
    # expire_time = datetime.now() + timedelta(days=30, hours=16)
    # expires参数使用的是格林尼治时间，比北京时间多8小时，所有想要显示为北京时间，需要在设置的时间上减8小时
    # 同时设置了max_age和expires，则以max_age参数为准，会忽略expires参数。
    # .hai.com表示以此为后缀的域名都可用此cookie
    resp.set_cookie('username', 'xiaoming', max_age=60, expires=expire_time, domain='.hai.com')
    return resp


@app.route('/del_cookie/')
def del_cookie():
    resp = Response('Hello World! -- delete cookie!')
    # 删除cookie中对应key的信息
    resp.delete_cookie('username')
    return resp


if __name__ == '__main__':
    app.run(debug=True)
```

**views.py文件**

```py
from flask import Blueprint, request

# 新建蓝图对象，用于子域名的访问
bp = Blueprint('hi', __name__, subdomain='hi')


@bp.route('/')
def index():
    username = request.cookies.get('username')
    return 'hi 首页: {}'.format(username)
```



