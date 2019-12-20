# **RequestHandler**

在Application中定义了路由及对应的RequestHandler处理类后，浏览器中输入对应的URL后，返回的页面其实是经过RequestHandler处理后render的页面；即RequestHandler用于处理请求，程序会为每一个请求创建一个RequestHandler对象，然后调用对应的HTTP方法。

RequestHandler类中常用的方法，在子类中可根据需要进行重写：

* **get\(\)和post\(\)：**在RequestHandler子类中可以重写get和post方法，当有数据提交到本路由的页面后，就可以根据提交的方式get或post执行对应的get或post方法，并返回方法中render函数传入的模板页面；
* **render\(template\_name,\*\*kwargs\)：**render方法有两个参数，第一个为模板名称，即需要跳转到的目标页面，第二个参数为需要传递进模板页面的参数字典，在模板中可以使用控制语句`{% %}`和表达语句`{{ }}`使用这些参数；
* **render\_string\(template\_name,\*\*kwargs\)：**将一个HTML模板内容以字符串的形式传递，后面的参数为传入HTML的参数；
* **write\(chunk\)：**一般是直接将字符串中的HTML代码写入浏览器客户端，chunk只能是`bytes/unicode_type/dict`中的一种，这个方法以chunk“块”写入HTTP的响应；
* **get\_argument\(argument\_name, default=object\(\)\)：**获取从客户端传入的参数值，但只能获取单个参数的值，没有获取到就返回默认的值；
* **get\_arguments\(argument\_names\)：**获取从客户端传入的某类参数值，即多个值；
* **request.files\['filename'\]：**获取从客户端传入的文件；
* **set\_status\(status\_code, reason=None\)：**设置在HTTP响应的状态码，status\_code是int类型，reason即状态码对应的信息，如果没有设置reason，那么就必须能在HTTP的responses中找到状态码对应的reason；
* **redirect\(url\)：**重定向到一个给定的路由。

```py
# 一个简单的RequestHandler子类示例
class DemoHandler(tornado.web.RequestHandler):
    # 处理get方式的请求，并跳转到demo页面
    def get(self, arg_get=None):
        if arg_get is None:
            arg_get = 'test arg'
        self.render(
            'demo.html',
            arg_demo=arg_get,
        )

    # 获取post方式请求中提交的参数demo_title，如果没有内容就重定向到demo页面
    def post(self):
        arg_title = self.get_argument('demo_title', None)
        if arg_title is None:
            self.redirect('/demo/')
```



