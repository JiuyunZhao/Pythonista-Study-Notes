# 数据库ORM操作

Python中使用sqlalchemy插件可以实现ORM（Object Relationship Mapping，模型关系映射）框架，而Flask中的flask-sqlalchemy其实就是在sqlalchemy外进行了一层封装，使得在flask中使用起来更加方便，当然sqlalchemy的原有的方法也都可以使用。也就是说sqlalchemy在普通的Python程序中也可以使用，而flask-sqlalchemy是为flask“定制”的。

我这里使用的是MySQL数据库，Python3中对应的驱动插件为pymysql（Python2中对应的驱动插件为mysql-python），所以这里的数据库操作需要安装MySQL、pymysql（或mysql-python）和flask-sqlalchemy。

## 一、MySQL数据库安装（其实MySQL的东西网上一大堆）

**安装版：**exe安装版按照步骤一步步来就行了，可以只安装MySQL服务即可，如果电脑上没有对应版本VS C++也会提示你安装的。

**免安装版/解压版：**根据以下步骤进行即可：

* **下载：**将下载的zip包解压到任意目录。
* **配置环境变量：**将解压后的文件夹中的bin目录路径放入环境变量的path中。
* **配置文件：**在解压的文件夹下（也就是bin的上一级目录）新建一个ini配置文件如“my-default.ini”（如果有就需要修改其中的内容），在配置文件中写入（修改或追加）如下内容：

```
[mysqld]
basedir=D:\MySQL Server 8.0
datadir=D:\MySQL Server 8.0\data
```

**注：**解压后可以自己更改文件夹名称，比如我这儿是“MySQL Server 8.0”，配置这两个信息后，其中的“data”文件夹不要自己创建（默认是没有这个文件夹的，但是会在下一步自动创建），否则MySQL服务会启动失败。

* **安装MySQL服务：**在DOS窗口cd到bin目录，依次执行命令“mysqld -install”和“mysqld --initialize-insecure”（第二个命令会自动创建data文件夹，里面包含数据库的一些基本信息，并且该命令执行成功后是不会返回任何信息的）。
* **启动（停止）MySQL服务：**执行命令“net start mysql”启动MySQL服务（停止服务是“net stop mysql”）。
* **设置密码：**重开DOS窗口（保证是管理者权限），执行命令“mysqladmin -u root -p password”，回车，显示“Enter password”，不用输入任何东西直接回车（未设置密码时默认没有密码），显示“New password”，输入自己的密码即可，回车，显示“Confirm new password”，再次输入密码即可，回车，设置密码成功。
* **登录：**输入命令“mysql -u root -p”，回车，然后输入设置的密码即可。

**安装和使用过程中遇到的问题和解决方案：**

* 使用命令“mysql -u root -p”输入密码后提示“access denied for user root @localhost\(using password:YES\)”，进不去数据库：删除全部文件夹重新安装“MySQL安装（解压版）”，重新设置一遍密码就OK了（关键是网上找了许多办法也不管用，只能这招了）。
* 运行Flask连接数据库时提示“Client does not support authentication protocol requested by server; consider upgrading MySQL client”：进入数据库后依次执行命名“USE mysql;”、“ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql\_native\_password BY '123456';”和“FLUSH PRIVILEGES;”，其中除了“'123456'”是个人设置的密码外，其他的字母和字符都copy就行了（第二条命令执行失败的话，重新进入数据库或者重新安装数据库再试一下吧，我第一次也没成功，后来重设密码后又成功了）

## 二、数据库驱动插件安装（pymysql/mysql-python）

**pymysql安装：**Python3连接数据库的驱动插件为pymysql，直接用pip3 install pymysql即可。

**mysql-python安装：**Python2连接MySQL数据库的驱动为MySQL-python，也可使用pip install MySQL-python，这里如果安装失败，可直接下载安装包进行安装：https://pypi.org/project/MySQL-python/\#files。

## 三、SQLAlchemy使用

**安装：**pip install sqlalchemy

**连接数据库：**设置数据库连接字符串，并使用create\_engine函数创建一个engine对象，并使用engine对象的connect方法连接数据库，注意连接数据库后需要使用close方法关闭连接。

