# time模块（时间）

time模块通常用来操作时间戳信息（各种“秒”），常用的方法有：

* **time.sleep\(seconds\)：**将当前程序阻塞指定秒数，然后继续运行程序。
* **time.time\(\)：**返回当前时间的时间戳，即1970年到现在经过的浮点秒数。
* **time.struct\_time：**struct\_time类初始化时需要传入一个由9个时间信息组成的元组，该元组的时间信息依次为：tm\_year=2018（年），tm\_mon=8（月），tm\_mday=8（日），tm\_hour=23（时），tm\_min=38（分），tm\_sec=54（秒），tm\_wday=2（星期，星期一为0），tm\_yday=220（一年中的第几天），tm\_isdst=0（1表示支持夏令时，0表示不支持夏令时，-1表示未知）。当然，一个对象是struct\_time对象，可以通过属性的方式获取这些值，比如：time.localtime\(\).tm\_year。
* **time.localtime\(seconds=None\)：**返回当前时间的struct\_time对象，也可以指定秒数seconds（默认采用time.time\(\)的int值），则返回1970加上指定秒数后的struct\_time对象。
* **time.mktime\(p\_tuple\)：**将struct\_time对象（元组）转换为时间戳，然后返回。
* **time.strptime\(string, format\)：**将时间字符串string按照指定格式format转化成struct\_time对象，如：time.strptime\('2016-09-04', '%Y-%m-%d'\)。
* **time.strftime\(format, p\_tuple=None\)：**将struct\_time对象（元组）转换成指定格式的字符串，默认使用time.localtime\(\)的结果来进行转换，如：time.strftime\('%Y-%m-%d %H:%M:%S', time.localtime\(\)\)。

**注：**方法中的参数p\_tuple表示可以是struct\_time对象，也可以是满足struct\_time对象初始化的时间信息元组，其实struct\_time对象就可以看成是一个由时间信息组成的元组。

##### 简单示例：

```
>>> import time
>>> # time.time()方法，返回当前时间的时间戳（1970年到现在经过的浮点秒数）
>>> time.time()
1473691580.9504104
>>> # time.struct_time对象，存储时间信息
>>> structtime_obj = time.struct_time((2018, 8, 8, 23, 38, 54, 2, 220, 0))
>>> structtime_obj
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=8, tm_hour=23, tm_min=38, tm_sec=54, tm_wday=2, tm_yday=220, tm_isdst=0)
>>> structtime_obj.tm_year
>>> # time.localtime()方法，默认返回即当前时间（time.time()的int值）的struct_time对象
>>> time.localtime()
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=8, tm_hour=23, tm_min=22, tm_sec=2, tm_wday=2, tm_yday=220, tm_isdst=0)
>>> time.localtime(time.time())
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=8, tm_hour=23, tm_min=22, tm_sec=2, tm_wday=2, tm_yday=220, tm_isdst=0)
>>> time.localtime(80000)
time.struct_time(tm_year=1970, tm_mon=1, tm_mday=2, tm_hour=6, tm_min=13, tm_sec=20, tm_wday=4, tm_yday=2, tm_isdst=0)
>>> # time.mktime()方法，将struct_time对象转换成时间戳
>>> time.mktime(time.localtime())
1533746402.0
>>> # time.strptime()方法，将字符串按照指定格式转换为struct_time对象
>>> time.strptime('2016-09-04', '%Y-%m-%d')
time.struct_time(tm_year=2016, tm_mon=9, tm_mday=4, tm_hour=0, tm_min=0, tm_sec=0, tm_wday=6, tm_yday=248, tm_isdst=-1)
>>> # time.strftime()方法，将struct_time对象转换成指定格式的字符串
>>> time.strftime('%Y-%m-%d %H:%M:%S', time.localtime())
'2018-08-09 00:43:04'
```



