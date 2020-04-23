# **Tronado的web应用安全（cookie和CSRF/XSRF）**

安全cookies是web应用的安全防范之一，浏览器中的cookies存储了用户的个人信息，当然包括了某些重要的敏感的信息，如果一些恶意的脚本得到甚至修改了用户的cookies的信息，用户的信息就得不到安全的保障，所以应该对用户的cookies进行保护。Tornado的安全cookies可以对cookies签名进行安全加密，以检查cookies是否被修改过，因为恶意脚本不知道安全密钥，所以无法修改（但是恶意脚本仍然可以截获cookies来“冒充”用户，只是不能修改cookies而已，这也是另外一个安全隐患，本文并不讨论这点）。

Tornado的get\_secure\_cookie\(\)和set\_secure\_cookie\(\)可以安全的获取和发送浏览器的cookies，可以防止浏览器中的恶意修改，但是为了使用这个功能，必须在tornado.web.Application的settings中设置cookie\_secret，其值为一个唯一的随机字符串（比如：base64.b64encode\(uuid.uuid4\(\).bytes + uuid.uuid4\(\).bytes\)可以产生一个唯一的随机字符串）。

CSRF或XSRF，即跨站请求伪造，是web应用都涉及到的一个安全漏洞，它利用了浏览器的一个安全漏洞：浏览器允许恶意攻击者在受害者网站注入脚本使未授权请求代表一个已登录用户。即别人可以“冒充”你做一些事情，而服务器也会认为这些操作是你做的。

为了防止XSRF攻击，应该注意：一是开发者考虑某些重要的请求时需要使用POST方法，二是Tornado的一个防范伪造POST的功能（这个功能是一种策略，tornado也实现了这种策略），就是在每个请求中包含一个参数值（隐藏的HTML表单元素值）和存储的cookie值，若两者（称之为令牌）匹配上了，则证明请求有效，当某个不可信的站点没有访问cookie数据的权限时，它就不能在请求中包含这个令牌cookie值，自然就无法发送有效的请求了。tornado中使用这个功能需要在tornado.web.Application的settings中设置xsrf\_cookies，值为True，同时必须在HTML的表单中包含xsrf\_form\_html\(\)函数，以此形成cookie令牌。

tornado的用户验证可以使用装饰器@tornado.web.authenticated，使用这个装饰器时需注意：①必须重写get\_current\_user\(\)方法，这个返回的值将赋给self.current\_user，②被装饰的方法被执行前会检查self.current\_user的bool值是否为False（默认是None），若为False则会重定向到tornado.web.Application的settings中login\_url指定的URL，③需要在tornado.web.Application的settings中login\_url的URL，以便用户验证self.current\_user之前可以重定向到这个URL。④当Tornado构建重定向URL时，它还会给查询字符串添加一个next参数，其值为重定向之前的URL，可以使用如self.redirect\(self.get\_argument\('next', '/'\)\)这样的语句在用户验证成功后回到原来的页面。

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-
import tornado.httpserver
import tornado.ioloop
import tornado.web
import tornado.options
import os.path
import base64
import uuid

from tornado.options import define, options

define('port', default=8000, help='run on the given port', type=int)


class BaseHandler(tornado.web.RequestHandler):
    def get_current_user(self):
        """重写此方法，返回的值将赋给self.current_user"""
        return self.get_secure_cookie('username')  # 获取cookies中username的值


class LoginHandler(BaseHandler):
    def get(self):
        self.render('login.html')

    def post(self):
        self.set_secure_cookie('username', self.get_argument('username'))  # 将请求中的username值赋给cookie中的username
        self.redirect('/')


class HomeHandler(BaseHandler):
    @tornado.web.authenticated  # 使用此装饰器必须重写方法get_current_user
    def get(self):
        """被@tornado.web.authenticated装饰的方法被执行前会检查self.current_user的bool值是否为False，不为False时才会执行此方法"""
        self.render('index.html', user=self.current_user)


class LogoutHandler(BaseHandler):
    def get(self):
        if self.get_argument('logout', None):
            self.clear_cookie('username')  # 清楚cookies中名为username的cookie
            self.redirect('/')


if __name__ == '__main__':
    tornado.options.parse_command_line()

    settings = {
        'template_path': os.path.join(os.path.dirname(__file__), 'templates'),
        'cookie_secret': base64.b64encode(uuid.uuid4().bytes + uuid.uuid4().bytes),  # 设置cookies安全，值为一个唯一的随机字符串
        'xsrf_cookies': True,  # 设置xsrf安全，设置了此项后必须在HTML表单中包含xsrf_form_html()
        'login_url': '/login'  # 当被@tornado.web.authenticated装饰器包装的方法检查到self.current_user的bool值为False时，会重定向到这个URL
    }

    application = tornado.web.Application([
        (r'/', HomeHandler),
        (r'/login', LoginHandler),
        (r'/logout', LogoutHandler)
    ], **settings)

    http_server = tornado.httpserver.HTTPServer(application)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Welcom Back!</title>
</head>
<body>
    <h1>Welcom back, {{ user }}</h1>
</body>
</html>
```

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Please Log In</title>
</head>
<body>
    <form action="/login" method="POST">
        {% raw xsrf_form_html() %}<!-- 这其实是一个隐藏的<input>元素，定义了“_xsrf”的值，会检查POST请求以防止跨站点请求伪造 -->
        Username: <input type="text" name="username" />
        <input type="submit" value="Log In" />
    </form>
</body>
</html>
```



