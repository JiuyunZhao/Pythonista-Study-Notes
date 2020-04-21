# 视图函数返回值

**返回HTML模板：**使用“from flask import render\_template”，在函数中传入相对于文件夹“templates”HTML模板路径名称字符串即可（默认模板路径），flask会自动到项目根目录的“templates”文件夹（创建flask项目时，PyCharm会自动创建两个空文件夹，其中一个就是“templates”）下寻找对应的HTML文件。

* **默认模板路径：**项目根目录下的templates文件夹。
* **自定义模板路径：**如果不想使用默认的模板路径，即项目根目录的templates文件夹，可以在实例化flask时指定参数template\_folder的值，以指定默认模板的路径。
* **模板传参：**如果需要给HTML模板传参，则在“render\_template”中使用变量名或字典进行传参即可（在Python2中，如果涉及中文，需要使用Unicode字符串）。

![](/assets/flask-9.png)

**返回Response对象：**视图函数返回响应Response给浏览器，一般来说只能是字符串和固定格式的元组，当然也可以自定义，但无论是返回哪种数据格式，最终都是被包装成一个Response对象返回给浏览器的。返回的是字符串时，其实也是被包装成Response对象返回给浏览器的。也可以是一个固定格式的元组：\(Response, status, headers\)，即响应体、状态码和头信息的元组，可以只返回一个Response，或者两个\(Response, status\)，或者全部返回\(Response, status, headers\)。当可以自定义返回的响应体时需要注意以下几点：

* **Response子类：**自定义的Response子类必须继承自from flask import Response（其实就是from werkzeug.wrappers import Response）。
* **response\_class：**使用app.response\_class=MyResponse使之生效。
* **force\_type\(response, environ=None\)：**当返回的数据类型，既不是字符串，也不是元组时，flask就会调用Response的force\_type方法来处理，如果不能处理就会返回错误，所以Response子类一般是需要重写这个方法来返回一个合法的数据，参数response即为传入的不合法的数据，可以经过处理后返回一个合法的Response对象。

```py
from flask import Flask, Response

app = Flask(__name__)


class MyResponse(Response):
    """自定义Response类"""
    @classmethod
    def force_type(cls, response, environ=None):
        """重写force_type方法，当参数既不是字符串，
        也不是(Response, status, headers)元组时会调用此方法"""
        if isinstance(response, list):
            response = Response('+'.join(response))

        # 这里需要包装成Response对象才能传入父类的force_type中，
        # 只传字符串会报错
        return super().force_type(response, environ)


# 指定自定义的Response类，使之生效
app.response_class = MyResponse


@app.route('/listresponse/')
def list_response():
    # 返回一个不合法的数据
    # 如果没有自定义的Response类来处理的话，就会报错
    return ['python', '36']


if __name__ == '__main__':
    app.run(debug=True)
```



