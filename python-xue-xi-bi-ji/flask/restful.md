# RESTful

RESTful是用于前台和后端进行通信的一种规范或者说一种风格，采用的是HTTP和HTTPS协议，数据传输的格式使用的都是JSON，而不是XML。通常，RESTful的URL中只有名词，没有动词，而且名词在复数的情况就应该使用其复数的形式。网上有专门讲解RESTful的详细教程，本文只是Flask中用于实现RESTful规范的插件Flask-RESTful的学习笔记，就不详细介绍RESTful了。

**安装：**pip install Flask-RESTful

**使用场景：**一般来说，如果URL是用于向前台返回json数据，就可以考虑使用这个RESTful插件了。



#### RESTful基础使用

RESTful的使用重点在视图类的定义，包括请求方法的定义，基础使用步骤如下，具体参见示例代码：

* **导入：**from flask\_restful import Api, Resource
* **创建Api对象：**api = Api\(app\)
* **创建视图类：**视图类需要继承Resource，然后定义对应的请求方法即可，返回值为json格式数据，即字典
* **添加url：**使用\`api.add\_resource\`方法添加url相关信息，第一个参数为视图类的名称，第二个参数是url字符串，并且可以有多个url，第三个参数是endpoint等关键字参数。

**简单示例：**

```py
from flask import Flask, url_for
from flask_restful import Api, Resource

app = Flask(__name__)
# 创建API对象
api = Api(app)


class LoginView(Resource):
    # 如果url中定义了参数，请求方法也需要定义对应的参数
    # 如果add_resource中有url没有这个参数，就需要给这个参数设置一个默认值
    def post(self, username='xiaowang'):
        # 如果使用的是restful，返回值的类型就需要是json格式数据，即字典
        return {'username': 'xiaoming'}


# 可以在url中指定参数，就和“app.route”中定义参数一样，也需要在对应请求方法里也定义这个参数
# add_resource定义url时，可以传入多个url，比如第二个url是没有传入参数的，这时候就需要在定义请求方法时将这个参数设置一个默认值，不然会报错
# endpoint参数用于url_for函数反转，如果不指定endpoint参数，那么将使用视图类的名称的全小写作为endpoint
api.add_resource(LoginView, '/login/<username>/', '/login/', endpoint='login')

# 创建请求上下文，测试endpoint参数
with app.test_request_context():
    # 测试url：/login/
    print(url_for('login'))
    # 测试url：/login/<username>/
    print(url_for('login', username='xiaoming'))

if __name__ == '__main__':
    app.run()
```

#### RESTful参数验证

对于请求中的form表单验证，RESTful也有实现对应的验证方法，即flask\_restful.reqparse，在对应的请求方法中进行form表单数据验证的步骤如下，具体参见示例代码：

* **创建解析对象：**parser = reqparse.RequestParser\(\)
* **指定要验证的字段：**使用parser.add\_argument\(\)进行具体的验证，add\_argument的具体参数如下：
  * **default：**默认值，如果这个字段没有值，则使用默认值。
  * **required：**此字段是否必须，默认为False，如果设置为True，则表示此字段必须输入。
  * **type：**指定此字段的数据类型，可以是Python中的数据类型，也可以是\`flask\_restful.inputs\`中的数据类型，如果指定了具体的数据类型，将会使用指定的类型对字段值进行强制转换，转换失败则表示验证失败。需要注意的是，如果指定的是Python数据类型，就会将此字段的值传入Python的数据类型转换函数，即只会给转换函数传入一个参数值，如果对应的Python类型转换函数需要不只一个参数时，那么就会验证错误，此时，就需要考虑使用\`flask\_restful.inputs\`中的数据类型了。常用的有\`inputs.url\`、\`inputs.regex\`和\`inputs.date\`，其他数据类型可以进\`inputs.py\`源代码中查看。
  * **choices：**可提交的选项，列表类型，即提交的字段值只能是列表中的值才能通过验证。
  * **help：**错误信息，验证失败后的错误提示信息。
  * **trim：**是否去掉字段值的前后空格。
* **解析验证并返回结果：**args = parser.parse\_args\(\)，args为验证结果字典，即每个字段的值

**简单示例：**

```py
from flask import Flask, url_for
from flask_restful import Api, Resource, reqparse

