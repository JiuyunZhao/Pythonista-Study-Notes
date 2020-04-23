# 数据库迁移

数据库迁移是将代码中模型类（即表）的修改同步到数据库中， sqlalchemy的模型类一旦使用create\_all\(\)映射到数据库中后，对这个模型类的修改（例如添加了一个新的字段）就不会再映射到数据库中了，这时候想要在数据库中得到新的表就需要删掉重新映射一次，可是这样做的话原先表中的数据也没了，这肯定是不行的，数据库中的数据怎么能随便删呢，而数据库迁移操作就完美解决了这个问题。

就像ORM操作有sqlalchemy和flask-sqlalchemy一样，数据库迁移也有alembic和flask-migrate，alembic是所有Python程序都可以用的三方库，而flask-migrate则是flask专用的，它们的用法也都是相似的。

flask中如果再配合另一个三方库flask-script就能更加方便的管理数据库了，flask-script能让Python中的函数能以命令的形式在DOS窗口中执行，让我们更加方便地管理项目，当然也包括数据库操作。

## 一、flask-script使用

**安装：**pip install flask-script

**主文件：**一般用于启动flask-script的Manager，可以通过主文件调用所有的函数（必须是被**\[Manager\_obj\].command**装饰的函数才能被调用，而且一般不会所有的函数都定义在主文件中），比如新建并写好一个简单的py文件“manage.py”，然后就可以在DOS中使用命令“**python manage.py runserver**”调用了：

```py
from flask_script import Manager
from flask_script_migrate_demo import app

# 实例化Manager，app为Flask实例对象，可以从其他文件导入
manage = Manager(app)


# 使用此装饰器装饰的函数才能在DOS中调用
@manage.command
def runserver():
    # ... #
    print('The server has been started!')


if __name__ == '__main__':
    manage.run()  # 启动Manager
```

![](/assets/flask-7.png)

**命令传参：**使用@\[Manager\_obj\].option\(\)定义命令的参数，有多少个参数就需要多少个装饰器，且他们之间的顺序不能乱。

```py
# 装饰器@manager.option：有多少个参数就使用多少个manager.option装饰器，且顺序要与函数一致（从上到下）
# 命令简写：-u是命令--username的简写，其实真正的命令是--username，就是说这两个命令都可以使用
# 命令全称：dest是用于映射函数的参数名称
# 使用示例：DOS中可以使用如下命令：python manage.py add_user -u Jason -a 18
@manager.option('-u', '--username', dest='username')
@manager.option('-a', '--age', dest='age')
def add_user(username, age):
    print("用户名：%s，年龄：%s" % (username, age))
```

**子文件（子命令）：**当flask\_script管理的命令太多的时候，就需要归类放到不同的子文件中，且子文件一般用于放置操作函数，然后通过主文件来调用，需要注意的是子文件中的函数同样需要装饰器“\[Manager\_obj\].command”，不过子文件中的Manager实例化不需要app（Flask对象）了，同时在主文件中需要使用“**\[Manager\_obj\].add\_command\('\[sub\_manager\_obj\_name\]', \[sub\_manager\_obj\]\)**”将子文件中的Manager添加到主文件中，“sub\_manager\_obj\_name”为子文件中Manager对象的别名（DOS调用的就是它），如下代码在DOS中使用命令“**python manage.py db\_manage db\_init**”调用即可：

**主文件manage.py和子文件db\_scripts.py：**

```py
from flask_script import Manager
from flask_script_migrate_demo import app
from db_scripts import DBManage

manage = Manager(app)
# 将子文件db_scripts的Manager对象DBManage导入，别名db_manage将用于DOS调用
manage.add_command('db_manage', DBManage)


if __name__ == '__main__':
    manage.run()  # 启动Manager
```

```py
from flask_script import Manager

# 需要APP（Flask对象）来实例化
DBManage = Manager()


# 同样需要command装饰器
@DBManage.command
def db_init():
    # ... #
    print('The database has been initialized successfully!')
```

![](/assets/flask-8.png)

# 二、alembic使用

**安装：**pip install alembic

**迁移过程：**定义模型类 -&gt; 生成迁移脚本 -&gt; 映射更新到数据库

**注意：**使用alembic时需要先CD到项目根目录下，然后再使用alembic

**准备操作： **

* **alembic init \[alembic\_name\]：**在DOS中（项目根目录）执行此命令，初始化创建一个alembic迁移仓库，一般名称可以就叫alembic，以便识别
* **sqlalchemy.url：**在alembic.ini文件中修改sqlalchemy.url的值为数据库连接字符串即可，例如sqlalchemy.url = mysql+pymysql://root:123456@localhost/alembic\_demo?charset=utf8，如果是本机的话HOST的值就写localhost就行，其中alembic\_demo为数据库名称
* **target\_metadata：**在创建好的\[alembic\_name\]文件夹下的env.py中给target\_metadata赋值为对应的metadata，例如：target\_metadata = alembic\_demo.Base.metadata。

