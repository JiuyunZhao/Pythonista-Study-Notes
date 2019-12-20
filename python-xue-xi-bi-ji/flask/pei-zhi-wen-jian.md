# **配置文件**

如果设置项比较少的话可以使用“app.config\['param\_name'\]=value”的形式直接使用，如果需要设置的参数比较多的话，可以单独新建一个配置文件用来存放配置信息（配置文件中的参数需大写），且需要将配置文件中的内容导入到app.config中。

**导入配置文件的两种方式：**

* **from\_object：**例如在项目文件夹下新建一个“config”py文件，在代码中import config后使用app.config.from\_object\(config\)即可。
* **from\_pyfile\(filename, silent=False\)：**例如在项目文件夹下新建一个配置文件，可以不用是py文件，txt文件或其他文件也行，直接使用app.config.from\_pyfile\('cfg.txt'\)就行。filename必须是包含后缀名的文件名全称，silent默认为False，即该配置文件cfg.txt不存在时，会报错，设置silent=True时，则不会报错，直接跳过此行代码。

**配置文件中参数设置及其含义如下：**

* **DEBUG：**设置为“True”时表示开启debug模式，设置为“False”表示关闭debug模式，只针对py文件。
* **TEMPLATES\_AUTO\_RELOAD：**设置为True时，HTML模板有修改时，ctrl+s后就可自动加载HTML文件，不用重新启动服务器。类似DEBUG，但只针对HTML模板文件。
* **SQLALCHEMY\_DATABASE\_URI：**设置数据库连接字符串（这里使用的是sqlalchemy插件），字符串形式是固定的，为“dialect+driver://username:password@host:port/database”，比如MySQL的连接字符串可以如图配置（其中的花括号是Python的一种字符串格式化，就像“%”使用一样），其中driver为Python2的mysqldb，Python3的pymysql，具体以安装的插件为准：

```py
# 数据库连接固定格式格式字符串
# dialect+driver://username:password@host:port/database
DIALECT = 'mysql'
DRIVER = 'mysqldb'
USERNAME = 'root'
PASSWORD = 123456
HOST = '127.0.0.1'
PORT = '3306'
DATABASE = 'db_demo1'

SQLALCHEMY_DATABASE_URI = '{dialect}+{driver}://{username}:{password}@{host}:{port}/{database}?charset=utf8'.format(
    dialect=DIALECT, driver=DRIVER, username=USERNAME, password=PASSWORD, host=HOST, port=PORT, database=DATABASE
)
```

* **SECRET\_KEY：**包含24个字符的字符串，用来对session中的数据进行加密操作的，一般采用随机字符串，可以使用os.urandom\(24\)来生成这个字符串，注意如果重启服务器后这个字符串改变了的话，之前设置的session就获取不到了。
* **PERMANENT\_SESSION\_LIFETIME：**设置session的过期时间，datetime.timedelta类型，如果没有设置，则默认为31天，比如设置过期时间为7天：PERMANENT\_SESSION\_LIFETIME=datetime.timedelta\(days=7\)。
* **SERVER\_NAME：**设置域名和端口号，如SERVER\_NAME='xxx.com:5000'。



