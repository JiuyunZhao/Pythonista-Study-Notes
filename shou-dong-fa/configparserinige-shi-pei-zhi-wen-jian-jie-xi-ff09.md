# configparser（INI格式配置文件解析）

在平时的开发中感觉INI格式的配置文件使用还是挺需要的，有时会使用一个单独的py来存放一些常量或者配置项，大多时候这样倒是挺好用的，但是如果某些配置项需要在运行时由用户来修改指定，比如很多app在关闭时会有一个弹出框提示“是否关闭”和“下次不再提醒”，这种配置项如果使用INI格式的配置文件来操作的话就会方便很多，Python中操作配置文件的模块为configparser，这个模块可以用来解析与Windows上INI文件结构类似的文件。

**官方文档：**https://docs.python.org/3/library/configparser.html

看官方文档的时候发现不同Python版本之间某些API还是有些小区别的，所以先说一下，本文使用的是Python3.6。

##### 一个普通的INI配置文件cfg.ini示例如下：

```INIFILE
; DEFAULT为默认section，当获取其他section中同名option，而该section又没有这个option时，会取DEFAULT中的该option
[DEFAULT]
close_prompt = yes

[baidu]
website = www.baidu.com

# 本机信息
[home]
ip = 127.0.0.1
port = 8080
```

#### INI配置文件组成：

* **section：**表示一个区块，由方括号及方括号中的名称组成，section的范围为当前方括号到下一个方括号的内容，如“DEFAULT”，“baidu”，“home”。

  * **大小写和空格检查：**section中的名称在保存和获取的时候是原样保存和获取的，即大小写不一样或者空格不一样等都是不同的section；

  * **重复性检查：**同一个配置文件中section名称是不允许重复的。

* **option：**表示section中的配置项，由key、分隔符和value组成的键值对，如“home”下的“port=8080”。

  * **大小写检查：**key是大小写不敏感的，保存进文件的时候会自动将key小写保存，但value是大小写敏感的；

  * **空格检查：**通过key获取value时，会自动将文件中的key和value前后空格去掉再进行匹配，即文件中保存为'  ip     = 127.0.0.1      '时，用'ip'也可以获取到对应的value值'127.0.0.1';

  * **跨多行检查：**key是不能跨行的，但是value可以跨行，只要第二行即之后的行的缩进与第一行不同即可，一直到下一个option为止；

  * **重复性检查：**和section一样，同一section下的key是不允许重复的；

  * **分隔符：**可以是等号“=”或者冒号“:”。

* **注释：**行注释用井号“\#”或者分号“;”表示，特别需要注意的是必须得是行开头（前面可以有空格），用在行中间的就不会算作是注释了。

* **DEFAULT：**这是一个特殊的section，会用作其他section的option取不到值时的备用值，或者可以理解为它是一个root，其他的section都是它的子section，但不是必须提供的。、

##### 

##### 向配置文件中写数据：

```py
# -*- coding:utf-8 -*-
from configparser import ConfigParser

# 使用字典的方式给配置对象添加配置信息
config = ConfigParser()
config['DEFAULT'] = {
    'close_prompt': 'yes',
}
config['baidu'] = {}
config['baidu']['website'] = 'www.baidu.com'
config['home'] = {}
home = config['home']
home['ip'] = '127.0.0.1'
home['port'] = '8080'

# 将配置信息写入文件
with open('cfg.ini', 'w') as cfg_file:
    config.write(cfg_file)
```

##### 从配置文件中读取数据：

```py
# -*- coding:utf-8 -*-
from configparser import ConfigParser

# 以字典的方式读取配置对象中的数据
config = ConfigParser()

print(config.sections())  # 输出：[]

# 从配置文件中读取数据，如果配置文件中有中文信息，注意编码
config.read('cfg.ini', encoding='utf-8')

print(config.sections())  # 输出：['baidu', 'home']
print('baidu' in config)  # 输出：True
print(config['baidu']['website'])  # 输出：www.baidu.com

home = config['home']
print(home['ip'])  # 输出：127.0.0.1
for key in home:
    print(key)  # 依次输出：ip,port,close_prompt
```



### configparser.ConfigParser

从上面的例子可以看出ConfigParser实例可以像操作字典一样去操作它，每个section对应一个由key/value组成的option字典，虽然可以通过添加和设置section和option等方法来操作，但还是推荐使用字典的方式，可读性也要强一点。其实还有另一个解析类RawConfigParser，与ConfigParser的区别在于前者不允许进行字符串的格式化，而且后者也是继承自前者的，所以这里就只讲ConfigParser。（至于还有一个SafeConfigParser，在Python3.2之后就合道ConfigParser中，跟RawConfigParser和ConfigParser的区别在于可以跨section进行字符串的格式化，这里也不讲了）



**初始化方法：**ConfigParser\(defaults=None, dict\_type=\_default\_dict, allow\_no\_value=False, \*, delimiters=\('=', ':'\), comment\_prefixes=\('\#', ';'\), inline\_comment\_prefixes=None, strict=True, empty\_lines\_in\_values=True, default\_section=DEFAULTSECT, interpolation=\_UNSET, converters=\_UNSET\)：