```py
from sqlalchemy import create_engine

# 数据库连接信息
HOSTNAME = '127.0.0.1'  # 本机地址
PORT = '3306'  # MySQL默认端口
DATABASE = 'sqlalchemy_demo'  # 数据库名
USERNAME = 'root'  # 安装数据库时设置的用户名和密码
PASSWORD = '123456'

# 数据库连接字符串
# dialect+driver://username:password@host:port/database
DB_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}'.format(
    username=USERNAME, password=PASSWORD, host=HOSTNAME, port=PORT, database=DATABASE
)

# 创建一个engine对象
engine = create_engine(DB_URI)
# 连接数据库
conn = engine.connect()
# 执行SQL语句，并返回执行结果
sel_res = conn.execute('select * from User')
# 获取并打印一条查询结果
print(sel_res.fetchone())
# 关闭连接
conn.close()
```

**定义模型类：**使用declarative\_base函数生成一个模型类的基类，以便模型类映射到数据库中，且必须使用\_\_tablename\_\_指定表名（一个类对应数据库中一张表，不同的类实例对应表中不同的记录）。

```py
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

# 数据库连接信息
HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'sqlalchemy_demo'
USERNAME = 'root'
PASSWORD = '123456'

# 数据库连接字符串
# dialect+driver://username:password@host:port/database
DB_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}'.format(
    username=USERNAME, password=PASSWORD, host=HOSTNAME, port=PORT, database=DATABASE
)

# 创建一个engine对象
engine = create_engine(DB_URI)
# 创建所有模型类的基类
Base = declarative_base(engine)

# 注意以下定义全是在类中直接进行的，没有在__init__等方法中进行
class Person(Base):

    # 使用__table__name指定表名
    __tablename__ = 'person'

    # 定义属性字段id为整型、主键、自动增长
    id = Column(Integer, primary_key=True, autoincrement=True)
    # 定义属性字段name为最长50个字符的字符串类型，且非空
    name = Column(String(50), nullable=False)
    # 定义属性字段age为整型
    age = Column(Integer)

# 删除表，不用特地指定表，在加载Python文件时会自动检查定义了哪些表，并删除这些表
Base.metadata.drop_all()

# 将所有模型映射到数据库中，并且相同的模型类只在第一次执行时会生效，即如果第一次执行后，
# 又对类进行了修改，重新运行程序执行metadata.create_all()后并不会把更新的内容映射到数据库中，前提是数据库中的表没有被删除
Base.metadata.create_all()
```

**常用数据类型：**数据类型对应的类直接从模块sqlalchemy中导入即可。

* **Integer: **整型。
* **Float\(32位\)/Double\(64位\): **浮点类型，可能存在精度丢失的问题，这时候可以使用DECIMAL定点类型。
* **Boolean: **True/False，对应数据库里的1/0。
* **DECIMAL: **定点类型，需要指定数字的总位数（小数点前的位数+小数点后的位数）和小数位数。
* **Enum: **枚举类型，需要指定所有的枚举值，且给这类型的字段赋值的时候就只能使用这几个指定的枚举值。
* **Date:** 日期类型（年月日），即datetime.date\(\)。
* **DateTime: **日期时间类型（年月日 时分秒），即datetime.datetime\(\)。
* **Time: **时间类型（时分秒），即datetime.time\(\)。
* **String: **字符串类型，需要指定最大字符个数。
* **Text: **文本类型。

**Column常用参数：**Column的第一个参数name用来指定数据中的属性名，如果不指定则默认为该属性变量名为属性名。

* **name：**数据库中的属性名，默认使用模型对应属性的变量名。
* **type\_：**属性的数据类型（加下划线是因为Python已经有了关键字type）。
* **default: **默认值，即没有给该属性赋值时采用默认值。
* **nullable: **是否可为空，默认为True。
* **primary\_key: **是否为主键，默认False。
* **unique: **是否唯一，默认为False。
* **autoincrement:** 是否自动增长，默认False。
* **onupdate: **在某条记录更新的时候执行的函数（无论更新的内容是不是该属性字段都会执行）。

