# 蓝图Blueprint

蓝图这个名字好像就是根据单词Blueprint字面意思来，跟平常我们理解的蓝图完全挂不上钩，这里蓝图就是指Blueprint。

使用蓝图的好处是可以将不同功能作用的视图函数/类视图放到不同的模块中，可以更加方便的开发和维护。

1. **导入Blueprint：**from flask import Blueprint
2. **创建一个蓝图：**例如user\_bp = Blueprint\(‘user’, \_\_name\_\_, prefix=’/user’\)，第一参数指定蓝图名称，第二个参数与flask中的使用是相同的（用于指定静态文件的相对路径，也方便其他三方插件报错时定位问题），第三个参数prefix为这个蓝图指定url前缀，这个前缀会和视图函数/类视图指定的url直接连接起来形成一个有效的url
3. **视图函数：**也是和Flask的使用一样，使用对应的route装饰器即可
4. **注册蓝图：**使用方法app.register\_blueprint\(user\_bp\)即可
5. **HTML模板查找规则：**如果创建蓝图时，第二个参数使用的是\_\_name\_\_，那么默认的模板文件路径就是项目根目录下的templates文件夹（Flask实例化时的\_\_name\_\_），如果不想使用这个templates文件夹，可以在实例化Blueprint时指定template\_folder参数，那么此时模板文件的查找顺序就是先在templates文件夹中查找，查找不到时，就会在蓝图文件同级目下template\_folder参数指定的文件夹（Blueprint实例化时的\_\_name\_\_）中查找
6. **静态文件查找规则：**如果创建蓝图时，如果第二个参数使用的是\_\_name\_\_，那么，在使用url\_for\(‘static’, filename=xxx\)时，就只会在项目根目录的static文件夹中查找，如果使用url\_for\(‘user.static’, filename=xxx\)就会在蓝图创建时static\_folder参数指定的文件夹中查找
7. **url\_for反转：**反转获取蓝图中的url时，必须加上蓝图名称的前缀，即便是就在该蓝图模块中使用url\_for，也要加上蓝图的名称，例如url\_for\(‘blue\_name.viewfunc\_name’\)
8. **子域名：**在创建蓝图的时候可以使用subdomain参数指定子域名，需要注意的是具体的IP地址和localhost是不能有子域名的



