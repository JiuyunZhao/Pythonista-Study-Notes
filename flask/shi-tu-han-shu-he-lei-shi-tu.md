# 视图函数和类视图

当一个url请求进入后台时，一般有两种方式来进行处理：视图函数和类视图。视图函数直接使用一个函数来进行处理并返回数据给浏览器，类视图则是使用类来进行处理并返回的，所以当需要进行的处理比较简单，则可以考虑使用前者，处理比较复杂就考虑使用后者，但是最终还是需要看使用环境和需求而定。

**视图函数：  
**

* **函数注册：**使用app.route装饰器对函数进行注册，或者使用add\_url\_rule进行注册
* **app.route\(rule, endpoint, methods\)：**

  * **url：**字符串类型，指定对应的url，参考[https://www.cnblogs.com/guyuyun/p/9142860.html](https://www.cnblogs.com/guyuyun/p/9142860.html)
  * **endpoint：**字符串类型，相当于给该url指定一个别名，一般用于url\_for进行url反转，可以不指定该参数，如果没有指定endpoint，则默认使用函数名，如果指定该参数，则必须使用该参数值，否则使用函数名会报错
  * **methods：**列表类型，列表中为该函数接收url的请求类型，比如GET、POST等，也可以不指定该参数，代表接收所有类型的请求

* **add\_url\_rule\(rule, endpoint, view\_func\)：**如果用于注册视图函数，则rule和endpoint的使用和app.route一样，view\_func则是对应的视图函数名称（注意此处函数不要加括号），与app.route的不同在于，app.route是装饰器，相当于自动进行了注册，而add\_url\_rule则是手动进行注册

* **函数返回值：**可以是HTML模板、字符串、response对象、response的子类或者固定格式的元组，参考[https://www.cnblogs.com/guyuyun/p/9743441.html](https://www.cnblogs.com/guyuyun/p/9743441.html)
* **自定义装饰器：**如果给视图函数使用了自定义的装饰器，那么这个（些）装饰器需要放在app.route装饰器之下的位置。

**标准类视图：  
**

* **定义：**类需要继承Flask.views.View，并且重写dispatch\_request方法
* **dispatch\_request：**当访问到对应的url时，就会实例化这个类，并执行dispatch\_request方法，并将这个方法的返回值返回给浏览器。dispatch\_request其实也是一个视图函数，所以它的返回值跟视图函数一样，必须是HTML模板，字符串，response对象，response对象的子类或者固定格式的元组。
* **类注册：**add\_url\_rule方法进行注册，注册方式与视图函数基本一致，不过参数view\_func跟视图函数的使用有点区别，需要使用视图类的as\_view方法进行转换，同时指定一个视图名称，如：app.add\_url\_rule\(‘/my\_url/’, ‘my\_endpoint\_name’, MyViewClass.as\_view\(‘my\_view\_name’\)\)
* **自定义装饰器：**如果需要给dispatch\_request使用自定义的装饰器，那么需要定义一个decorators的类变量，decorators可以是元组或者列表，里面存放的装饰器的顺序与视图函数的从上到下的顺序是一致的

**请求类视图：  
**

* **定义：**类需要继承Flask.views.MethodView，并且定义对应的请求方法
* **请求方法：**当对应的url访问进来后就会根据url的请求方式查找对应的请求方法，并将该方法的返回值返回给浏览器，如果没有定义对应的请求方法就会报错，提示“Method Not Allowed”
* **类注册：**注册方式和标准类视图是相同的
* **自定义装饰器：**与标准类视图是相同的



