# **UIModule**

tornado.web.UIModule子类为自定义的UI模板，在HTML中使用`{% module module_name %}`时会自动包含`module_name`类中render方法返回的字符串（一般为包含HTML标签内容的字符串），如果是HTML文件，返回的就是HTML模板中内容的字符串，返回的HTML内容字符串的“style”可以由其他的方法设置。

继承UIModule时一般需要重写render方法，以`render_string`方法返回特定格式HTML内容。

UIModule类中常用的方法，可以重写这些方法以满足自己对HTML格式的要求：

    * **embedded\_css\(\)：**函数中返回的CSS规则被包裹在`<style>`中，并被直接添加到`<head>`的闭标签之前；
    * **css\_files\(\)：**返回外部CSS文件路径的字符串；
    * **embedded\_javascript\(\)：**函数中返回的js代码字符串将被`script`包围，并被插入到`<body>`的闭标签中；
    * **javascript\_files\(\)：**返回外部js文件路径的字符串；
    * **html\_body\(\)：**函数中返回的包含HTML标签的字符串在闭合的`</body>`标签前添加进去。
```py
# 自定模板，可将其设置的HTML内容嵌入到其他HTML模板中
class MyModule(tornado.web.UIModule):
    def render(self, arg_mod):
        return self.render_string(
            'module/mod.html',
            arg_mod=arg_mod,
        )

    # 设置外部的css文件
    def css_files(self):
        return '/static/css/style.css'

    # 设置外部的js文件
    def javascript_files(self):
        return '/static/js/jquery-3.2.1.js'
```



