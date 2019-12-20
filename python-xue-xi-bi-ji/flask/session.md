# session

#### session与cookie：

cookie是一项浏览器的技术，而不是服务器的技术，服务器端是无法直接操作cookie的，只能通过返回Response响应告诉浏览器怎么操作cookie。而session则更像是一种解决方案，一种在服务器端存储授权信息的解决方案，不同的语言，不同的框架对于session的实现方式可能都是不同的，对session的理解和操作可能都不一样。同时，session更是一种解决cookie安全隐患的方式，用户名和密码等信息都是存放在session中的，而session内容是无法被用户直接看到的。



#### session存储：

服务器生成的session信息既可以存放在服务器中（cookie中就只有session\_id），也可以加密后存放在cookie中（浏览器拿到cookie后也不能知道加密的session内容）， Flask就是采用的第二种机制。



#### 操作session：

使用\`from flask import session\`对象，session对象就相当于一个字典，操作session时可以像操作字典一样去操作它，如：session\['username'\] = 'xiaoming'，session.get\('username'\)，session.pop\('username'\)，session.clear\(\)等。代码中导入此session对象后直接操作使用即可，它会自动加入cookie中并返回给浏览器，也会自动从访问的请求中的cookie中提取出来，所以不需要我们手动将它加入到Response的cookie中，也不需要手动从request的cookie中获取出来。



#### session加密：

使用session时，SECRET\_KEY是必须配置的 ，这个值用来加密session内容然后返回给浏览器。如：\`app.config\['SECRET\_KEY'\] = os.urandom\(24\)\`，即24位的随机数。



#### session有效期：

session默认有效期和cookie一样，直到浏览器回话结束。可以通过设置\`session.permanent=True\`来设置自定义的有效期，而此时默认有效期为31天，如果配置了PERMANENT\_SESSION\_LIFETIME，则以此配置项的值为准，此配置项的类型为\`from datetime import timedelta\`类型。如：\`app.config\['PERMANENT\_SESSION\_LIFETIME'\] = timedelta\(days=7\)\`，即有效期为7天。



#### 简单示例：

```py
import os
from datetime import timedelta

from flask import Flask, session

app = Flask(__name__)
# 使用session必须配置此SECRET_KEY，用于session的加密
app.config['SECRET_KEY'] = os.urandom(24)
# 没有配置此项时，默认为31天，配置后设置session.permanent=True时，以此配置项的值为准。
app.config['PERMANENT_SESSION_LIFETIME'] = timedelta(days=7)


@app.route('/')
def hello_world():
    # 此session对象其实就相当于一个字典，可以当做字典来使用
    # 此处设置了session后，会自动将此session内容加密后放入cookie中返回给浏览器
    # session在浏览器cookie中的存放形式为：在cookie中添加一个“键值对”，key为“session”，value为session内容加密后的字符串。
    session['username'] = 'xiaoming'
    
    # 设置session的持久化，默认为False，即有效期至浏览器回话结束，设置为True表示有效期为31天
    session.permanent = True
    
    return 'Hello World!'


@app.route('/username/')
def get_username():
    # 获取session信息
    username = session.get('username')

    return username or '没有用户登录！'


@app.route('/delete_user/')
def del_user():
    # 删除session信息
    username = session.pop('username')

    return '{}成功删除！'.format(username) if username else '删除失败！'


if __name__ == '__main__':
    app.run(debug=True)
```