**增删改查操作：**使用sessionmaker创建session，并在session中进行增删改查操作，注意如果涉及到修改数据库，一定要commit后才能在数据库生效。

* **增加数据：**可以使用session的add\(obj\)方法（单条数据）和add\_all\(\[obj1, obj2, ...\]\)（多条数据），最后使用commit方法提交修改。
* **删除数据：**可以使用session的delete\(obj\)删除数据，最后使用commit方法提交修改。
* **修改数据：**直接对查询出的实例属性值进行重新赋值即可。
* **查询数据：**可以使用session的query方法得到query对象，再对query对象进行filter等过滤即可得出想要的查询结果，详细使用见示例代码。

**增删改查简单示例（数据库连接代码和模型类代码见前面的代码示例）：**

```py
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

engine = create_engine(DB_URI)

# 别忘了实例化后再一次调用（加括号），会调用类的__call__生成session
session = sessionmaker(engine)()


# 增加数据
def add_data():
    p = Person(name='Jason', age=19)
    # 添加一条数据到内存中
    session.add(p)
    # 添加多条数据到内存中
    p1 = Person(name='Mickle', age=20)
    p2 = Person(name='Marya', age=21)
    session.add_all([p1, p2])
    # 提交到数据库中
    session.commit()


# 删除数据
def del_data():
    p = session.query(Person).first()
    # 直接删除提交即可
    session.delete(p)
    session.commit()


# 修改数据
def update_data():
    p = session.query(Person).first()
    # 获取对象后，直接修改提交即可
    p.name = 'JiuyunnZhao'
    session.commit()


# 查询数据
def search_data():
    # 查找模型类对应表的的所有数据（可用for遍历）
    # persons = session.query(Person).all()
    # 查询满足某个条件的所有数据（可for遍历），查询条件只有参数和一个等号（这种方式比较简单，因为只针对一个类）
    # persons = session.query(Person).filter_by(name='Mickle').all()
    # 查询满足某个条件的所有数据（可for遍历），查询条件类名.参数和双等号（filter中的条件可以更加复杂，这里只是简单举例）
    # persons = session.query(Person).filter(Person.name == 'Mickle').all()
    # for p in persons:
    #     print(p)
    # 根据主键查询某一条数据，没有对应的主键则返回None
    # p = session.query(Person).get(100)
    # 获取某张表的第一条数据
    p = session.query(Person).first()
    p = session.query(Person).filter(Person.name == 'Mickle').first()
    p = session.query(Person).filter_by(name='Mickle').first()
    print(p)


# 测试代码
if __name__ == '__main__':
    add_data()
    # del_data()
    # update_data()
    # search_data()
```

**聚合函数：**从sqlalchemy中导入func，func中有许多聚合函数可以使用。其实func中没有“真正”的聚合函数，都是MySQL中的函数，也就是或MySQL中的聚合函数都可以在func中找到和使用。

* **func.count\(col\_name\)：**统计某个字段的行数。
* **func.avg\(col\_name\)：**统计某个字段的平均值。
* **func.max: **统计某个字段的最大值。
* **func.min: **统计某个字段的最小值。
* **func.sum: **统计某个字段的和。
* **示例代码：**session.query\(func.avg\(Person.age\)\).first\(\)，结果是元组，其实也只有一条数据。

**filter中的条件过滤（注意不是filter\_by方法）：**

