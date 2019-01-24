# xlrd和xlwt（Excel读写）

## xlrd模块

Python的三方库xlrd用于对excel文件进行读取，可以是“.xls”或“.xlsx”格式（旧版本可能不支持“.xlsx”）。

**下载安装：**[https://pypi.org/project/xlrd/\#files，或者使用pip安装](https://pypi.org/project/xlrd/#files，或者使用pip安装) “pip install xlrd”

**API文档：**[https://xlrd.readthedocs.io/en/latest/api.html](https://xlrd.readthedocs.io/en/latest/api.html)

##### 常用方法：

* **work\_book = xlrd.open\_workbook\(filename\)：**打开指定路径的excel文件，返回excel处理对象，但无法打开不存在的文件。
* **work\_book.nsheets：**返回excel中的sheet个数。
* **work\_book.sheets\(\)：**加载并返回excel中的所有sheet对象组成的列表。
* **work\_book.sheet\_by\_index\(sheetx\)：**返回对应索引的sheet对象，索引范围为range\(work\_book.nsheets\)。
* **work\_book.sheet\_by\_name\(sheet\_name\)：**返回对应sheet名称的sheet对象。
* **work\_book.sheet\_names\(\)：**返回excel中所有sheet名称组成的列表。
* **sheet.book：**sheet所属的work\_book。
* **sheet.name：**sheet的名称。
* **sheet.nrows：**sheet中的行数。
* **sheet.ncols：**sheet中的列数。
* **sheet.row\(rowx\)：**返回对应行的cell对象组成的列表。
* **sheet.row\_slice\(rowx, start\_colx=0, end\_colx=None\)：**返回对应行的cell对象组成的列表，也自定义切片获取行的cell对象列表。
* **sheet.col\(colx\)：**返回对应列的cell对象组成的列表。
* **sheet.col\_slice\(colx, start\_rowx=0, end\_rowx=None\)：**返回对应列的cell对象组成的列表，也自定义切片获取行的cell对象列表。
* **sheet.cell\(rowx, colx\)：**返回对应单元格的cell对象。
* **sheet.cell\_value\(rowx, colx\)：**返回对应单元格的值。
* **sheet.row\_len\(rowx\)：**返回对应行的有效单元格数。
* **sheet.get\_rows\(\)：**返回一个行的迭代器，每次迭代返回一个cell对象组成的列表，即这一行的cell对象列表。
* **sheet.row\_values\(rowx, start\_colx=0, end\_colx=None\)：**返回对应行的值的列表，也可以自定义切片获取某些值。
* **sheet.col\_values\(colx, start\_rowx=0, end\_rowx=None\)：**返回对应列的值的列表，也可以自定义切片获取某些值。
* **sheet.cell\(rowx, colx\).value：**返回对应单元格的值。

**注：**

* 无论传入的参数，还是获取出来的数据，都是Unicode格式的。
* xlrd的索引都是从0开始的。
* xlrd中还有很多其他方法和属性，可以自行查阅API文档。

## xlwt模块

Python的三方库xlwt用于新建一个excel文件，可以是“.xls”或“.xlsx”格式（旧版本可能不支持“.xlsx”）。

**下载安装：**[https://pypi.org/project/xlwt/\#files](https://pypi.org/project/xlwt/#files) 或者使用pip安装“pip install xlwt”

**常用方法：**

* **work\_book = xlwt.Workbook\(encoding='ascii'\)：**新建一个excel对象（必须使用save方法才能生成最后的excel文件），可以设置编码格式，默认是ASCII，这时候代码中操作excel最好都使用Unicode字符串，特别是有中文的情况下，必须使用Unicode，不然会编码报错；也可以UTF-8（或其他，我没试过），这时候除了最后执行save方法保存时文件路径必须使用Unicode字符串外，其他的操作都可以是普通字符串。
* **work\_book.save\(filename\)：**将excel对象保存为excel文件，filename可以是相对路径，也可以是绝对路径，但是路径中的目录（文件夹）是必须存在的，而且不能存在同名文件，不然会报错，如果路径中包含中文，注意使用Unicode字符串。
* **work\_book.add\_sheet\(sheetname, cell\_overwrite\_ok=False\)：**在work\_book中添加一个指定名称的sheet页，当参数cell\_overwrite\_ok设为True时，sheet中的单元格即便被多次重写也不会报错。
* **sheet.write\(r, c, label='', style=Style.default\_style\)：**在单元格“\(r, c\)”中写入值“label”，可以指定单元格的格式style。
* **sheet.merge\(r1, r2, c1, c2, style=Style.default\_style\)：**合并单元格。
* **sheet.write\_merge\(r1, r2, c1, c2, label="", style=Style.default\_style\)：**合并单元格，并写入值。
* **sheet.row\(indx\)：**获取行对象，可以通过行对象的值来获取和设置行属性，比如设置行高：sheet.row\(0\).height=40。
* **sheet.col\(indx\)：**获取列对象，可以通过列对象的值来获取和设置列属性，比如设置列宽：sheet.col0\).width=40
* **sheet.row\_height\(row\)：**获取行高。
* **sheet.col\_width\(col\)：**获取列宽。
* **xlwt.Formula\(s\)：**s为excel中的公式字符串，可以将这个Formula对象作为write等方法的值传入进去。

##### **xlwt.Formula简单示例：**

```text
# 设置Excel内的超链接，这部分整体作为value传入write等写入方法中，
# 其中的第一个双引号为Excel中的公式表示，不能用单引号或三引号；
# value为写入单元格的值，sheet_name为链接的目的地址，
# col（1,2,3...）和row（A,B,C...）表示连接到sheet_name的单元格位置。
xlwt.Formula('HYPERLINK("#%s!%s%s";"%s")' % (sheet_name, col, row, value))

# 设置Excel外的链接。
xlwt.Formula('HYPERLINK("https://www.baidu.com";"百度")')

# 设置某个单元格的值为“A1*B1”的值。
xlwt.Formula('A1*B1')

# 设置某个单元格的值为“SUM(A1, B1)”的值。
xlwt.Formula('SUM(A1, B1)')
```

##### 设置单元格字体：

```text
cell_font = xlwt.Font() # 字体对象
cell_font.name = 'Times New Roman' # 设置字体
cell_font.bold = True # 粗体
cell_font.underline = True # 下划线
cell_font.italic = True # 斜体
cell_style = xlwt.XFStyle() # 格式对象
cell_style.font = cell_font # 将字体样式赋给格式对象中的字体
sheet.write(1, 0, value, cell_style) # 在单元格写入等方法中将格式参数传进去
```

##### 设置单元格边框：

```text
cell_borders = xlwt.Borders() # 边框对象
cell_borders.left = xlwt.Borders.DASHED # 设置左边框（常用值：NO_LINE（无边框）, THIN（薄）, MEDIUM（中）, THICK（厚）,DASHED（虚线）, DOTTED（点虚线））
cell_borders.right = xlwt.Borders.DASHED
cell_borders.top = xlwt.Borders.DASHED
cell_borders.bottom = xlwt.Borders.DASHED
cell_style = xlwt.XFStyle() # 格式对象
cell_style.borders = cell_borders # 将边框样式赋给格式对象
sheet.write(0, 0, value, cell_style)
```

##### 设置单元格背景色：

```text
cell_pattern = xlwt.Pattern() 
cell_pattern.pattern = xlwt.Pattern.SOLID_PATTERN  # SOLID_PATTERN 或 NO_PATTERN
cell_pattern.pattern_fore_colour = 5 # 颜色（不止这些）：0 = Black, 1 = White, 2 = Red, 3 = Green, 4 = Blue, 5 = Yellow, 6 = Magenta, 7 = Cyan, 16 = Maroon, 17 = Dark Green, 18 = Dark Blue, 19 = Dark Yellow , 20 = Dark Magenta, 21 = Teal, 22 = Light Gray, 23 = Dark Gray
cell_style = xlwt.XFStyle() 
cell_style.pattern = cell_pattern 
sheet.write(0, 0, value, cell_style)
```