* **defaults：**设置配置文件中名为DEFAULT的默认section信息，默认为None，可以传入一个包含option信息的字典；
* **dict\_type：**设置读取配置信息时的字典类型，默认为有序字典，即collections.OrderedDict，如果实在要考虑性能等原因，可以使用python默认字典dict；
* **allow\_no\_value：**是否允许key没有对应的value，默认为False，如果加载的配置文件中有这种情况，需要手动设置为True；
* **delimiters：**设置分隔符，默认为“=”和“:”，且一个option中第二个及之后的分隔符会算作value的一部分；
* **comment\_prefixes：**设置注释符，默认为“\#”和“;”，即一行的开头（取出空格后）为“\#”或者“;”，则这一行算作注释内容，包括value有多行的情况也是如此；
* **inline\_comment\_prefixes：**设置行中的注释前缀，即一行中这个符号之后的内容被认为是注释；
* **strict：**默认为True，即读取配置数据时不允许出现重复的section和option；
* **empty\_lines\_in\_values：**是否允许value中出现空行，默认为True，如果设置为False，则value中的空行将作为这个option的结束标志；
* **default\_section：**更改默认的section名称，原本默认的section是DEFAULT（注意更改操作需要在实例化之后，读取数据之前）；
* **interpolation：**设置value的字符串格式化功能，如果不想使用value的字符串格式化功能，可以设置None；
* **converters：**设置将value转换为特定类型的数据，参数值为一个字典，字典的key为转换方法的名称，value为对应的转换函数，提供这个字典后，会自动生成对应的get/set方法，比如提供一个字典{'int': int}就会生成getint转换方法（当然这个方法已经内置有了，这里只是举个例子）。



**value字符串格式化：**可以使用%\(name\)s进行字符串的格式化，且name只能是本section和DEFAULT中的option项。

**自定义option的key配置方式：**如原本是key是自动转化为小写的，现在设置其区分大小写：parser.optionxform = lambda option: option（注意更改操作需要在实例化之后，读取数据之前）。

**自定义section自定义配置方式：**如原本section是包含了空格以及大小写区分的，现在利用正则表达式设置其去掉首位的空格：parser.SECTCRE = re.compile\(r"\\[ \*\(?P&lt;header&gt;\[^\]\]+?\) \*\\]"\)（注意更改操作需要在实例化之后，读取数据之前）。



#### 常用方法：

* **defaults\(\)：**以字典的方式返回默认的section，即DEFAULT；
* **sections\(\)：**返回section名称的列表，但是不包括DEFAULT；
* **add\_section\(section\)：**添加一个section，字符串类型，且已经存在的section不能再往里添加；
* **has\_section\(section\)：**判断当前配置中是否有此section，DEFAULT不包含在此判断中；
* **options\(section\)：**返回此section下的option列表；
* **has\_option\(section, option\)：**如果指定的section存在，且包含该option，则返回True，否则返回False；如果传入的section为None或者空字符串，则使用DEFAULT这个section进行查找判断；
* **read\(filenames, encoding=None\)：**可以传入单个文件，或者多个文件的列表，如果多个文件中某个文件无法打开，则这个文件会被忽略；
* **read\_file\(f, source=None\)：**从一个文件流（不是文件名称）读取配置，source为文件流的名称；
* **read\_string\(string, source='&lt;string&gt;'\)：**从一个字符串读取配置，source为字符串的名称；
* **read\_dict\(dictionary, source='&lt;dict&gt;'\)：**从一个类字典对象中读取配置信息，source为类字典对象的名称；
* **get\(section, option, \*, raw=False, vars=None\[, fallback\]\)：**获取section下指定option的值，如果vars被提供了（必须是一个字典），则按照vars、section、DEFAULT这个顺序进行查找。raw指定为True时，option中value值不会自动进行格式化字符串的转换，直接返回原内容。fallback用于指定当查找的option没有时返回的默认值；
* **getint\(section, option, \*, raw=False, vars=None\[, fallback\]\)：**将get的值强转成int类型（raw、vars和fallback参数请参考get方法）；
* **getfloat\(section, option, \*, raw=Fasle, vars=None\[, ballback\]\)：**将get的值强转成float类型（raw、vars和fallback参数请参考get方法）；
* **getboolean\(section, option, \*, raw=False, vars=None\[, fallback\]\)：**将get的值强转成boolean类型True或False，转换原则为yes/no、on/off、true/false和1/0可以转换为True和False，其他项则会报错（raw、vars和fallback参数请参考get方法）。如果想要自定义转换为True或False的项，可以通过设置parser.BOOLEAN\_STATES来定指定，如：parser.BOOLEAN\_STATES = {'open': True, 'close': False}，但是这时候意味着没在这个字典中的项就会报错了，包括原来的yes/no等项；
* **items\(\(raw=False, vars=None\)：**返回section的迭代器，包括DEFAULT（raw和vars参数请参考get方法）；
* **items\(section, raw=False, vars=None\)：**返回指定section下option键值对元组的列表（raw和vars参数请参考get方法）；
* **set\(section, option, value\)：**给指定的section设置一个option键值对；
* **write\(fileobject, space\_around\_delimiters=True\)：**将配置信息写进一个open的文件对象，space\_around\_delimiters为True时，option的分隔符两边会有空格；
* **remove\_option\(section, option\)：**移除指定section下的指定option，成功返回True，否则返回False；
* **remove\_section\(section\)：**移除一个section，如果存在且移除成功，则返回True，否则返回False。