```py
from sqlalchemy import Column, String, Integer
from sqlalchemy.ext.declarative import declarative_base

USERNAME = 'root'
PASSWORD = '123456'
HOST = '127.0.0.1'  # 本机
PORT = '3306'  # MySQL默认端口
DATABASE = 'alembic_demo'  # 数据库名

# 数据库连接字符串
DB_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}?charset=utf8'.format(
    username=USERNAME, password=PASSWORD, host=HOST, port=PORT, database=DATABASE
)

# 模型类基类
Base = declarative_base(DB_URI)


class User(Base):
    """简单模型类"""
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)
    age = Column(Integer, nullable=False, default=18)
```

**生成迁移脚本：**alembic revision --autogenerate -m "\[message\]"。其中message是自己定义的此次提交/更新的信息，方便以后查看和问题跟踪。并且执行成功后会在\[alembic\_name\]/versions下生成一个对应版本的脚本迁移文件

**映射更新数据库：**alembic upgrade head。head表示最新迁移脚本的版本号，也可以不用head，而使用指定脚本的版本号，迁移脚本中revision的值即为该脚本对应的版本号。同时第一次upgrade时会在数据库中创建一张名为alembic\_version的表，此表只有一列一行数据，即当前数据库的版本号。

**alembic常用命令和参数：**

* **init:** 创建一个alembic仓库
* **revision:** 创建一个新的版本号
* **--autogenerate: **自动将当前的模型的修改生成迁移脚本
* **-m: **定义本次迁移的描述
* **upgrade: **将指定版本号的迁移脚本映射到数据库中（更新），即执行迁移脚本中的upgrade函数
* **downgrade: **将指定版本的迁移脚本映射到数据库中（回退），即执行迁移脚本中的downgrade函数
* **head:** 代表的最新迁移脚本的版本号
* **heads: **查看最新的版本号（head）
* **history: **查看所有的迁移信息（版本号变化及每次迁移时-m参数定义的消息）
* **current: **查看当前的版本号（数据库）
* **注：**映射数据库时（更新或回退）会执行两个版本间的所有迁移脚本

**常见错误和解决方案：**

**错误1：** Target database is not up to date

**原因：**数据库的版本（current）和最新迁移脚本（head）的版本不一致

**解决方案：**使用alembic upgrade head将数据库更新为最新的版本即可



**错误2：** Can't locate revision identified by 'xxxxx'

**原因：**数据库中的版本号没有在迁移脚本中找到对应的版本（或许对应的迁移脚本文件被删了）

**解决方案：**可以先删除alembic\_version中的版本数据，再upgrade（这时候可能需要将所有的迁移脚本中的upgrade函数pass掉），这里的解决方案不确定，知道原因后灵活操作就行了。

# 三、flask-migrate使用

**安装：**pip install flask-migrate

**作用：**一般数据库模型类都是使用专门的py来定义存放的，比如“models.py”，当需求变化时，可能这些Model都已经映射到数据库中了，再想要修改时（比如新增加一列），如果没有其他插件，就只能删除数据库中的表，再重新生成了，显然这是不实用的，所以flask-migrate就显得非常有用了，它可以将代码中对数据库的修改同步更新（迁移操作）到数据库中。

**依赖插件：**flask-migrate插件是依赖于flask-script的，所以要使用flask-migrate，就需要先装好flask-script。

**基本使用（db为自己定义的flask-script子命令，见示例代码）：**

* **初始化：**Python manage.py db init
* **生成迁移脚本：**Python manage.py db migrate
* **映射到数据库中：**Python manage.py db upgrade
* **查看命令：**查看所有的migrate命令Python manage.py db --help
* **使用示例：**第一次执行manage.py的时候，需要依次使用命令“python manage.py db init”（初始化迁移环境）、“python manage.py db migrate”（生成对应的迁移文件）和“python manage.py db upgrade”（将迁移文件同步更新到数据库中）来完成数据库的映射，当之后你的model类中有改变时，只需要依次运行后面两个命令“migrate”和“upgrade”即可对数据库进行同步更新。其中“manage.py”和“db”是自定义的名称，见如下代码：

```py
from flask_script import Manager
from flask_script_migrate_demo import app
from flask_migrate import Migrate, MigrateCommand

from extends import db  # db为SQLAlchemy对象
from models import User  # User为db.Model的子类，即模型类，其他未导入的模型类就不能映射到数据库了

manager = Manager(app)
# 绑定app和db到flask_migrate中
Migrate(app, db)
# 为manager取一个别名，并作为子命令使用
manager.add_command('db', MigrateCommand)


if __name__ == '__main__':
    manager.run()  # 启动Manager
```