* **==\(equal\)：**如session.query\(Person\).filter\(Person.name == 'Mickle'\).first\(\)。
* **!=\(not equal\)：**如session.query\(Person\).filter\(Person.name != 'Mickle'\).all\(\)。
* **like/ilike：**后者不区分大小写，且两个方法查询条件中的通配符与SQL一致，如session.query\(Person\).filter\(Person.name.like\('Mi%'\)\).all\(\)。
* **in\_\(in\)：**之所以有下划线，Python本身有in的关键字了，如session.query\(Person\).filter\(Person.name.in\_\(\['Mickle', 'Marya'\]\)\).all\(\)。
* **notin\_\(not in\)：**如session.query\(Person\).filter\(Person.name.notin\_\(\['Mickle', 'Marya'\]\)\).all\(\)。
* **~\(取反\)：**如session.query\(Person\).filter\(~Person.name.in\_\(\['Mickle', 'Marya'\]\)\).all\(\)，这里的效果就相当于not in了。
* **==None/!=None\(is null/is not null\)：**如session.query\(Person\).filter\(Person.name == None\).all\(\)。
* **and\_\(and\)：**这个方法不常用，使用方法：导入“from sqlalchemy import and\_”，如session.query\(Person\).filter\(and\_\(Person.name.like\('M%'\), Person.age == 18\)\).all\(\)。常用的是session.query\(Person\).filter\(Person.name.like\('M%'\), Person.age == 18\).all\(\)，多个and条件使用逗号分隔传参即可。
* **or\_\(or\)：**导入“from sqlalchemy import or\_”，如session.query\(Person\).filter\(or\_\(Person.name.like\('Mi%'\), Person.name.like\('Ma%'\)\)\).all\(\)，多个or条件使用逗号分隔传参即可。

**外键及外键约束：**在想要定义为外键的列中传入参数ForeignKey\('tablename.column\_name'\)即可，如果想要定义外键约束，在ForeignKey中指定参数ondelete为相应约束即可，默认约束为RESTRICT。

* **RESTRICT: **当要删除的父表中的数据在子表中有外键引用时，就会阻止删除操作，即不可删除。
* **NO ACTION: **在MySQL中，效果同RESTRICT。
* **CASCADE: **级联删除，即删除父表中的数据时，会同时删除与之有外键关联的子表的数据。
* **SET NULL: **删除父表数据时，子表相应的外键列的值就会设置为NULL（此时，外键列的nullable属性就不可设置为False了，因为这自相矛盾，会报错的）。
* **注：**在ORM中删除数据，即使用session.delete\(xxx\)时会无视MySQL数据库中的外键约束，可以直接删除数据库中的数据。当删除一条数据时，如果有从表引用了它，即从表中有它的外键时，按照MySQL中RESTRICT约束，会删除失败，但是直接在ORM的代码中使用session.delete\(xxx\)时，就会删除成功，且删除的步骤为，先将该数据对应的从表中的外键设置为NULL，再删除该条数据。所以为了避免这种与数据库不一致的行为（略带危险性），需要给外键加上一个nullable=False的约束，这样不能设置为NULL后，就不会去进行后面的删除步骤了。

```py
from sqlalchemy import create_engine, Column, Integer, String, Text, ForeignKey
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'sqlalchemy_demo'
USERNAME = 'root'
PASSWORD = '123456'

# dialect+driver://username:password@host:port/database
DB_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}'.format(
    username=USERNAME, password=PASSWORD, host=HOSTNAME, port=PORT, database=DATABASE
)


engine = create_engine(DB_URI)
Base = declarative_base(engine)


# 父表
class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)

# 子表
class Article(Base):
    __tablename__ = 'article'
    atc_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    content = Column(Text, nullable=False)

    # 外键列，意为一个用户可以有多篇文章，注意此列的类型应该对应父表的主键类型一致
    # 注意这里是“表名.属性名”，而不是“类名.属性变量名”
    user_id = Column(Integer, ForeignKey('user.user_id', ondelete='CASCADE'))
```

**外键关系：**通过relationship将两个存在外键关联的两张表建立一个关系属性，并通过这个属性去进行互相访问。

