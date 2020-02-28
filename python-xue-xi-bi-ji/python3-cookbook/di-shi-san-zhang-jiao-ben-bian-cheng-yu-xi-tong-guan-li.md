### 13.7 复制或者移动文件和目录

在使用shutil.copytree进行文件夹的拷贝时，可以使用它的ignore参数来指定忽略的规则，参数值可以是一个函数，此函数接受一个目录名和文件名列表并返回要忽略的名称列表，也可以是shutil.ignore\_patterns指定的忽略规则。

```py
def ignore_pyc_files(dirname, filenames):
    return [name in filenames if name.endswith('.pyc')]

shutil.copytree(src, dst, ignore=ignore_pyc_files)

shutil.copytree(src, dst, ignore=shutil.ignore_patterns('*~', '*.pyc'))
```



### 13.8 创建和解压归档文件

如果只是简单的解压或者压缩文档，使用shutil.unpack\_archive和shutil.make\_archive就可以了。

```py
>>> import shutil
>>> shutil.unpack_archive('Python-3.3.0.tgz')

>>> shutil.make_archive('py33','zip','Python-3.3.0')
'/Users/beazley/Downloads/py33.zip'
>>>
>>># 支持的格式如下
>>> shutil.get_archive_formats()
[('bztar', "bzip2'ed tar-file"), ('gztar', "gzip'ed tar-file"),
 ('tar', 'uncompressed tar file'), ('zip', 'ZIP file')]
>>>
```



### 13.10 读取配置文件

对于配置数据，特别是用户配置数据，应该记录在专门的配置文件中，比如ini文件，对于ini文件，可以使用内置的configparser模块来进行读写。

这里只是列了些简单的读写操作，关于configparser模块更多信息，可以查看官方文档，或者可以参考下我的笔记：[https://www.cnblogs.com/guyuyun/p/10965125.html](https://www.cnblogs.com/guyuyun/p/10965125.html)

config.ini文件内容如下：

```
; config.ini
; Sample configuration file

[installation]
library=%(prefix)s/lib
include=%(prefix)s/include
bin=%(prefix)s/bin
prefix=/usr/local

# Setting related to debug configuration
[debug]
log_errors=true
show_warnings=False

[server]
port: 8080
nworkers: 32
pid-file=/tmp/spam.pid
root=/www/root
signature:
    =================================
    Brought to you by the Python Cookbook
    =================================
```

读取配置文件：

```py
>>> from configparser import ConfigParser
>>> cfg = ConfigParser()
>>> cfg.read('config.ini')
['config.ini']
>>> cfg.sections()
['installation', 'debug', 'server']
>>> cfg.get('installation','library')
'/usr/local/lib'
>>> cfg.getboolean('debug','log_errors')

True
>>> cfg.getint('server','port')
8080
>>> cfg.getint('server','nworkers')
32
>>> print(cfg.get('server','signature'))

\=================================
Brought to you by the Python Cookbook
\=================================
>>>
```

修改配置文件：

```py
>>> cfg.set('server','port','9000')
>>> cfg.set('debug','log_errors','False')
>>> import sys
>>> cfg.write(sys.stdout)  # 如果想将修改内容更新到文件中，将这里的sys.stdout替换为对应文件即可
```



### 13.15 启动一个WEB浏览器

webbrowser模块能被用来启动一个浏览器，并且与平台无关。默认使用系统默认的浏览器，其他支持的浏览器可以查看官方文档。

```py
>>> import webbrowser
>>> webbrowser.open('http://www.python.org')  # 以默认浏览器打开url
True
>>> webbrowser.open_new('http://www.python.org')  # 在一个新的浏览器窗口打开url
True
>>> webbrowser.open_new_tab('http://www.python.org')  # 在浏览器的新标签中打开url
True
>>> c = webbrowser.get('firefox')  # 使用指定的浏览器打开url
>>> c.open('http://www.python.org')
True
```



