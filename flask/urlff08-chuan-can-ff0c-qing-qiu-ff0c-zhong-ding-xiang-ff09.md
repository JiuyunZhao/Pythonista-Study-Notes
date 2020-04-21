# **URL（传参，请求，重定向）**

## 一、**URL传参**

* **良好的URL：**视图函数对应的url以/结尾是一种良好url，因为用户在访问的时候无论他有没有加上最后这个斜杠，都是能访问到的，相反，视图函数的url没有以/结尾，用户访问的时候却加上了这个/，那么用户是访问不到这个网页的。
* **使用path形式传参：**使用尖括号，如“&lt;value&gt;”将参数“value”通过URL传入视图函数，在视图函数中也需要有同名的参数。如果URL中有多个参数，则视图函数中也需要按顺序定义相同的参数。这样可以使用相同URL，但是因为参数不同而加载的数据却不同。如图：

![](/assets/flask-4.png)

* **path传参中指定URL参数类型：**在“&lt;&gt;”中使用形如“/args/&lt;int:value&gt;”来指定参数类型（此处指定的是int类型），注意冒号后面不能有空格，必须紧跟参数名。参数的常用数据类型有：
  * **string：**默认的类型，可以是任何不包含“/”和“\”的文本。
  * **int：**整型。
  * **float：**浮点类型。
  * **path：**和string类似，区别在于path类型可以包含“/”，即path类型为一个路径表示文本。
  * **uuid：**uuid字符串。
  * **any：**url的同一个位置可以是多个值。例如下图中“/user/111”和“/author/222”两个不同URL都将使用同一个视图函数：

![](/assets/flask-5.png)

* **自定义url参数类型：**自定义的类型需要继承类BaseConverter，即from werkzeug.routing import BaseConverter，并且需要把这个类注册到app.url\_map.converters中才会生效。有两种方式：
  * **第一种：**在子类中重新定义变量regex，根据自身需要写正则表达式，这个正则表达式就是参数需要满足的条件和格式。
  * **第二种：**在子类中重写to\_python和to\_url方法，这两个方法的参数都只有一个，即value。to\_python的value是url中传入的字符串，它的返回值会作为对应视图函数的参数值传入；而to\_url是用作函数url\_for的返回值的，它的参数value是url\_for函数传入的参数值，参数的格式和类型则是to\_python方法的返回值，to\_url的返回值则会在填充到url字符串中对应参数的位置。

```py
from flask import Flask, url_for
from werkzeug.routing import BaseConverter

app = Flask(__name__)


class TelConverter(BaseConverter):
    """
    第一种方式：只重新定义变量regex的正则表达式
    例如：该参数必须为11位的数字
    """
    regex = r'\d{11}'


class PlusConverter(BaseConverter):
    """
    第二种方式：重写方法to_python和to_url，前者用于视图函数的参数获取，后者用于url_for的url字符串获取
    例如：参数需要以+号来进行分割进行获取
    """

    def to_python(self, value):
        """例如传入的value为：python+27"""
        return value.split('+')

    def to_url(self, value):
        """例如传入的value为：['python', '27']"""
        return '+'.join(value)


# 将自定义的参数类型进行注册，注册后才能生效
app.url_map.converters['tel'] = TelConverter
app.url_map.converters['plus'] = PlusConverter


@app.route('/')
def hello_world():
    """
    这里的url_for函数返回的则是：/python+27/
    就是说返回的url字符串中的参数值是经过to_url方法处理后返回的
    """
    return url_for('get_plus', plus_args=['python', '27'])


@app.route('/<tel:telephone>/')
def get_telephone(telephone):
    """此处的参数则被要求必须是11位的数字"""
    return '你的手机号是：{}'.format(telephone)


@app.route('/<plus:plus_args>/')
def get_plus(plus_args):
    """
    如果访问：http://127.0.0.1:5000/python+27/
    则plus_args的值为['python', '27']
    也就是说传入的参数值是经过了to_python方法的处理返回的
    """
    return str(plus_args)


if __name__ == '__main__':
    app.run(debug=True)
```

* **使用查询字符串的方式传参：**即在浏览器的URL中使用“?key=value”的形式传递参数（多个参数之间使用“&”连接即可），在后台则使用“from flask import request”，然后使用“request.args.get\(key\)”来获取参数key的值value。

## 二、**URL请求**

需要导入request对象“from flask import request”，提交的请求的相关信息都在这个对象中了。

* **在route中指定视图函数的请求接收方式：**指定参数如“methods=\['GET', 'POST'\]”，methods参数的值是一个列表，在其中指定需要的请求方式即可，若没有指定则默认是GET，如果接收到的是POST请求，但却没有指定，则会报错。
* **get请求获取参数值：**使用查询字符串的方式传递参数时，使用“request.args.get\('param\_name\)'\)”即可。
* **post请求获取参数值：**使用form表单传递参数时，使用“request.form.get\('param\_name'\)”即可，这个“param\_name”是HTML中form表单元素的name属性的值，当然也需要在form中指定提交请求的方式为post“method="POST"”（特别注意在视图函数的route中也要指明post请求方式）。
* **获取request请求方式：**使用“request.method”即可，它的值为请求方式的全大写，比如“POST”。

## 三、**URL重定向**

需要“from flask import redirect”，它的第一个参数是一个URL字符串，这个URL字符串推荐使用“url\_for”来获取，第二个参数可以指定HTTP状态码，默认是302（302表示暂时性重定向，301表示永久性重定向）。如图：

![](/assets/flask-6.png)



