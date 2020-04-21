# Hello World

**安装Tornado：**pip install tornado

#### **helloworld简单示例：**

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import tornado.web
import tornado.ioloop


# 用于处理网页的请求
class MainHandler(tornado.web.RequestHandler):
    def get(self, *args, **kwargs):
        self.write('Hello Tornado!')

# 设置不同路由的网页对应的处理类
app = tornado.web.Application([
    (r'/', MainHandler),
])

# 开始主程序I/O循环
if __name__ == '__main__':
    app.listen(8888)
    tornado.ioloop.IOLoop.instance().start()
```

运行Tornado的helloworld所需的基本组成：

1. **app.listen\(8888\)：**设置服务器监听的端口，这里可以随意设置可用的port，比如：8080；
2. **tornado.ioloop.IOLoop.instance\(\).start\(\)：**开启I/O循环，响应客户端的操作；
3. **tornado.web.Application：**实例化一个web应用类，用于处理用户的请求，可传入一个列表，列表中每个元素由一个访问路由和对应的处理类构成；
4. **tornado.web.RequestHandler：**定义请求处理类，用于处理对应的请求。