* **一对一：**在一张表中定义一个外键，另一张表中定义一个到此表的关系，并指明relationship的参数uselist为False，表示此表与另一张表是一对一的关系（默认是True，即一对多关系）；也可使用backref函数进行反向引用，使得外键的定义和关系的定义都可以在同一个模型类中。
* **一对多：**通过外键进行自动关联，且外键所在一方为“多”，另一方为“一”；定义关系时可以使用backref参数，使得外键的定义和关系的定义都可以在同一个模型类中；在“一”中定义的表示“多”的关系变量其实继承自list，所以append等方法也适用于它。
* **多对多：**需要用到Table类生成中间表和relationship的参数secondary指明这个中间表。
* **relationship参数：**
  * **第一个参数：**用于关联另一张表时用的是类名，而不是表名。
  * **backref：**用于进行反向引用，即可以在backref中定义另一张表对此表的引用。
  * **secondary：**用于多对多关系建立时指定中间表。
  * **uselist：**指定此表与另一张表是一对一的关系。
  * **cascade：**指定关系的约束，可以指定多个约束，每个值用逗号连接即可，且只会影响到定义cascade时所在的类，cascade参数值选项及含义如下：
    * **save-update：**默认选项，执行commit操作提交一条数据时，会将该数据通过relationship关联的其他数据一起更新到数据库中。
    * **merge：**也是默认选项，等效于session.merge\(xxx\)，当添加的数据和已有的数据冲突时，会替换掉已有的数据。
    * **delete：**当删除一条数据时，会同时删除与其通过relationship关联的其他数据。
    * **delete-orphan：**配合delete使用，只能使用在一对多的“一”中，表示当主表的数据被删除时，该数据的从表中的数据也会被删除。这种效果也可以在从表的relationship中指定参数single\_parent=True来达到，这种情况下，就算是多对多的关系，只要删除了主表数据，从表的对应数据也会被删除，只是反过来就不行了， 但delete-orphan是可以的，所以它使用的前提条件是一对多的“一”。
    * **expunge：**移除数据，等效于session.expunge\(xxx\)，与session.add\(xxx\)作用刚好相反。
    * **all：**等效于cascade='save-update,merge,refresh-expire,expunge,delete'\(注意没有delete-orphan\)。
  * **lazy：**值为dynamic时即可实现懒加载，即查询数据或append追加数据的时候并不会马上进行数据库操作，而是返回一个AppenderQuery查询对象，在最后才进入数据库进行操作。

**一对一关系简单示例：**

```py
# 一对一关系示例1：一张表定义外键，另一张表定义一对一关系，即uselist=False
class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)
    # 添加一个到模型Article的一对一关系，关键在于uselist=False，因为默认是True，那就是一对多了
    article = relationship('Article', uselist=False)


class Article(Base):
    __tablename__ = 'article'
    atc_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    content = Column(Text, nullable=False)

    # 外键列“表名.属性名”，而不是“类名.属性变量名”
    user_id = Column(Integer, ForeignKey('user.user_id'))

    # 添加一个到模型User的关系，由于User类中定义了一对一uselist=False，所以这个关系自然也是一对一了，也不用再指定uselist=False
    author = relationship('User')


# 一对一示例2：利用反向引用函数backref建立一对一关系
# backref函数中的参数和relationship中的参数是一致的，relationship中可用的参数它都可以用
from sqlalchemy.orm import sessionmaker, relationship, backref


class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)


class Article(Base):
    __tablename__ = 'article'
    atc_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    content = Column(Text, nullable=False)

    # 外键列
    user_id = Column(Integer, ForeignKey('user.user_id'))
    # 这里backref的使用相当于在User类中建立了一个名为article的关系，且参数uselist为False
    author = relationship('User', backref=backref('article', uselist=False))
```

**一对多关系简单示例：**

```py
# 一对多关系示例1
class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)
    # 添加一个到模型Article的关系，一对多，表示“多”的的变量articles其实继承自list，所以append等方法也适用这个属性
    articles = relationship('Article')


class Article(Base):
    __tablename__ = 'article'
    atc_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    content = Column(Text, nullable=False)

    # 外键列，注意这里是“表名.属性名”，而不是“类名.属性变量名”
    user_id = Column(Integer, ForeignKey('user.user_id'))

    # 添加一个到模型User的关系，多对一
    author = relationship('User')


# 一对多关系示例2
class User(Base):
    __tablename__ = 'user'
    user_id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)


class Article(Base):
    __tablename__ = 'article'
    atc_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    content = Column(Text, nullable=False)

    # 外键列，注意这里是“表名.属性名”，而不是“类名.属性变量名”
    user_id = Column(Integer, ForeignKey('user.user_id'))

    # 反向引用，backref参数就相当于自动给User模型添加了一个名为articles且关联到User模型的的关系
    author = relationship('User', backref='articles')

# 使用示例，以下代码将在article表中添加两条数据
user =User(username='Mickle')
article1 = Article(title='aaa', content='1111')
article2 = Article(title='bbb', content='222')
user.articles.append(article1)
user.articles.append(article2)
session.add(user)
session.commit()
```