app = Flask(__name__)
# 创建API对象
api = Api(app)


class LoginView(Resource):
    def post(self):
        # 创建解析对象
        parser = reqparse.RequestParser()
        # 获取form中的字段，并指定对应的验证方式
        parser.add_argument('username', type=str, help='用户名验证错误！')
        parser.add_argument('password', type=str, help='密码验证错误！')
        # 开始解析并验证字段信息
        # 返回结果args是一个字典，如：{'username': 'xiaozhi', 'password': '123456'}
        args = parser.parse_args()
        # 如果验证失败则不会正常返回，而是返回关于验证失败的一个json数据，即字典数据，可打印查看具体信息和字典结构
        print(args)

        # 如果使用的是restful，返回值的类型就需要是json，即字典
        return {'username': 'xiaoming'}


api.add_resource(LoginView, '/login/', endpoint='login')


if __name__ == '__main__':
    app.run()
```

#### RESTful标准化返回值

标准化返回值，即对同一个视图类中的请求方法返回的json数据进行统一化和规范化，具体见示例代码：

* **字典定义：**需要标准化的字典数据不再是直接定义在请求方法中了，而是定义在类中，作为统一的返回值字典格式。
* **标准化：**在对应的请求方法上使用装饰器“from flask\_restful import marshal\_with”，并给装饰器传入对应的字典，只有传入了装饰器的字典才会作为此请求方法的返回值。
* **返回值：**如果请求方法使用了marshal\_with装饰器，那么指定的字典就会是最后的返回值格式，并且会在请求方法最后的return对象中查找字典中的数据信息，并用作最后的返回值，需要注意的是，字典中有的信息都会输出，但是return对象中有多余的key项是不会添加到返回的字典中的。

**简单示例（只列了两个关键文件，其他文件省略了）：**

**models.py**

```py
from exts import db


class User(db.Model):
    """user表，存储用户信息"""
    __tablename__ = 'user'
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(50))
    email = db.Column(db.String(50))


# 中间表，存储article表和tag表的对应关系
article_tag_table = db.Table(
    'article_tag',
    db.Column('article_id', db.Integer, db.ForeignKey('article.id'), primary_key=True),
    db.Column('tag_id', db.Integer, db.ForeignKey('tag.id'), primary_key=True))


class Article(db.Model):
    """article表，存储文章信息"""
    __tablename__ = 'article'
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(100))
    content = db.Column(db.Text)
    author_id = db.Column(db.Integer, db.ForeignKey('user.id'))

    # 文章对应的作者，是多对一的关系
    author = db.relationship('User', backref='articles')

    # 文章对应的标签，是多对多的关系
    tags = db.relationship('Tag', secondary=article_tag_table, backref='tags')


class Tag(db.Model):
    """tag表，存放文章的标签信息"""
    __tablename__ = 'tag'
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(50))
```

**主文件**

```py
from flask import Flask
from flask_restful import Api, Resource, fields, marshal_with

import config
from exts import db
from models import User, Article, Tag

app = Flask(__name__)
# config文件中就定义了数据库连接的信息，一行代码：SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}?charset=utf8'.format(username='root', password='123456', host='127.0.0.1', port=3306, database='restful_demo')
app.config.from_object(config)
api = Api(app)
# exts文件中就两句代码：from flask_sqlalchemy import SQLAlchemy；db = SQLAlchemy()
db.init_app(app)


