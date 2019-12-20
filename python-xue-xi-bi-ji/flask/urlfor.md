# url\_for

**URL反转：**反转是指通过视图函数名称得到其对应的URL（有反转也就有正转，即通过URL得到视图函数返回的内容，也就是我们平时的访问网页了），需要“url\_for\(endpoint, \*\*values\)”，第一个参数endpoint如果没有指定则使用视图函数名称字符串，第二个参数是需要传入URL的参数（如果有），如果传入URL的参数有多余的，则多余的参数就会以查询字符串的方式添加在URL后面。如图（“test\_args”为视图函数名，“value”为参数名）：

![](/assets/flask-10.png)

**链接URL和静态文件URL：**两者都可以使用“url_for”来得到对应的url。链接使用如`<a href="{{ url_for('func_name', *args) }}">xxx</a>`，此时传入的是视图函数名称及其参数；静态文件则使用如`<link rel="stylesheet" href="{{ url_for('static', filename='css/index.css') }}" >`，其中“static”是静态文件存放的总文件夹，“filename”的值则是静态文件的具体路径。如图：：

![](/assets/flask-11.png)

