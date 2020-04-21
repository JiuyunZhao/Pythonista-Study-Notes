# logging（日志处理）

在一个软件中，日志是可以说必不可少的一个组成部分，通常会在定位客户问题或者记录软件使用情况等场景中会用到。logging模板块是Python的一个内置标准库，用于实现对日志的控制输出，对于平常的日志输出，甚至是系统级的日志输出，也都可以使用logging模块来进行实现。

#### 一、使用basicConfig进行简单的一次性配置

##### basicConfig一次性配置，简单示例：

```py
# -*- coding:utf-8 -*-
import logging
import datetime


# filename：设置日志输出文件，以天为单位输出到不同的日志文件，以免单个日志文件日志信息过多，
# 日志文件如果不存在则会自动创建，但前面的路径如log文件夹必须存在，否则会报错
log_file = 'log/sys_%s.log' % datetime.datetime.strftime(datetime.datetime.now(), '%Y-%m-%d')
# level：设置日志输出的最低级别，即低于此级别的日志都不会输出
# 在平时开发测试的时候可以设置成logging.debug以便定位问题，但正式上线后建议设置为logging.WARNING，既可以降低系统I/O的负荷，也可以避免输出过多的无用日志信息
log_level = logging.WARNING
# format：设置日志的字符串输出格式
log_format = '%(asctime)s[%(levelname)s]: %(message)s'
logging.basicConfig(filename=log_file, level=logging.WARNING, format=log_format)
logger = logging.getLogger()

# 以下日志输出由于level被设置为了logging.WARNING，所以debug和info的日志不会被输出
logger.debug('This is a debug message!')
logger.info('This is a info message!')
logger.warning('This is a warning message!')
logger.error('This is a error message!')
logger.critical('This is a critical message!')
```

##### 运行后，日志文件sys\_2019-04-14.log的内容如下：

```
2019-04-14 23:34:42,444[WARNING]: This is a warning message!
2019-04-14 23:34:42,444[ERROR]: This is a error message!
2019-04-14 23:34:42,444[CRITICAL]: This is a critical message!
```

**logging.basicConfig：**用于对logging模块整个日志输出的一次性配置，也就是说多次配置时以第一次配置的为准，之后再使用basicConfig进行配置则无效。

**logging.basicConfig的参数：**

* **filename：**设置日志输出的文件，默认输出到控制台。
* **filemode：**设置打开日志文件的方式，默认为“a”，即追加。
* **format：**设置日志输出的字符串格式，具体的格式有如下几种：
  * **%\(name\)s：**日志记录器的名称
  * **%\(levelno\)s：**日志级别数值
  * **%\(levelname\)s：**日志级别名称
  * **%\(pathname\)s：**输出日志时当前文件的绝对路径
  * **%\(filename\)s：**输出日志时当前文件名（包含后缀）
  * **%\(module\)s：**输出日志时的模块名（即%\(filename\)s不包含后缀名）
  * **%\(funcName\)s：**输出日志时所在函数名
  * **%\(lineno\)d：**输出日志时在文件中的行号
  * **%\(asctime\)s：**输出日志时的时间
  * **%\(thread\)d：**输出日志的当前线程ID
  * **%\(threadName\)s：**输出日志的当前线程名称
  * **%\(process\)s：**输出日志的当前进程ID
  * **%\(processName\)s：**输出日志的当前进程名称
  * **%\(message\)s：**输出日志的内容