**多对多关系简单示例：**

```py
# 多对多关系示例：一篇文章可以有多个标签，一个标签也可以用于多篇文章
from sqlalchemy import create_engine, Column, Integer, String, Text, ForeignKey, Table
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker, relationship


HOSTNAME = '127.0.0.1'
PORT = '3306'
DATABASE = 'sqlalchemy_demo'
USERNAME = 'root'
PASSWORD = '123456'

# dialect+driver://username:password@host:port/database
DB_URI = 'mysql+pymysql://{username}:{password}@{host}:{port}/{database}'.format(
    username=USERNAME, password=PASSWORD, host=HOSTNAME, port=PORT, database=DATABASE
)

engine = create_engine(DB_URI)
Base = declarative_base(engine)
session = sessionmaker(engine)()


class Article(Base):
    __tablename__ = 'article'
    artcle_id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)

    # 与Tag建立关联关系和反向引用关系，并指明中间表为article_tag
    # 这个关系可以定义在关联的两张表中的任意一张表中都可以
    tags = relationship('Tag', backref='articles', secondary=article_tag)


class Tag(Base):
    __tablename__ = 'tag'
    tag_id = Column(Integer, primary_key=True, autoincrement=True)
    name = Column(String(50), nullable=False)


# 通过类Table来生成一个中间表
article_tag = Table(
    'article_tag',  # 表名
    Base.metadata,  # 继承的模型类型
    # 定义中间表中的列，声明此列关联到表article的列article_id，并指明以此列来作为复合主键的其中一列
    Column('article_id', Integer, ForeignKey('article.article_id'), primary_key=True),
    # 定义中间表中的列，声明此列关联到表tag的列tag_id，并指明以此列来作为复合主键的其中一列
    Column('tag_id', Integer, ForeignKey('tag.tag_id'), primary_key=True)
)

# 添加数据到数据库中
article1 = Article(title='my article 1')
article2 = Article(title='my article 2')
tag1 = Tag(name='my tag 1')
tag2 = Tag(name='my tag 2')
article1.tags.append(tag1)
article1.tags.append(tag2)
article2.tags.append(tag1)
article2.tags.append(tag2)
session.add(article1)
session.add(article2)
session.commit()
```

**查询排序：**默认升序，降序一般使用对应属性的desc\(\)方法即可。

* **session中指定排序：**使用Query结果对象的order\_by\(attr\_name\)方法即可，降序使用order\_by\(attr\_name.desc\(\)\)或者使用“-”取反排序也可。
* **\_\_mapper\_args\_\_中指定排序：**\_\_mapper\_args\_\_是一个字典，设置键order\_by的值为对应的属性即可，降序同样使用属性的desc\(\)方法。
* **realtionship中指定排序：**设置order\_by参数的值为对应的属性即可，降序同样使用属性的desc\(\)方法。

**升序降序简单示例：**

```py
# 以下列子以模型类Article中的字段create_time作为示例（字符串中的属性名是数据库中的属性名而不是变量名（没有定义时两者是一样的））
# 方式一：在session中使用order_by查询排序
# 升序
articles = session.query(Article).order_by(Article.create_time).all()
articles = session.query(Article).order_by('create_time').all()
# 降序）
articles = session.query(Article).order_by(Article.create_time.desc()).all()
articles = session.query(Article).order_by('-create_time').all()

# 方式二、在模型类中使用__mapper_args__的order_by指定查询排序
class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    create_time = Column(DateTime, nullable=False, default=datetime.now)

    __mapper_args__ = {
        'order_by': create_time,  # 升序
        # 'order_by': create_time.desc(),  # 降序
    }

# 方式三、使用模型类中relationship的roder_by进行指定查询排序
class User(Base):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True, autoincrement=True)
    username = Column(String(50), nullable=False)

class Article(Base):
    __tablename__ = 'article'
    id = Column(Integer, primary_key=True, autoincrement=True)
    title = Column(String(50), nullable=False)
    create_time = Column(DateTime, nullable=False, default=datetime.now)

    user_id = Column(Integer, ForeignKey('user.id'))

    author = realtionship('User', backref=backref('articles', order_by=create_time))  # 升序
    # author = realtionship('User', backref=backref('articles', order_by=create_time.desc()))  # 降序
```

