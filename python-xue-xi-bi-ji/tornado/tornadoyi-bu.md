\# **Tornado异步**

**同步：**同步的意思就像是代码的顺序执行，当这一行的代码没有执行完时，就会一直等它，直到它执行完了再执行下一行，所以遇到耗时较长的代码行时，这行代码说不定就是整个程序的“锅”了；

**阻塞：**当程序在某一处代码“停住”时，其他的代码就不能执行了，程序就阻塞在了此处了，比如同步的程序中卡在某行耗时长的代码时，这行代码就阻塞了后面代码的执行；

**异步：**异步就像是你做你的，我不用等你，你做完了自然会往下执行，我还是自己做自己的，比如web中，当一个请求没有处理完时，程序还可以同时处理另一个请求，不用等你这请求处理完再来处理下一个请求，所以在访问者较多时，异步的优势就不言而喻了。

Tornado的异步处理主要在以下几点（可以保持当前客户端连接不关闭，不必等当前请求处理完成后再处理下一个请求）：

1. 异步装饰器@tornado.web.asynchronous：在web方法比如get上使用异步装饰器表明这是一个异步处理方法，被这个装饰器装饰的方法永远不会自己关闭客户端连接（Tornado默认在处理函数返回时自动关闭连接），必须使用finish方法手动关闭。
2. 异步HTTP客户端处理类tornado.httpclient.AsyncHTTPClient：

   * AsyncHTTPClient的实例可以执行异步的HTTP请求。
   * AsyncHTTPClient的fetch方法不会返回url的调用结果（HTTPResponse），而是使用了一个callback参数来指定HTTPResponse的处理方法，将HTTPResponse作为这个指定方法的参数传入进去进行相关的处理。
   * 在fetch方法的callback参数指定的处理方法的结尾处可以调用tornado.web.RequestHandler的finish方法来关闭客户端链接。

3. 异步处理模块tornado.gen：

   * 装饰器@tornado.gen.engine：告诉tornado被装饰的方法将使用tornado.gen.Task类。
   * 使用yield生成tornado.gen.Task类的实例，将我们想要的调用（比如：fetch方法）和需要传入该调用函数的参数传入这个Task实例：相比于第2点的fetch方法的callback回调功能将处理方法分成两个部分，这个异步生成器的好处在于，一是该请求的处理会在yield处停止执行，直到这个回调函数返回，但这并不会影响其他请求的处理，它依然是异步的，二是回调函数中可能还有回调函数，这样循环下去不容易维护，但是这个异步生成器可以让所有处理都在一个方法里，易开发和维护。

**同步示例：**

当运行以下代码时，在两个窗口短时间内（10s）访问如http://localhost:8000/?word=tornado时，在第一个窗口（请求）未加载出来时（这儿设置了等待10s），第二个窗口（请求）无法处理并加载，第一个窗口加载完成后，第二个窗口也要等待10s后才能加载出来，相当于第二个窗口等待了20s。

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web
import tornado.httpclient

import urllib
import time

from tornado.options import define, options

define('port', default=8000, help='run on the given port', type=int)


class IndexHandler(tornado.web.RequestHandler):
    def get(self):
        query = self.get_argument('word')
        client = tornado.httpclient.HTTPClient()
        response = client.fetch('https://www.baidu.com/baidu?' +
                                urllib.urlencode({'word': query}))
        time.sleep(10)  # 在这个请求未处理完之前，其他的请求只能“排队”
        self.write('<h1>Tornado Synchronous Test(request_time): %s</h1>' % response.request_time)


if __name__ == '__main__':
    tornado.options.parse_command_line()  # 可以从命令行解析命令
    app = tornado.web.Application(handlers=[(r'/', IndexHandler)])
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

**异步示例：**

与同步的代码相比，不同之处在于三点：①@tornado.web.asynchronous，②tornado.httpclient.AsyncHTTPClient\(\)，③fetch的callback参数（需要在回调函数中使用self.finish\(\)，不然浏览器不会将处理结果加载出来，因为它不知道你已经处理完了）。同样运行代码发现访问如http://localhost:8000/?word=tornado时，如果开了两个窗口，这两个窗口都只需等待10s就可加载出来，第二个窗口无需等待第一个窗口加载完成。

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web
import tornado.httpclient

import urllib
import time

from tornado.options import define, options

define('port', default=8000, help='run on the given port', type=int)


class IndexHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    def get(self):
        query = self.get_argument('word')
        client = tornado.httpclient.AsyncHTTPClient()
        client.fetch('https://www.baidu.com/baidu?' +
                     urllib.urlencode({'word': query}),
                     callback=self.response_process)

    def response_process(self, response):
        time.sleep(10)  # 在这个请求未处理完之前，其他的请求不用“排队”，可以直接处理
        self.write('<h1>Tornado Synchronous Test(request_time): %s</h1>' % response.request_time)
        self.finish()


if __name__ == '__main__':
    tornado.options.parse_command_line()  # 可以从命令行解析命令
    app = tornado.web.Application(handlers=[(r'/', IndexHandler)])
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```

**异步生成器示例：**

异步效果与之前的代码一样，但与之前的异步代码相比，不同之处在于：①tornado.gen，②@tornado.gen.engine（注意调用顺序），③response是通过yield返回的。与之前的异步代码优势在于易于维护，之前的异步代码可能在callback的回调中还有callback回调，如果回调太多就很不好维护了。

```py
#!/usr/bin/env python
# -*- coding: utf-8 -*-

import tornado.httpserver
import tornado.ioloop
import tornado.options
import tornado.web
import tornado.httpclient
import tornado.gen

import urllib
import time

from tornado.options import define, options

define('port', default=8000, help='run on the given port', type=int)


class IndexHandler(tornado.web.RequestHandler):
    # 以下装饰器的使用顺序不能乱
    @tornado.web.asynchronous
    @tornado.gen.engine
    def get(self):
        query = self.get_argument('word')
        client = tornado.httpclient.AsyncHTTPClient()
        response = yield tornado.gen.Task(client.fetch,
                                          'https://www.baidu.com/baidu?' +
                                          urllib.urlencode({'word': query})
                                          )
        time.sleep(10)
        self.write('<h1>Tornado Synchronous Test(request_time): %s</h1>' % response.request_time)
        self.finish()


if __name__ == '__main__':
    tornado.options.parse_command_line()  # 可以从命令行解析命令
    app = tornado.web.Application(handlers=[(r'/', IndexHandler)])
    http_server = tornado.httpserver.HTTPServer(app)
    http_server.listen(options.port)
    tornado.ioloop.IOLoop.instance().start()
```



