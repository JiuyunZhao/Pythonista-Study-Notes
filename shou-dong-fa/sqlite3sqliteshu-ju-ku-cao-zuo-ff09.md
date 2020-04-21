# sqlite3（sqlite数据库操作）

对于数据库的操作，Python中可以通过下载一些对应的三方插件和对应的数据库来实现数据库的操作，但是这样不免使得Python程序变得更加复杂了。如果只是想要使用数据库，又不想下载一些不必要的插件和辅助软件，可以使用Python内置模块sqlite3。

sqlite3是Python对于sqlite数据库的支持，模块名称之所以是sqlite3而不是sqlite，是因为sqlite的版本中最流行的就是3.x的版本。

**sqlite数据库教程：**[http://www.runoob.com/sqlite/sqlite-intro.html](http://www.runoob.com/sqlite/sqlite-intro.html)

**sqlite GUI软件SQLiteSpy：**[https://www.yunqa.de/delphi/products/sqlitespy/index，数据库的可视化管理工具有很多，你也可以选择其他的软件，但是SQLiteSpy这个软件免费而且非常小，对于平常的sqlite数据库操作其实已经可以满足了。](https://www.yunqa.de/delphi/products/sqlitespy/index，数据库的可视化管理工具有很多，你也可以选择其他的软件，但是SQLiteSpy这个软件免费而且非常小，对于平常的sqlite数据库操作其实已经可以满足了。)

#### 常用API（方括号中为可选参数）

**conn = sqlite3.connect\(database\[, optional\_params\]\)：**连接（磁盘上的）数据库database，database是一个数据库文件，可以是相对路径，也可以是绝对路径，如果这个文件存在且连接成功则返回一个连接对象；如果文件不存在，则自动创建一个数据库文件。

**csr = conn.cursor\(\[optional\_params\]\)：**获取去连接对象的游标对象，游标可以理解为存放SQL结果的缓冲区，也可以理解为指向SQL结果的光标。

**csr.execute\(sql\[, optional\_params\]\)：**执行SQL语句，返回值也是一个cursor游标对象。SQL语句可以使用问号“?”作为占位符，例如：csr.execute\('INSERT INTO STUDENTS VALUES\(?, ?, ?\)', \(1, 'Jason', 18\)\) ，其中表students有三个字段ID、NAME和AGE。

**csr.executemany\(sql, seq\_of\_parameters\)：**对seq\_of\_parameters中的所有数据执行同一个sql。例如需要向数据库中插入大量数据时，sql中可以使用占位符，要插入的数据放入seq\_of\_parameters中。

**conn.commit\(\)：**提交事务。如果之前对数据库进行了修改，比如插入操作，就需要先commit\(\)。

**conn.close\(\)：**关闭数据库连接。关闭数据库连接之前，如果对数据库进行了修改，就需要先commit，关闭的时候不会自动commit的，如果没有commit，那么所有的修改都会被丢失。

**csr.fetchone\(\)：**获取查询结果集中的下一行，返回的是一个表示一行数据的元组，如果没有数据则返回None。

**csr.fetchmany\(size\)：**获取查询结果集中的指定大小的下一组数据，默认返回一行数据（不同与fetchone，返回的是列表，而列表中只有一个元组），如果没有数据则返回空列表。

**csr.fetchall\(\)：**返回查询结果集中的全部数据。

##### 简单示例

```py
# -*- coding:utf-8 -*-
import sqlite3

# 数据库文件
db = 'sqlite_test.db'

# 连接数据库，如果没有则创建数据库
conn = sqlite3.connect(db)

# 获取游标
csr = conn.cursor()

# 创建表STUDENTS
csr.execute('CREATE TABLE STUDENTS'
            '(ID INT PRIMARY KEY NOT NULL,'
            'NAME TEXT NOT NULL,'
            'AGE INT NOT NULL);')

# 插入一条数据
csr.execute('INSERT INTO STUDENTS VALUES(1, "Jason", 18)')

# 插入语句sql，并使用占位符
insert_sql = 'INSERT INTO STUDENTS VALUES(?, ?, ?)'

# 需要批量插入的数据
students_info = [(2, 'Mickle', 20),
                 (3, 'Mary', 30),
                 (4, 'Bob', 40)]

# 批量插入数据
csr.executemany(insert_sql, students_info)

# 提交事务
conn.commit()

# 获取一行数据
# datas = csr.execute('SELECT * FROM STUDENTS;').fetchone()

# 获取2行数据
# datas = csr.execute('SELECT * FROM STUDENTS;').fetchmany(2)

# 获取全部数据
# datas = csr.execute('SELECT * FROM STUDENTS;').fetchall()

# 关闭数据库连接
conn.close()
```

#### 常用SQL

sqlite的更多知识可以参考笔记开始的sqlite教程。

##### 查询数据库中的表：

SELECT name FROM sqlite\_master WHERE type='table';

##### 查询表中的字段信息：

PRAGMA TABLE\_INFO\(\[table\_name\]\);

##### 使用分享：

1. SQL中的表名或者字段名带有特殊符号时应该使用嵌套引号括起来，这些特殊符号可能是语法的一部分，应该使用嵌套引号表名它是一个整体，特别是使用格式化字符串时别忘了使用引号嵌套。如包含点号：'SELECT "aa.bb" FROM xxx;'或者'SELECT xxx FROM "%s";' % 'ccc.ddd.eee'



