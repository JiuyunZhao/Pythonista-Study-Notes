# 上下文

#### 线程隔离Thread Local：

如果一个对象具有线程隔离的特性，就可以称之为“Thread Local”，线程隔离是指该对象在不同的线程中都是独立的，在一个线程中对该对象的操作不会影响另一个线程对该对象操作，比如在线程A中修改了该对象的某个属性值，但是在线程B中该对象的这个属性值并没有被修改。



#### Flask线程隔离对象：

在Flask中，线程隔离对象包括request、session、g、current\_app等，在项目中这些对象都可以直接导入使用，而不用担心运行时不同的地方同时使用这些对象时会出问题。比如有多个不同的请求同时进来，它们都会使用到request对象，但因为request是线程隔离的，而每一个请求都有自己的线程，所以request对象使用各自线程的数据。



#### 应用上下文和请求上下文：

应用上下文和请求上下文都有各自的上下文context对象和LocalStack栈对象，在视图函数中使用上下文操作时（current\_app对象、url\_for函数等），会自动创建对应的上下文context对象并将其push到对应的LocalStack栈的顶端，进行上下文操作时，会自动到这个LocalStack栈的顶端获取context对象。所以在视图函数中是可以直接使用上下文操作的。



如果想要在视图函数外面使用上下文操作，就需要手动创建对应的context对象并将其push到对应的LocalStack栈中。简单示例代码如下：

**手动创建应用上下文：**

```py
from flask import Flask, current_app

app = Flask(__name__)

# 第一种方式：在文件中直接创建context对象，并push到LocalStack栈中
app_context = app.app_context()
app_context.push()
# 获取当前app信息的current_app对象属于操作应用上下文的对象
print(current_app)


# 第二种方式：在with块中使用应用上下文，这种方式只有在with块中才能进行应用上下文的操作，在with外面就不行了
# 这种方式创建context对象后会自动push到LocalStack栈中
with app.app_context():
    print(current_app)
```

**手动创建请求上下文：**

```py
from flask import Flask, url_for

app = Flask(__name__)


@app.route('/detail/')
def detail():
    return 'detail information!'


# 会先检查应用LocalStack栈顶端有没有应用上下文context对象，如果没有，就会自动创建应用上下文context对象并将其push到应用LocalStack栈顶端
# 然后才创建请求context对象，并将其push到请求LocalStack栈的顶端
# 同样，这种方法只能在with块中才能使用请求上下文操作，在with外面就不行了
with app.test_request_context():
    # url_for反转获取视图的url属于请求上下文的操作
    print(url_for('detail'))
```

#### g对象：

使用“from flask import g”即可使用g对象（可以看做是global的缩写），g对象是一个用来保存全局数据的对象，同时g对象也是线程隔离的。通常我们在整个Flask程序中需要经常使用的一些自定义数据，就可以放在g对象中，需要用的时候在g对象中获取即可。如在一个文件中定义好\`g.username = 'xiaoming'\`，然后在另一个文件中直接就可以使用\`g.username\`即可。



