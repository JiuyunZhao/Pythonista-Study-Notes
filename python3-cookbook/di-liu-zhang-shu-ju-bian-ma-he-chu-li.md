###  6.1 读写CSV数据

对于CSV文件，如果不是需要特殊处理，为了尽可能少地出意外，那么总是应该选择CSV模块来读写CSV文件。下面只列几个简单读写CSV文件的示例：

CSV文件stocks.csv，内容如下：

```
Symbol,Price,Date,Time,Change,Volume
"AA",39.48,"6/11/2007","9:36am",-0.18,181800
"AIG",71.38,"6/11/2007","9:36am",-0.15,195500
"AXP",62.58,"6/11/2007","9:36am",-0.46,935000
"BA",98.31,"6/11/2007","9:36am",+0.12,104800
"C",53.08,"6/11/2007","9:36am",-0.25,360900
"CAT",78.29,"6/11/2007","9:36am",-0.23,225400
```

```py
import csv

# 以列表形式读取数据
with open('stocks.csv') as f:
    f_csv = csv.reader(f)
    headers = next(f_csv)
    # headers和row都是一个列表
    print(headers)
    for row in f_csv:
        print(row)
```

```py
import csv

# 以字典形式读取数据
with open('stocks.csv') as f:
    f_csv = csv.DictReader(f)
    # row是一个OrderedDict字典类型
    for row in f_csv:
        # 第一条输出为：OrderedDict([('Symbol', 'AA'), ('Price', '39.48'), ('Date', '6/11/2007'), ('Time', '9:36am'), ('Change', '-0.18'), ('Volume', '181800')])
        print(row)
```

```py
headers = ['Symbol','Price','Date','Time','Change','Volume']
rows = [('AA', 39.48, '6/11/2007', '9:36am', -0.18, 181800),
         ('AIG', 71.38, '6/11/2007', '9:36am', -0.15, 195500),
         ('AXP', 62.58, '6/11/2007', '9:36am', -0.46, 935000),
       ]

# 以列表形式写入数据
with open('stocks.csv','w') as f:
    f_csv = csv.writer(f)
    # 写入单行数据
    f_csv.writerow(headers)
    # 写入多行数据
    f_csv.writerows(rows)
```

```py
headers = ['Symbol', 'Price', 'Date', 'Time', 'Change', 'Volume']
rows = [{'Symbol':'AA', 'Price':39.48, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.18, 'Volume':181800},
        {'Symbol':'AIG', 'Price': 71.38, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.15, 'Volume': 195500},
        {'Symbol':'AXP', 'Price': 62.58, 'Date':'6/11/2007',
        'Time':'9:36am', 'Change':-0.46, 'Volume': 935000},
        ]

# 以字典形式写入数据
with open('stocks.csv','w') as f:
    f_csv = csv.DictWriter(f, headers)
    f_csv.writeheader()
    f_csv.writerows(rows)
```



### 6.3 解析简单的XML数据

就如此小节的标题所写，这里只讲了简单的XML解析，如果是较小且不复杂的XML文件，可以使用内置的xml.etree.ElementTree，如果是复杂的XML文档，可以使用三方库lxml，功能更加强大且速度更快。对于以下示例代码，可以直接替换为from lxml.etree import parse。

```py
from urllib.request import urlopen
from xml.etree.ElementTree import parse

# 下载XML文件并解析
u = urlopen('http://planet.python.org/rss20.xml')
doc = parse(u)

# 查找节点channel下的title节点
e = doc.find('channel/title')
# 打印节点名称：title
print(e.tag)
# 打印节点文本：Planet Python
print(e.text)
# 打印节点的某个属性值，因为这个节点没有其他属性，所以获取xxx的结果就是None
print(e.get('xxx'))

# 遍历channel下的item节点
for item in doc.iterfind('channel/item'):
    # 在item节点中查找对应子节点的文本
    title = item.findtext('title')
    date = item.findtext('pubDate')
    link = item.findtext('link')

    print(title)
    print(date)
    print(link)
    print()
```

```py
title
Planet Python
None
Codementor: Automating Everything With Python: Reading Time: 3 Mins
Sat, 22 Feb 2020 09:01:58 +0000
https://www.codementor.io/maxongzb/automating-everything-with-python-reading-time-3-mins-13v57qt7y6

Quansight Labs Blog: My Unexpected Dive into Open-Source Python
Fri, 21 Feb 2020 18:38:07 +0000
https://labs.quansight.org/blog/2020/02/my-unexpected-dive-into-open-source-python/

...
```



### 6.4 增量式解析大型XML文件

如果需要解析的XML文件太大，那么可以考虑使用from xml.etree.ElementTree import iterparse进行增量式解析，需要说明的是，以下示例的两个版本中，将整个XML文档加载到内存中的做法性能要优于增量式解析，但是在内存的占用消耗上却是要远远大于增量式解析了。

需要解析的XML文件potholes.xml部分内容如下，现在需要对row节点中zip节点的内容进行统计：

