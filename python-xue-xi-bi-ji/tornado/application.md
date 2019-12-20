# **Application**

1. **Application：**tornado.web.Aplication新建一个应用，可通过直接实例化这个类或实例化它的子类来新建应用。
2. **handlers：**实例化时至少需要传入参数handlers，handlers为元素为元组的列表，元组中第一个元素为路由，第二个元素为路由对应的RequestHandler处理类；路由为正则表达式，当正则表达式中有分组时，访问时会将分组的结果当做参数传入模板中。
3. **settings：**还有一个实例化时经常用到的参数settings，这个参数是一个字典：

    * **template\_path：**设置模板文件（HTML文件）；
    * **static\_path：**设置静态文件（如图像、CSS文件、JavaScript文件等）的路径；
    * **debug：**设置成True时开启调试模式，tornado会调用tornado.autoreload模块，当Python文件被修改时，会尝试重新启动服务器并在模板改变时刷新浏览器（在开发时使用，不要在生产中使用）；
    * **ui\_modules：**设置自定义UI模块，传入一个模块名（变量名）为键、类为值的字典，这个类为tornado.web.UIModule的子类，在HTML中使用{% module module\_name %}时会自动包含module\_name类中render方法返回的字符串（一般为包含HTML标签内容的字符串），如果是HTML文件，返回的就是HTML模板中内容的字符串。

4. **数据库：**可以在Application的子类中连接数据库“self.db = database”，然后在每个RequestHandler中都可以使用连接的数据库“db\_name = self.application.db.database”。

```py
# 定义了模板路径和静态文件路径后，在使用到模板和静态文件的地方就不需要逐一添加和修改了
settings = {
    'template_path': os.path.join(os.path.dirname(__file__), 'templates'),
   'static_path': os.path.join(os.path.dirname(__file__), 'static'),
    'debug': True,
}

# 对应的RequestHandler类代码未贴出来 
app = tornado.web.Application(
    handlers=[(r'/',MainHandler), 
　　　　　　　　 (r'/home', HomePageHandler), 
　　　　　　　　 (r'/demo/([0-9Xx\-]+)', DemoHandler), # 有分组时会将分组结果当参数传入对应的模板中 
 　　　　　　　　], 
　　　　　　**settings 
)
```



