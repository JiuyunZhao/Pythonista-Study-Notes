# datetime模块（日期）

datetime模块通常用来操作日期信息（年月日和时分秒），常用的方法有：

* **datetime.datetime.now\(\)：**返回当前日期时间的datetime对象，对象中包含年月日和时分秒信息，可通过str它来得到日期时间信息的字符串。
* **datetime.datetime.fromtimestamp\(timestamp）：**将时间戳转换成datetime对象，并返回。
* **datetime.datetime.strptime\(date\_string, format\)：**将字符串按照指定格式转成datetime对象，并返回，如：datetime.datetime.strptime\('2016-09-04', '%Y-%m-%d'\)。
* **datetime.datetime.strftime\(datetime, format\)：**将datetime对象转换为指定格式的字符串，并返回，如：datetime.datetime.strftime\(datetime.datetime.now\(\), '%Y-%m-%d %H:%M:%S'\)。
* **datetime.timedelta：**timedelta对象初始化时指定日期时间信息，可用它与datetime对象进行加减操作，并返回新的datetime对象，当然timedelta对象之间也可以进行加减操作，返回新的timedelta对象。
* **timetuple\(\)：**将datetime对象转换成struct\_time对象，并返回。

## 日期时间格式化字符串：

| %a | 星期的缩写，如星期三Wed。 |
| :--- | :--- |
| %A | 星期的全名，如星期三Wednesday。 |
| %b | 月份的缩写，如四月份Apr。 |
| %B | 月份的全名，如4月份April。 |
| %c | 日期和时间字符串表示，如04/07/10 10:43:39。 |
| %d | 每月的第几天。 |
| %p | AM或PM。 |
| %H | 小时（24小时格式）。 |
| %I | 小时（12小时格式）。 |
| %j | 一年中的第几天。 |
| %m | 月份。 |
| %M | 分钟。 |
| %S | 秒。 |
| %f | 微秒。 |
| %U | 一年中的第几周（星期天为一周的第一天，新的一年的第一个星期天被认为是第0周的开始）。 |
| %w | 星期几（星期天为0）。 |
| %W | 一年中的第几周（星期一为一周的第一天，新的一年的第一个星期一被认为是第0周的开始）。 |
| %x | 日期字符串。 |
| %X | 时间字符串。 |
| %y | 年份（不含世纪，即两个数字表示）。 |
| %Y | 年份（含世纪，即4个数字表示）。 |
| %z | 时区名称。 |
| %Z | Time zone offset indicating a positive or negative time difference from UTC/GMT of the form +HHMM or -HHMM, where H represents decimal hour digits and M represents decimal minute digits \[-23:59, +23:59\]. |
| %% | 百分号“%”。 |

## 简单示例：

```text
>>> import datetime
>>> # now()方法，返回当期时间的datetime对象，可通过str得到日期时间信息的字符串
>>> datetime_now = datetime.datetime.now()
>>> datetime_now
datetime.datetime(2018, 8, 9, 0, 59, 3, 487000)
>>> str(datetime_now)
'2018-08-09 00:59:03.487000'
>>> # fromtimestamp()方法，将时间戳转换成datetime对象，并返回
>>> datetime.datetime.fromtimestamp(time.time())
datetime.datetime(2018, 8, 9, 1, 2, 51, 546000)
>>> # strptime()方法，将字符串按照指定格式转成datetime对象
>>> datetime.datetime.strptime('2016-09-04', '%Y-%m-%d')
datetime.datetime(2016, 9, 4, 0, 0)
>>> # strftime()方法，将datetime对象转换为指定格式的字符串，并返回
>>> datetime.datetime.strftime(datetime.datetime.now(), '%Y-%m-%d %H:%M:%S')
'2018-08-09 01:18:10'
>>> # timedelta对象，可用于日期时间信息的加减操作
>>> timedelta_hour = datetime.timedelta(hours=4)
>>> timedelta_day = datetime.timedelta(days=8)
>>> timedelta_new = timedelta_hour + timedelta_day
>>> timedelta_hour
datetime.timedelta(0, 14400)
>>> timedelta_day
datetime.timedelta(8)
>>> timedelta_new
datetime.timedelta(8, 14400)
>>> datetime.datetime.now()
datetime.datetime(2018, 8, 9, 1, 30, 0, 593000)
>>> datetime_new = datetime.datetime.now() + timedelta_new
>>> datetime_new
datetime.datetime(2018, 8, 17, 5, 30, 1, 70000)
>>> # timetuple()方法，将datetime对象转换成struct_time对象
>>> now_datetime = datetime.datetime.now()
>>> now_datetime.timetuple()
time.struct_time(tm_year=2018, tm_mon=8, tm_mday=9, tm_hour=2, tm_min=5, tm_sec=50, tm_wday=3, tm_yday=221, tm_isdst=-1)
```