class ArticleView(Resource):
    # 定义字典信息，用作请求方法的标准返回值
    resource_fields = {
        # 字典key值会用作返回值字典的key值，并且如果没有在数据类型中指定attribute参数值，将会以这个key值在return对象中查找key值，当然，指定了attribute值，就会以attribute参数值去查找
        # 字典的value值为flask_restful.fields中的数据类型
        'article_title': fields.String(attribute='title'),
        'content': fields.String,
        # 如果value值是另外一个对象，比如这里是author对象，就需要使用fields.Nested来获取该对象的详细信息
        'author': fields.Nested({
            'username': fields.String,
            'email': fields.String
        }),
        # 如果value值为对象列表，比如这里是tag对象列表，则需要在fields.Nested外在加一层fields.List
        'tags': fields.List(fields.Nested({
            'name': fields.String,
        })),
        # 如果该key值没有找到对应信息，比如这里的read_count在article对象中就没有这个字段，可以使用default指定默认值
        'read_count': fields.Integer(default=0)
    }

    # 必须使用flask_restful.marshal_with装饰器，并传入对应字典，此请求方法才会返回这个字典
    @marshal_with(resource_fields)
    def get(self, article_id):
        article = Article.query.get(article_id)
        return article


api.add_resource(ArticleView, '/article/<article_id>/', endpoint='article')


@app.route('/')
def hello_world():
    # 往数据库表中添加信息，方便测试
    user = User(username='xiaoming', email='123456.@qq.com')
    article = Article(title='python', content='python xxxxxxxx')
    article.author = user
    tag_flask = Tag(name='flask')
    tag_django = Tag(name='django')
    article.tags.append(tag_flask)
    article.tags.append(tag_django)
    db.session.add(article)
    db.session.commit()
    return 'Hello World!'


if __name__ == '__main__':
    app.run()
```

#### RESTful中使用蓝图

如果使用的是蓝图，也可以使用RESTful，只是实例化Api时用的不再是app对象了，而是蓝图对象，参见示例代码：

**简单示例：**

```py
from flask import Blueprint
from flask_restful import Api, Resource, fields, marshal_with

from models import Article

# 创建蓝图对象，然后在app中注册蓝图即可：app.register_blueprint(article_bp)
article_bp = Blueprint('article', __name__, url_prefix='/article')
# 使用蓝图对象实例化Api，有多个蓝图对象就实例化多个Api即可。
api = Api(article_bp)


class ArticleView(Resource):
    pass

# 注意，由于蓝图中已经有url前缀了，这里的url不要添加多余的前缀了
api.add_resource(ArticleView, '/<article_id>/', endpoint='article')
```

#### RESTful返回HTML模板

虽然RESTful通常只是用来返回json数据，但是RESTful也可以返回其他类型的数据，比如HTML模板，参见示例代码：

* **装饰器：**使用api对象的一个装饰器\`@api.representation\('text/html'\)\`对一个函数进行装饰即可，当Resource子类的请求方法返回的是装饰器指定的数据类型时，就会去执行这个函数，并且返回这个函数的返回值给浏览器。比如这里指定的是text/html，所以当请求方法返回的是HTML模板时就执行这个函数。
* **参数：**被装饰函数的参数为data/code/headers，分别为对应的数据文件（比如HTML文件）、状态码和头部信息。
* **返回值：**被装饰函数的返回值只能是Response对象。

**简单示例：**

```py
# 这个装饰器表示Resource的子类中的请求方法如果返回的是指定的类型，这里是text/html类型，就会执行这个函数，并返回这个函数的返回值，并且这个返回值只能是Response对象
@api.representation('text/html')
def output_html(data, code, headers):
    # data表示text/html文件，code为状态码，headers为头部信息

    # 如果访问http://127.0.0.1:5000/article/list/就不会返回list.html的内容了，而是返回hello字符串了
    # return make_response('hello')

    # 如果访问http://127.0.0.1:5000/article/list/就会返回模板list.html中的内容了
    return make_response(data)


class ListView(Resource):
    def get(self):
        return render_template('list.html')

api.add_resource(ListView, '/list/', endpoint='list')
```