**query对象常用方法：**

* **limit：**指定每次查询数据的条数，例只查找前面10篇数据：articles = session.query\(Article\).limit\(10\).all\(\)。
* **offset：**指定查询的起始偏移量，例从第5条数据开始，并查询15条数据：articles = session.query\(Article\).offset\(5\).limit\(15\).all\(\)。
* **slice\(start, stop\)：**指定查询的区间（切片操作），例查询第3到第18条数据：articles = session.query\(Article\)\[3:18\]或者articles = session.query\(Article\).slice\(3, 18\).all\(\)，第一种方式最常用，第二种不太常用。

**高级查询：**

* **group\_by：**实现查询分组，例根据年龄统计人数：session.query\(User.age, func.count\(User.id\)\).group\_by\(User.age\).all\(\)。
* **having：**对查询结果进行二次查询，例根据年龄统计人数并筛选出年龄小于18的分组：session.query\(User.age, func.count\(User.id\)\).group\_by\(User.age\).having\(User.age &lt; 18\).all\(\)。
* **join/outterjoin：**即inner join和left join，例统计用户文章数看，并根据用户名分组，根据文章数排序：session.query\(User.username, func.count\(Article.id\)\).join\(Article, User.id == Article.user\_id\).group\_by\(User.username\).order\_by\(func.count\(Article.id\).desc\(\)\).all\(\)，这里如果User和Article之间有唯一的外键在关联的话，join中的连接条件User.id == Article.user\_id是可以不用写的，sqlalchemy会自动使用外键进行连接。
* **subquery：**对查询结果进行子查询，例如查找和用户“Jason”同年龄和同城市的用户：sub\_q = session.query\(User.city.label\('city'\), User.age.label\('age'\)\).filter\(User.username == 'Jason'\).subquery\(\); q\_res = session.query\(User\).filter\(User.city == sub\_q.c.city, User.age == sub\_q.c.age\).all\(\)。

## 四、Flask-SQLAlchemy使用

由于Flask-SQLAlchemy其实是SQLAlchemy在外面进行了一层封装，以便更好的适用于flask，所以Flask-SQLAlchemy和SQLAlchemy在使用上除了最外层封装的定义外都是一样的，下面只讲一些常需要注意的不同之处。

**安装：**pip install flask-sqlalchemy

**配置：**需要在配置文件中配置数据库连接字符串“SQLALCHEMY\_DATABASE\_URI”，其中driver为Python2的mysqldb，Python3的pymysql，具体以安装的插件为准。

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

**使用：**Column、relationship等都可以直接在“from flask\_sqlalchemy import SQLAlchemy”的SQLAlchemy实例中直接使用。

* **实例化：**实例化一个SQLAlchemy对象，需要使用一个Flask对象作为参数传入，如果实例化的时候不传入，可以在需要的时候使用init\_app\(flask\_obj\)方法传入。
* **class模型：**需要继承db.Model，对应数据库中的表，一个class实例对应一条记录。
* **\_\_tablename\_\_：**对应数据库中的表名，若未指定此变量的值，则使用class模型类名的全小写作为表名。

**简单示例（配置信息写在了配置文件config.py中了）**

```py
from flask import Flask
from flask_sqlalchemy import SQLAlchemy
import config

app = Flask(__name__)
app.config.from_object(config)
db = SQLAlchemy(app)
db.create_all()


class User(db.Model):
    __tablename__ = 'user'  # 设置表名，未设置则以类名全小写为表名
    id = db.Column(db.Integer, primary_key=True, autoincrement=True)  # 定义列id，整型，主键，自增长
    username = db.Column(db.String(100), nullable=False)  # 定义列username，字符型（最大100个字符），非空
```