```
<response>
    <row>
        <row ...>
            <creation_date>2012-11-18T00:00:00</creation_date>
            <status>Completed</status>
            <completion_date>2012-11-18T00:00:00</completion_date>
            <service_request_number>12-01906549</service_request_number>
            <type_of_service_request>Pot Hole in Street</type_of_service_request>
            <current_activity>Final Outcome</current_activity>
            <most_recent_action>CDOT Street Cut ... Outcome</most_recent_action>
            <street_address>4714 S TALMAN AVE</street_address>
            <zip>60632</zip>
            <x_coordinate>1159494.68618856</x_coordinate>
            <y_coordinate>1873313.83503384</y_coordinate>
            <ward>14</ward>
            <police_district>9</police_district>
            <community_area>58</community_area>
            <latitude>41.808090232127896</latitude>
            <longitude>-87.69053684711305</longitude>
            <location latitude="41.808090232127896"
            longitude="-87.69053684711305" />
        </row>
        <row ...>
            <creation_date>2012-11-18T00:00:00</creation_date>
            <status>Completed</status>
            <completion_date>2012-11-18T00:00:00</completion_date>
            <service_request_number>12-01906695</service_request_number>
            <type_of_service_request>Pot Hole in Street</type_of_service_request>
            <current_activity>Final Outcome</current_activity>
            <most_recent_action>CDOT Street Cut ... Outcome</most_recent_action>
            <street_address>3510 W NORTH AVE</street_address>
            <zip>60647</zip>
            <x_coordinate>1152732.14127696</x_coordinate>
            <y_coordinate>1910409.38979075</y_coordinate>
            <ward>26</ward>
            <police_district>14</police_district>
            <community_area>23</community_area>
            <latitude>41.91002084292946</latitude>
            <longitude>-87.71435952353961</longitude>
            <location latitude="41.91002084292946"
            longitude="-87.71435952353961" />
        </row>
    </row>
</response>
```

全部加载到内存中解析：

```py
from xml.etree.ElementTree import parse
from collections import Counter

potholes_by_zip = Counter()

doc = parse('potholes.xml')
for pothole in doc.iterfind('row/row'):
    potholes_by_zip[pothole.findtext('zip')] += 1
for zipcode, num in potholes_by_zip.most_common():
    print(zipcode, num)
```

增量式解析：

```py
from xml.etree.ElementTree import iterparse
from collections import Counter


def parse_and_remove(filename, path):
    path_parts = path.split('/')
    # start事件：某个节点被创建时产生
    # end事件：某个节点被创建完成时产生
    doc = iterparse(filename, ('start', 'end'))
    # 跳过根节点
    next(doc)

    tag_stack = []
    elem_stack = []
    for event, elem in doc:
        if event == 'start':
            tag_stack.append(elem.tag)
            elem_stack.append(elem)
        elif event == 'end':
            if tag_stack == path_parts:
                yield elem
                # 此处是减少内存消耗的核心语句：把yield产生的元素从它的父节点中删除掉
                elem_stack[-2].remove(elem)
            try:
                tag_stack.pop()
                elem_stack.pop()
            except IndexError:
                pass


potholes_by_zip = Counter()

data = parse_and_remove('potholes.xml', 'row/row')
for pothole in data:
    potholes_by_zip[pothole.findtext('zip')] += 1
for zipcode, num in potholes_by_zip.most_common():
    print(zipcode, num)
```



### 6.5 将字典转换为XML

from xml.etree.ElementTree import Element可以用来创建一个XML，但需要注意的是它只能构造字符串类型的值。

```py
from xml.etree.ElementTree import Element, tostring


def dict_to_xml(tag, d):
    """根据一个字典创建一个XML"""
    elem = Element(tag)
    for key, val in d.items():
        child = Element(key)
        # text的值需要是str类型
        child.text = str(val)
        elem.append(child)
    return elem


s = {'name': 'GOOG', 'shares': 100, 'price': 490.1}
e = dict_to_xml('stock', s)
# 给某个节点设置属性值
e.set('_id', '1234')
print(e)
print(tostring(e))
```

```
<Element 'stock' at 0x000001761DB01B88>
b'<stock _id="1234"><name>GOOG</name><shares>100</shares><price>490.1</price></stock>'
```



### 6.6 解析和修改XML

示例中修改XML时需要注意的是，所有的修改都是针对父节点来操作的，并且可以将它视为一个列表来处理。

* **删除节点：**使用父节点的remove\(\)方法。
* **添加节点：**使用父节点的insert\(\)和append\(\)方法。
* **索引和切片：**可以对节点使用如element\[i\]或element\[i:j\]进行索引和切片操作。
* **创建新节点：**使用Element类即可。

预先准备好的的文件pred.xml：

```
<?xml version="1.0"?>
<stop>
    <id>14791</id>
    <nm>Clark &amp; Balmoral</nm>
    <sri>
        <rt>22</rt>
        <d>North Bound</d>
        <dd>North Bound</dd>
    </sri>
    <cr>22</cr>
    <pre>
        <pt>5 MIN</pt>
        <fd>Howard</fd>
        <v>1378</v>
        <rn>22</rn>
    </pre>
    <pre>
        <pt>15 MIN</pt>
        <fd>Howard</fd>
        <v>1867</v>
        <rn>22</rn>
    </pre>
</stop>
```

```py
>>> from xml.etree.ElementTree import parse, Element
>>> doc = parse('pred.xml')
>>> root = doc.getroot()
>>> root
<Element 'stop' at 0x100770cb0>
>>> root.remove(root.find('sri'))
>>> root.remove(root.find('cr'))
>>> root.getchildren().index(root.find('nm'))
1
>>> e = Element('spam')
>>> e.text = 'This is a test'
>>> root.insert(2, e)
>>> doc.write('newpred.xml', xml_declaration=True)
>>>
```