* **datefmt：**设置日志时间字符串的输出格式，默认时间字符串格式为%Y-%m-%d %H:%M:%S。
* **style：**设置format字符串格式化的风格，可以是“%”，“{”或“$”，默认是“%”。
* **level：**设置日志的级别，具体有以下几种（由高到低）：
  * **CRITICAL：**致命错误
  * **ERROR：**严重错误
  * **WARNING：**需要给出的提示
  * **INFO：**一般的日志信息
  * **DEBUG：**在debug过程中的debug信息
* **stream：**指定日志的输出Stream，可以是sys.stderr，sys.stdout或者是文件（即使用open函数打开的文件流，但是这个文件流logging模块不会主动关闭它），默认是sys.stderr，如果同时指定了filename参数和stream参数，那stream参数就会被忽略。



#### 二、使用Handler将日志同时输出到文件和控制台

##### 添加Handler打印日志，简单示例：

```py
# -*- coding:utf-8 -*-
import logging
import datetime


logger = logging.getLogger()
# 设置此logger的最低日志级别，之后添加的Handler级别如果低于这个设置，则以这个设置为最低限制
logger.setLevel(logging.INFO)

# 创建一个FileHandler，将日志输出到文件
log_file = 'log/sys_%s.log' % datetime.datetime.strftime(datetime.datetime.now(), '%Y-%m-%d')
file_handler = logging.FileHandler(log_file)
# 设置此Handler的最低日志级别
file_handler.setLevel(logging.WARNING)
# 设置此Handler的日志输出字符串格式
log_formatter = logging.Formatter('%(asctime)s[%(levelname)s]: %(message)s')
file_handler.setFormatter(log_formatter)

# 创建一个StreamHandler，将日志输出到Stream，默认输出到sys.stderr
stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.INFO)

# 将不同的Handler添加到logger中，日志就会同时输出到不同的Handler控制的输出中
# 注意如果此logger在之前使用basicConfig进行基础配置，因为basicConfig会自动创建一个Handler，所以此logger将会有3个Handler
# 会将日志同时输出到3个Handler控制的输出中
logger.addHandler(file_handler)
logger.addHandler(stream_handler)

# 文件中将会输出WARNING及以上级别的日志
# 控制台将会输出INFO及以上级别的日志
logger.debug('This is a debug message!')
logger.info('This is a info message!')
logger.warning('This is a warning message!')
logger.error('This is a error message!')
logger.critical('This is a critical message!')
```

##### 运行后，日志文件sys\_2019-04-15.log的内容如下：

```
2019-04-15 21:50:40,292[WARNING]: This is a warning message!
2019-04-15 21:50:40,292[ERROR]: This is a error message!
2019-04-15 21:50:40,293[CRITICAL]: This is a critical message!
```

##### 控制台打印的内容如下：

```
This is a info message!
This is a warning message!
This is a error message!
This is a critical message!
```

通过给logger添加不同的Handler，可以将日志同时输出到不同的地方，需要注意的是使用basicConfig进行一次性基础配置时，会根据配置内容自动创建一个Handler，所以如果之后再添加了N个Handler，实际上总的Handler数量有N+1个。



一般在logging或者logging.handlers下就可以你想要的Handler，不同的Handler会以不同的方式输出到不同的地方，以下是几种常用的Handler：

* **from logging import FileHandler: **以“a”（追加）的方式将日志输出到文件，如果文件不存在，则自动创建该文件。
* **from logging import StreamHandler: **将日志输出到Stream，比如sys.stderr、sys.stdour、文件流等。
* **from logging.handlers import RotatingFileHandler: **将日志输出到文件，可以通过设置文件大小，文件达到上限后自动创建一个新的文件来继续输出文件。
* **from logging.handlers import TimedRotatingFileHandler: **将日志输出到文件，可以通过设置时间，使日志根据不同的时间自动创建并输出到不同的文件中。
* **from logging.handlers import HTTPHandler: **将日志发送到一个HTTP服务器。
* **from logging.handlers import SMTPHandler: **将日志发送到一个指定的邮件地址。



#### 三、使用不同的日志级别输出日志

**最低级别：**logger.setLevel为设置logger的最低日志级别，如果handler中也设置了级别，则不能低于这个级别，低于这个级别的设置是无效的。

**不同级别的意义：**在开发或者部署应用程序时，需要尽可能详尽的信息来进行开发和调试，这时候用的比较多的是来自DEBUG和INFO级别的日志信息，但是到了正式上线或者生产环境时，应该使用WARNING和CRITICAL级别的日志信息以降低机器的I/O压力和提高获取到错误信息的效率。

**日志量：**日志的信息量应该是与日志级别成反比的：DEBUG&gt;INFO&gt;WARNING&gt;ERROR&gt;CRITICAL。

**不同级别的设置：**只有日志级别大于或等于指定日志级别的日志才会被输出，所以在程序中可以设置一个比较高的日志级别，比如WARNING，开发和调试的时候修改为DEBUG或INFO以便得到更多的日志信息，正式上线的时候这些日志还是保持WARNING的级别。



#### 四、输出traceback.format\_exc异常信息

可以在打印指定信息的同时在下一行打印出traceback.format\_exc异常信息，如：logger.error\(msg, exc\_info=True\)。如果指定了exc\_info为True来打印错误消息，但是没有发生错误的话，就会在消息的下一行打印“NoneType: None”。

**注：**对于参数exc\_info，其实info，debug等函数也有这个关键字参数，但是不推荐使用这些低级别的打印函数来打印错误消息。



#### 五、配置日志：配置文件和字典配置

**参考文章：**http://www.cnblogs.com/yyds/p/6885182.html

**配置文件方式：**将配置文件使用特定规则配置好对应section和option，然后通过logging.config.fileConfig函数加载并使用文件中的配置信息。

##### **配置文件配置logging，简单示例：**

##### **如下为配置文件log\_cfg.ini的配置内容**

```
[loggers]
keys=root, console

[handlers]
keys=consolehandler, filehandler

[formatters]
keys=consoleformatter, fileformatter

[logger_root]
level=DEBUG
handlers=filehandler

[logger_console]
level=WARNING
handlers=consolehandler
qualname=console
propagate=0

[handler_filehandler]
class=FileHandler
args=('log/sys.log', )
level=WARNING
formatter=fileformatter

[handler_consolehandler]
class=logging.StreamHandler
args=(sys.stdout, )
level=DEBUG
formatter=consoleformatter

[formatter_fileformatter]
format=%(asctime)s[%(name)s][%(levelname)s]: %(message)s

[formatter_consoleformatter]
format=%(asctime)s[%(levelname)s]: %(message)s
```

##### 如下为py代码

```py
# -*- coding:utf-8 -*-
import logging.config

# 加载配置文件
logging.config.fileConfig('log_cfg.ini')

# 获取配置好的logger
logger = logging.getLogger('console')

logger.debug('This is a debug message!')
logger.info('This is a info message!')
logger.warning('This is a warning message!')
logger.error('This is a error message!')
logger.critical('This is a critical message!')

# 获取没有配置的logger时，会自动使用root中配置的Handler
any_logger = logging.getLogger('any')
any_logger.critical('This is a test message!')
```

##### 运行结果：

##### 如下为sys.log文件内容

```
2019-04-21 21:39:43,029[any][CRITICAL]: This is a test message!
```

##### 如下为控制台输出

```
2019-04-21 21:39:43,029[WARNING]: This is a warning message!
2019-04-21 21:39:43,029[ERROR]: This is a error message!
2019-04-21 21:39:43,029[CRITICAL]: This is a critical message!
```

##### 配置规则

* **loggers：**此项为必须配置的section，并使用keys来指定需要配置的logger。keys中指定的logger必须要在文件中有对应的配置，且keys中必须包含root这个logger，但相反，已配置的logger可以不用在loggers下指定，可以在需要使用的使用再添加上去。
* **handlers：**此项为必须配置的section，并使用keys来指定需要配置的handler。keys中指定的handler必须要在文件中有对应的配置，但相反，已配置的handler可以不用在handlers下指定，可以在需要使用的使用再添加上去。
* **formatters：**此项为必须配置的section，并使用keys来指定需要配置的formatter。keys中指定的formatter必须要在文件中有对应的配置，但相反，已配置的formatter可以不用在formatters下指定，可以在需要使用的使用再添加上去。
* **logger：**命名方式为logger\_xxx，xxx为在loggers中指定的logger名称。对于root这个logger，必须指定level和handlers这两个option，而对于非root的其他logger，除了level和handlers必须配置外，还需要配置qualname，代码中使用logging.getLogger\(\)时读取的就是qualname这个option。另外对于非root的logger，还可以指定propagate（可选项），其默认值为1，表示会将消息传递给父logger的handler进行处理，也可以设置为0，表示不往上层传递消息。
* **handler：**命名方式为handler\_xxx，xxx为在handlers中指定的handler名称，class和args为必须配置的option，表示创建handler的类和对应的初始化参数，class可以是相对于logging模块的类，也可以是直接使用import进行导入的类，args则必须是元组格式，哪怕只有一个参数也要是元组的格式。level和formatter为可选的option，且指定的formatter也一定要是在formatters和formatter中已配置好的。
* **formatter：**命名方式为formatter\_xxx，xxx为在formatters中指定的formatter名称，这个section下的option都是可选的，即可以不配置任何option项。但是一般会配置format，用于指定输出字符串的格式，也可以指定datefmt，用于指定输出的时间字符串格式，默认为%Y-%m-%d %H:%M:%S。
* **获取没有配置的logger：**如果代码中获取的是没有配置的logger，则会默认以root的handler来进行处理。



**字典配置方式：**将字典或者类字典的配置文件（如JSON格式的配置文件或者YAML格式的配置文件）使用特定规则配置好对应key和value，通过logging.config.dictConfig函数加载并使用里面的配置信息。这里需要说明的是，dictConfig只需要一个可以解析成字典的对象即可，可以是Python的字典或者其他配置文件，只要能构建出这个字典就行。

##### 以YAML格式的文件来配置字典方式配置logging，简单示例：

##### 如下为配置文件log\_cfg.yml的配置内容

```
version: 1
root:
    level: DEBUG
    handlers: [filehandler, ]
loggers:
    console:
        level: WARNING
        handlers: [consolehandler, ]
        propagate: no
handlers:
    filehandler:
        class: logging.FileHandler
        filename: log/sys.log
        level: WARNING
        formatter: fileformatter
    consolehandler:
        class: logging.StreamHandler
        stream: ext://sys.stdout
        level: DEBUG
        formatter: consoleformatter
formatters:
    fileformatter:
        format: '%(asctime)s[%(name)s][%(levelname)s]: %(message)s'
    consoleformatter:
        format: '%(asctime)s[%(levelname)s]: %(message)s'
```

##### 如下为py代码

```py
# -*- coding:utf-8 -*-
import logging.config
import yaml

# 加载配置文件
with open('log_cfg.yml') as log_cfg_file:
    log_dictcfg = yaml.safe_load(log_cfg_file)

logging.config.dictConfig(log_dictcfg)

# 获取配置好的logger
logger = logging.getLogger('console')

logger.debug('This is a debug message!')
logger.info('This is a info message!')
logger.warning('This is a warning message!')
logger.error('This is a error message!')
logger.critical('This is a critical message!')

# 获取没有配置的logger时，会自动使用root中配置的Handler，如果没有配置root，则会使用默认的root
any_logger = logging.getLogger('any')
any_logger.critical('This is a test message!')
```

**输出结果：**同上一个配置文件方式的示例的输出是相同的。

##### 配置规则

* **version：**必选项，表示配置规则的版本号，不过可用值只有1。
* **root：**可选项，用于配置root，如果没有配置，则会自动创建一个默认的root。
* **loggers：**可选项，用于配置logger，这里propagate可以配置为yes/no，也可以配置为1/0、True/False等，当然也可以不配置，默认为yes下的效果。
* **handlers：**可选项，用于配置handler，配置handler时，除了class为必选项外，其他的都是可选项，且class的值必须是能import的，不能直接是FileHandler这种相对于logging模块的路径。另外，每个handler的配置字典中，除了class、level、formatter和filters外，其他的配置信息会传给class对应类的初始化函数中，比如上面配置文件中的filename和stream。
* **formatters：**可选项，用于配置formatter，可以配置format和datefmt等信息。
* **外部对象访问：**外部对象是指不能直接进行访问，需要import导入的对象，这时候需要加一个ext://前缀以便识别，然后系统就会import这个前缀后面的内容。



#### 六、日志中输出额外信息：日志传参、LoggerAdapter

##### 参考文章：http://www.cnblogs.com/yyds/p/6897964.html

##### 直接传参，简单示例：

```py
ip = '127.0.0.1'
username = 'Jason'
# 第一个参数为格式化消息字符串，第二个及之后的参数为对应的变量
logger.warning('[%s][%s]This is a warning message!', ip, username)
```

##### 使用extra参数，简单示例：

```py
logger = logging.getLogger()
logger.setLevel(logging.DEBUG)

console_handler = logging.StreamHandler()
console_handler.setLevel(logging.WARNING)
# 在formatter中定义好需要额外传递参数，且在之后的日志输出必须给出这些额外参数的字典值
console_formatter = logging.Formatter('%(asctime)s[%(ip)s][%(username)s]: %(message)s')
console_handler.setFormatter(console_formatter)

logger.addHandler(console_handler)

# 将对应的字典信息传给extra
extra = {'ip': '127.0.0.1', 'username': 'Jason'}
logger.warning('This is a warning message!', extra=extra)
# 如果不传入extra参数，则会报错
# logger.warning('This is a warning message!')
```

##### 使用LoggerAdapter设置默认的extra参数，简单示例：

```py
# -*- coding:utf-8 -*-
import logging


class IpUserLoggerAdapter(logging.LoggerAdapter):
    # 原process方法就两行代码，即使用默认的extra值
    # 重写process方法，如果没有传入extra，才使用默认值
    def process(self, msg, kwargs):
        if 'extra' not in kwargs:
            kwargs['extra'] = self.extra

        return msg, kwargs


def get_ipuser_logger():
    logger = logging.getLogger()
    logger.setLevel(logging.DEBUG)

    console_handler = logging.StreamHandler()
    console_handler.setLevel(logging.WARNING)
    console_formatter = logging.Formatter('%(asctime)s[%(ip)s][%(user)s][%(levelname)s]: %(message)s')
    console_handler.setFormatter(console_formatter)

    logger.addHandler(console_handler)

    # 设置默认ip和user，这里的key要和formatter中的key对应
    local_extra = {
        'ip': '127.0.0.1',
        'user': 'Jason'
    }
    return IpUserLoggerAdapter(logger, local_extra)


if __name__ == '__main__':
    ipuser_logger = get_ipuser_logger()
    ipuser_logger.debug('This is a debug message!')
    ipuser_logger.info('This is a info message!')
    ipuser_logger.warning('This is a warning message!')
    # 打印一个临时的ip和user
    ipuser_logger.error('This is a error message!', extra={'ip': '0.0.0.0', 'user': 'anyone'})
    ipuser_logger.critical('This is a critical message!')
```

##### 运行后，控制台输出为：

```
2019-04-24 21:46:44,433[127.0.0.1][Jason][WARNING]: This is a warning message!
2019-04-24 21:46:44,433[0.0.0.0][anyone][ERROR]: This is a error message!
2019-04-24 21:46:44,433[127.0.0.1][Jason][CRITICAL]: This is a critical message!
```



#### 七、子logger和共享配置

**子logger：**在logging模块中，logger是类似于树的层级结构，父logger和子logger之间使用点号“.”连接和识别，如：logging.getLogger\('root'\)，logging.getLogger\('root.child'\)，logging.getLogger\('root.child.grand'\)……

**配置共享：**子logger拥有父logger相同的配置，如果在子logger中又另外增加了一些配置，那么子logger不仅拥有父logger的配置，也有自己新增的配置，不过需要注意的是就算子logger新增的配置信息与父logger相同（比如文件），也会执行为它单独执行一次，即父logger和子logger的配置互不影响，并且子logger输出一条日志时会先执行子logger的配置，再执行父logger的配置。

**隔离父looger：**logger中有一个属性propagate，默认为True，即子logger会共享父logger的配置，可以手动设置为False，这样子logger执行完后就不会再去执行父logger的配置了。

**全局使用：**在程序运行过程中，如果想要使用之前使用的logger，不需要每次都运行一次对应的配置，直接使用logging.getLogger\(name\)得到对应名称的logger即可。

**注：**在使用logging.getLogger\(name=None\)时如果没有传入logger的名称name，那么会自动返回名为root的logger。

##### 简单示例：

```py
# -*- coding:utf-8 -*-
import logging
import datetime

# 创建一个父logger
logger = logging.getLogger('main')
logger.setLevel(logging.INFO)

# 给父logger添加一个文件输出的Handler
log_file = 'log/sys_%s.log' % datetime.datetime.strftime(datetime.datetime.now(), '%Y-%m-%d')
file_handler = logging.FileHandler(log_file)
file_handler.setLevel(logging.WARNING)
log_formatter = logging.Formatter('%(asctime)s[%(name)s][%(levelname)s]: %(message)s')
file_handler.setFormatter(log_formatter)
logger.addHandler(file_handler)

# 向文件输出一条日志
logger.error('This is a error message!')

# 创建一个子logger
child_logger = logging.getLogger('main.child')
# 给子logger添加一个输出到控制台的Handler
stream_handler = logging.StreamHandler()
stream_handler.setLevel(logging.WARNING)
child_logger.addHandler(stream_handler)
# 设置propagate属性为False可以隔离父logger的配置
# child_logger.propagate = False

# 同时向文件和控制台输出一条日志
child_logger.warning('This is a warning message!')
```

##### 运行后，日志文件sys\_2019-04-16.log文件内容如下：

```
2019-04-16 00:09:53,154[main][ERROR]: This is a error message!
2019-04-16 00:09:53,154[main.child][WARNING]: This is a warning message!
```

##### 控制台打印的内容如下：

```
This is a warning message!
```



