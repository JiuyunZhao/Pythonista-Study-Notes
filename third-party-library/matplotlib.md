# Matplotlib（数据可视化）

Matplotlib是一个可以将数据绘制为图形表示的Python三方库，包括线性图（折线图，函数图）、柱形图、饼图等基础而直观的图形，在平常的开发当中需要绘图时就非常有用了。

**安装：**pip install matplotlib或者下载安装[https://pypi.org/project/matplotlib/\#files](https://pypi.org/project/matplotlib/#files)

**demo效果图：**[https://matplotlib.org/gallery.html，这里有许多效果图，点击对应的图片就能看到源码和生成的图形，画图时可以看看这里有没有自己想要的效果图。](https://matplotlib.org/gallery.html，这里有许多效果图，点击对应的图片就能看到源码和生成的图形，画图时可以看看这里有没有自己想要的效果图。)

**API文档：**Matplotlib的代码是子文档的，可以直接在源代码中查看相关接口的文档说明，或者使用Python的help\(\)来进行查看。

**Matplotlib教程：**[https://liam.page/2014/09/11/matplotlib-tutorial-zh-cn/，这篇笔记也是根据这个教程整理的，里面介绍的更加详细，也有效果图和对应代码。](https://liam.page/2014/09/11/matplotlib-tutorial-zh-cn/，这篇笔记也是根据这个教程整理的，里面介绍的更加详细，也有效果图和对应代码。)

#### 绘图接口

**import matplotlib：**普通的绘图使用from matplotlib import pyplot就行了，绘图接口都在pyplot中，numpy也在pyplot中，使用from matplotlib.pyplot import np就行了（np就是numpy的别名，查看源码可以看到：import numpy as np）。

**import pylab：**这个接口的语法和绘图命令和Matlab相近，熟悉Matlab的可以选择使用这个接口，numpy也可以直接从pylab中导入：from pylab import np（np就是numpy的别名，查看源码可以看到：import numpy as np）。

**numpy：**这是一个用于数组运算的Python三方库，安装Matplotlib时会默认安装这个库，不会使用这个库也没关系，绘图时直接使用Python的列表代替就行了，只是说使用numpy在某些情况下会方便些，比如绘制函数图时。

#### 线性图

线性图其实就是根据一系列X轴的点（列表数据）和对应的Y轴的点（列表数据），在图上绘制出对应的点，每个点之间用线（实线、虚线等，默认是实线）连接起来，就形成了线性图，所以无论是折线图或者函数图使用的接口都是一样的。

##### 简单示例：

![](/assets/lines_chart.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 第一条线：一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x1, y1, color='red', linewidth=3, linestyle='--')

# 第二条线：一条自定义的函数图
x2 = np.array([1, 2, 3])
# y2其实就是x2的数组中每个元素进行运算（+0.5）后返回的新数组
y2 = x2 + 0.5
plt.plot(x2, y2)

# 第三条线：一条sin函数图
# linspace返回0到pi之间等间隔的200值组成的数组
x3 = np.linspace(0, np.pi, 200)
# 返回x3中每个元素的sin值组成的数组
y3 = np.sin(x3)
plt.plot(x3, y3)

# 绘制并展示图形
plt.show()
```

#### 条形图

条形图与线性图类似，也是根据x轴的点和y轴的点来进行绘制的，这时候y轴的点就代表了条形图的高度了。

##### 简单示例：

![](/assets/bars_chart.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# arange和Python的range用法相同
x = np.arange(5)
# uniform返回0到5之间（不包含5）的5个随机数
y = np.random.uniform(0, 5, 5)

# 绘制条形图，facecolor设置背景色，edgecolor设置边框颜色
plt.bar(x, y, facecolor='#9999ff', edgecolor='grey')

# 将每个对应的y值显示在条形图上方
for x_, y_ in zip(x, y):
    plt.text(x_, y_, '{:.2f}'.format(y_), ha='center', va='bottom')

# 绘制和显示图形
plt.show()
```

#### 饼状图

饼图的绘制需要给出一个数字的数组，如果数组中的数字的和小于1，那么饼图中的每个区域就会以数组中的元素的数值大小来进行绘图，不足1的那部分就会留作空白；如果数组中的数字的和大于等于1，那么就以每个元素的在数组和的占比来进行绘图。

##### 简单示例：

![](/assets/pie_chart.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 如果数组的和小于1，则以元素自身大小作为饼图占比，剩下的部分留作空白不绘制
# 如果数组的和大于等于1，则以元素在数组和的占比作为饼图占比
areas = np.array([0.1, 0.4, 0.2, 0.1, 0.2])
# 每个区域距离相邻区域的距离
explode = np.array([0, 0.1, 0, 0, 0])
# 在每个区域边上的文本
labels = np.array(['one', 'two', 'three', 'four', 'five'])
# autopct是在区域中显示的各自百分比的格式
# startangle表示开始绘制的旋转角度，90表示逆时针旋转90度
plt.pie(areas, explode=explode, labels=labels, autopct='%1.1f%%', startangle=90)

plt.show()
```

#### 设置X轴和Y轴范围

一般而言，X轴和Y轴的范围会自动判断和生成，但是因为X轴和Y轴的交点默认是各自的下限，并不是从0开始的，最后的绘制结果可能并不满足自己的需求，所以可以自行设置，使之更加的符合自己的需求。

##### 简单示例：

![](/assets/xylim.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 一条普通的折线图
x = np.array([1, 2, 3])
y = np.array([1, 2, 0.5])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x, y, color='red', linewidth=1, linestyle='--')

# 设置X轴的范围，上限和下限
plt.xlim(0, 5)
# 设置Y中的范围，上限和下限
plt.ylim(-1, 5)

# 绘制并展示图形
plt.show()
```

#### 设置坐标轴记号和标签

坐标轴默认的刻度记号显示很多时候并不符合我们自己的需求，甚至有时候我们需要在指定的刻度记号处显示特定的文本 ，但是如果自定义了显示刻度记号，那图上就只会显示指定的记号，其他的默认刻度记号就不会再显示了。

##### 简单示例：

![](/assets/xyticks.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 一条普通的折线图
x = np.array([1, 2, 3])
y = np.array([1, 2, 0.5])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x, y, color='red', linewidth=1, linestyle='--')

# 设置X轴的范围，上限和下限
plt.xlim(0, 5)
# 设置Y中的范围，上限和下限
plt.ylim(-1, 5)

# 可以只设置记号，即只传入第一个列表，且第一个列表的元素需要是在对应坐标轴的范围内的值
# 如果传入第二个列表，则第二个列表的元素内容（标签）会替换第一个列表对应位置的记号
# 设置X轴的记号和标签
plt.xticks([1, 3, 4, 5],
           [r'$x1$', r'$x3$', r'$x4$', r'$x5$'])
# 设置Y轴的记号和标签
plt.yticks([1, 2, 3, 5],
           [r'$y1$', r'$y2$', r'$y3$', r'$y5$'])

# 绘制并展示图形
plt.show()
```

#### 添加第二个Y轴

有时候在绘制多个图形时，不同的图形使用的Y轴刻度不同，或者使用两种不同的刻度显示会更加直观，这时候就可以左边的Y轴使用一种刻度，右边也添加一条Y轴，用于显示另一种刻度，但注意的是它们共享X轴的刻度。

##### 简单示例：

![](/assets/twinx.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 第一条线：一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x1, y1, color='red', linewidth=3, linestyle='--')

# 创建另一个坐标轴，此坐标轴与之前的坐标轴共用X轴，并将Y轴置于右边
plt.twinx()

# 第二条线：一条sin函数图
# linspace返回0到pi之间等间隔的200值组成的数组
x2 = np.linspace(0, np.pi, 200)
# 返回x2中每个元素的sin值组成的数组
y2 = np.sin(x2)
plt.plot(x2, y2)

# 绘制并展示图形
plt.show()
```

#### 移动坐标轴

横纵坐标轴的交点位置默认在左下角，并且起点都是在各自的下限，但是有些时候我们希望交点位置在横纵坐标的0点位置，也就是原点，这时候就可以移动坐标轴到我们想要的位置了。

##### 简单示例：

![](/assets/xyaxis.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 一条普通的折线图
x = np.array([1, 2, 3])
y = np.array([1, 2, 0.5])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x, y, color='red', linewidth=1, linestyle='--')

# 设置X轴的范围，上限和下限
plt.xlim(-2, 5)
# 设置Y中的范围，上限和下限
plt.ylim(-2, 5)

# 获取当前的坐标轴对象
ax = plt.gca()
# 设置右边框和上边框为无色
ax.spines['right'].set_color('none')
ax.spines['top'].set_color('none')
# 设置X轴的位置为下边框的位置
ax.xaxis.set_ticks_position('bottom')
# 设置下边框的位置在Y轴0的位置，data表示Y轴数据，0为Y轴上的数据值
ax.spines['bottom'].set_position(('data', 0))
# 设置Y轴的位置为左边框的位置
ax.yaxis.set_ticks_position('left')
# 设置左边框的位置在X轴0的位置，data表示X轴数据，0为X轴上的数据值
ax.spines['left'].set_position(('data', 0))

# 绘制并展示图形
plt.show()
```

#### 添加图例

一般画图时，都需要为对应的图形设置图例，标明对应图形的含义，不然图形就不够清晰明了了。

##### 简单示例：

![](/assets/legends.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 第一条线：一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x1, y1, label='first line', color='red', linewidth=3, linestyle='--')

# 第二条线：一条自定义的函数图
x2 = np.array([1, 2, 3])
# y2其实就是x2的数组中每个元素进行运算（+0.5）后返回的新数组
y2 = x2 + 0.5
plt.plot(x2, y2, label='second line')

# 第三条线：一条sin函数图
# linspace返回0到pi之间等间隔的200值组成的数组
x3 = np.linspace(0, np.pi, 200)
# 返回x3中每个元素的sin值组成的数组
y3 = np.sin(x3)
plt.plot(x3, y3, label='third line')


# 设置图例，upper left表示左上角，loc参数的选项有：
# best
# upper right
# upper left
# lower left
# lower right
# right
# center left
# center right
# lower center
# upper center
# center
# 如果画图时没有设置label参数或者不想使用label参数，可以给图例重新设置label
# 重新设置时，legend第一个参数传入图形列表，二个参数传入对应的label列表，第三个参数就是loc了
plt.legend(loc='upper left')


# 绘制并展示图形
plt.show()
```

#### 设置注释

绘图时，可能会需要给某些特殊的点注明一些说明性的注释，以便更好的理解和分析图形。

##### 简单示例

![](/assets/annotate.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 一条自定义的函数图
x = np.array([1, 2, 3])
# y2其实就是x2的数组中每个元素进行运算（+0.5）后返回的新数组
y = x + 0.5
plt.plot(x, y, label='first line')

# 在指定位置显示一个点
x_point = 2
y_point = x_point + 0.5
# scatter用于绘制散点图，第一个参数和第二个参数列表中只有一个元素时自然就只画一个点了
# 第三个参数用于指定绘制“点”的半径
plt.scatter([x_point, ], [y_point, ], 10, color='red')
# 给指定的点设置注释
plt.annotate(r'2.5=2+0.5',  # 注释文本
             xy=(x_point, y_point),  # 点的位置
             xycoords='data',
             xytext=(90, -50),  # 以要注释的点为原点，注释文本的坐标位置（单位为像素）
             textcoords='offset points',
             arrowprops=dict(arrowstyle='->', connectionstyle='arc3, rad=.5'))  # arc为直线，arc3为圆形，rad为半径

# 绘制并展示图形
plt.show()
```

#### 绘制子图

有时候我们可能需要在一张图（figure）上同时绘制几个或几张图形（subplot），以便更好的观察和分析。

如果绘制图形时没有手动创建figure和子图，Matplotlib自动创建一个figure和一个子图，所有绘制出来的图形直观效果就是多个图形都在同一个“图片”上，且共用一个坐标轴。

##### 多个figure，简单示例：

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 第一个figure
# num用于指定此figure的ID（int类型）或者指定此figure的标题
# figsize用于指定此figure的尺寸大小
plt.figure(num='My Figure Line', figsize=(6, 6))

# 一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x1, y1, label='first line', color='red', linewidth=3, linestyle='--')

# 第二个figure
# num用于指定此figure的ID（int类型）或者指定此figure的标题
# figsize用于指定此figure的尺寸大小
plt.figure(num=2, figsize=(5, 5))

# 一条sin函数图
# linspace返回0到pi之间等间隔的200值组成的数组
x2 = np.linspace(0, np.pi, 200)
# 返回x2中每个元素的sin值组成的数组
y2 = np.sin(x2)
plt.plot(x2, y2, label='first line')

plt.show()
```

##### 同一个figure中绘制多个子图，简单示例：

![](/assets/subplots.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 创建一个figure，且创建一个2行1列的子图网格，返回figure对象和子图数组
# 网格默认为一行一列，此时返回子图本身，如果网格只有一行或者一列，那就返回一个一维子图数组，如果有多行多列，就返回对应的二维子图数组
# subplots中传入的参数可以是figure的参数
fig, axes = plt.subplots(2, 1, num='My Figure Demo')

# 在第一个子图中绘制一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
axes[0].plot(x1, y1, label='first line', color='red', linewidth=3, linestyle='--')

# 显示图例
axes[0].legend()

# 在第二个子图中绘制一条sin函数图
# linspace返回0到pi之间等间隔的200值组成的数组
x2 = np.linspace(0, np.pi, 200)
# 返回x2中每个元素的sin值组成的数组
y2 = np.sin(x2)
axes[1].plot(x2, y2, label='second line')
# 显示图例
axes[1].legend()

# 绘制并显示
plt.show()
```

##### 跨行或跨列绘制子图，简单示例：

![](/assets/gridsubplot.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt
from matplotlib import gridspec

fig = plt.figure('My Figure Demo')
grid = gridspec.GridSpec(2, 2, figure=fig)

# 在第一行绘制一条普通的折线图，占据一行的空间
axe1 = plt.subplot(grid[0, :])
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
axe1.plot(x1, y1, label='first line', color='red', linewidth=3, linestyle='--')

# 显示图例
axe1.legend()

# 在第二行第一个子图中绘制一条sin函数图，在这里，以下三种表达都是相同的效果
# axe2 = plt.subplot(grid[1, :1])
# axe2 = plt.subplot(grid[1, :-1])
axe2 = plt.subplot(grid[1, 0])
# linspace返回0到pi之间等间隔的200值组成的数组
x2 = np.linspace(0, np.pi, 200)
# 返回x2中每个元素的sin值组成的数组
y2 = np.sin(x2)
axe2.plot(x2, y2, label='second line')

# 显示图例
axe2.legend()

# 绘制并显示
plt.show()
```



#### 显示中文

Matplotlib默认是不只支持中文的，但是可以使用自带的中文字体。

##### 简单示例：

![](/assets/display_cn.png)

```py
# -*- coding:utf-8 -*-
import numpy as np
from matplotlib import pyplot as plt

# 打印自带的字体，可以从里面选一种中文字体
# from matplotlib import font_manager
# for f in font_manager.fontManager.ttflist:
#     print(f.name)

# 设置字体为SimHei（黑体）
plt.rcParams['font.family'] = ['SimHei']


# 一条普通的折线图
x1 = np.array([1, 2, 3])
y1 = np.array([1, 2, 3])
# linewidth设置线宽，单位为像素，linestyle默认为实线，“--”表示虚线
plt.plot(x1, y1, label='一条普通的折线图', color='red', linewidth=3, linestyle='--')

# 显示图例
plt.legend(loc='upper left')

# 绘制并显示
plt.show()
```



