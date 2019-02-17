# wxPython（GUI图形用户界面）

wxPython是一套基于Python的第三方GUI插件，可用Python制作丰富的图形化界面程序。

**安装：**pip install wxPython 或者 网站下载安装[https://pypi.org/project/wxPython/\#files](https://pypi.org/project/wxPython/#files)

##### 

##### wxPython demo：

**运行demo：**直接cd到包路径，然后使用“python demo.py”或者“pythonw demo.pyw”。

wxPython中大部分控件的使用效果都可以在demo程序中看到，所以如果想要找某个效果的控件，就可以去demo程序中去找，第一次接触wxPython建议将demo程序的全部展示看一遍，以便以后用的时候方便找。另外，想要看demo的源代码，除了在demo程序中看，也可以使用PyCharm等工具直接打开对应文件包即可查看整个demo程序的源代码。

##### 

##### wxPython docs：

运行docs：直接使用浏览区打开文件夹中的index.html，然后就可以查找想要的文档信息了。

wxPython中控件的初始化方法、事件和各种方法在接口文档中都有详细的说明，或者你不知道某个控件有哪些方法时，也不妨去看看接口文档。

**demo和docs下载：**[https://extras.wxpython.org/wxPython4/extras/](https://extras.wxpython.org/wxPython4/extras/)

![](/assets/demo-docs.png)

##### 电子书推荐：

demo和docs中的说明虽然已经很丰富了，但是某些控件的demo代码并不是太直观，docs中的说明也是全英文的，所以在实际开发的时候还是会有很多不明白的问题，这些问题除了网上搜答案之外，推荐一本电子书**《wxPython实战\(中文版）高清.pdf》**，这本电子书虽然在wxPython版本上有差异，但是对于控件的使用说明可以说非常详细了，同时还有独立的demo代码，对学习和使用wxPython非常有帮助。

## wxPython使用

wxPython中的一些基础控件使用在demo和docs中都可以很容易找到，这里就写一些自己的使用总结和经验分享，涉及到的控件的构造函数、方法和事件等可自行查阅API文档或者相关书籍（如《wxPython实战\(中文版）高清.pdf》）。

##### 简单demo

![](/assets/simple demo.png)

```py
import wx


# 一般GUI程序的最外层框架使用wx.Frame
class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='StaticBoxSizer Test')
        # 新建一个panel，作为主面板
        self.panel = wx.Panel(self, size=(250, 50))
        # 新建两个按钮，用于弹出消息和关闭程序
        self.btn_hello = wx.Button(self.panel, label='Hello World')
        self.btn_exit = wx.Button(self.panel, label='Exit')
        # 给按钮事件绑定按钮事件方法，不同的控件都有对应的事件，可查阅API文档DOCS
        self.btn_hello.Bind(wx.EVT_BUTTON, self.on_helloworld)
        self.btn_exit.Bind(wx.EVT_BUTTON, self.on_exit)

        # 新建一个sizer用于界面布局，界面布局最好不使用pos参数写死，不然以后不好维护
        self.box_sizer = wx.BoxSizer()
        self.box_sizer.Add(self.btn_hello, 0, wx.ALL, 10)
        self.box_sizer.Add(self.btn_exit, 0, wx.ALL, 10)
        self.panel.SetSizer(self.box_sizer)

        # 界面设计好后，如果使用到了sizer布局控件，一般需要使用Layout重新绘制界面
        self.box_sizer.Layout()
        self.panel.Layout()
        self.Layout()

        # Fit方法使框架自适应内部控件
        self.Fit()

    # 事件方法，的第二个参数evt或者event一定不要忘了，不然这个方法会报错
    # 类中可能有很多个方法，事件方法建议约定一个容易识别的命名方式，比如我这里是以“on_”开头
    def on_helloworld(self, event):
        """弹出消息：Welcome to WxPython!"""
        # 新建一个消息弹窗，用于显示消息
        wx.MessageBox('Welcome to WxPython!')

    def on_exit(self, evt):
        """退出程序"""
        wx.Exit()


if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

##### 进度条对话框

**弹窗式进度条：**wx.ProgressDialog

![](/assets/progressdialog.png)

wx.ProgressDialog为进度条显示弹窗，它的唯一的方法Update\(value,newmsg=””\)，value参数是进度条的新的内部的值，调用update将导致进度条根据新的计数值与最大计算值的比例重绘。如果使用可选的参数newmsg，那么进度条上的文本消息将变为该字符串，这让你可以给用户一个关于当前进度的文本描述。Update方法返回两个参数“continue”和“skip”，分别默认返回“True”和“False”，在Update时，若用户点击过“Cancel”按钮（如果有）或者关闭了对话框，“continue”就返回的是“False”（可用于中断进度条），而“skip”只有点击了“Cancel”按钮后才会返回“True”。

##### GUI程序布局

虽然控件本身也可以使用pos参数来指定控件的位置，但是推荐使用wxpython的布局控件来进行布局，布局控件不仅能更好的展示布局效果，而且也方便日后的维护工作。

由于在demo和docs中不好直接看到布局控件的效果，以下介绍一些常用布局控件用法和效果：

** 简单网格布局控件：**wx.GridSizer

![](/assets/gridsizer.png)

```py
import wx


class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='GridSizer Test', size=(300, 400))
        # 创建一个wx.GridSizer, 3行3列，行间隔5， 列间隔5
        grid_sizer = wx.GridSizer(3, 3, 5, 5)
        for i in range(9):
            panel = wx.Panel(self)
            panel.SetMinSize((100, 100))
            # 从左到右，从上到下，一次将控件wx.Panel添加到wx.GridSizer中
            grid_sizer.Add(panel)
        # 将wx.GridSizer与框架控件wx.Frame关联起来
        self.SetSizer(grid_sizer)
        # 使框架自适应内部控件
        self.Fit()


if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

wx.GridSizer为简单网格布局控件，初始化时需要指定行数和列数，以及行和列之间的间隔。特点是每个“格子”的尺寸大小都是一样的，比较整齐，这里的格子指的是布局控件给定的空间，而不是放在格子中的控件的大小。

当窗口大小发生变化时，默认所有的格子都会随着发生变化。

如果格子中的控件的大小不一致时，则会以所有控件中最大的“长”和“宽”作为每个格子的尺寸大小。

使用Add方法在网格中添加控件时，只能按顺序添加控件。

**更加灵活的网格布局控件：**wx.FlexGridSizer

![](/assets/flexgridsizer.png)

```py
import wx


class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='FlexGridSizer Test', size=(300, 400))
        # 创建一个wx.FlexGridSizer, 3行3列，行间隔5， 列间隔5
        grid_sizer = wx.FlexGridSizer(3, 3, 5, 5)
        for i in range(9):
            panel = wx.Panel(self)
            if i == 4:
                panel.SetMinSize((200, 100))
            else:
                panel.SetMinSize((100, 100))
            # 从左到右，从上到下，一次将控件wx.Panel添加到wx.FlexGridSizer
            grid_sizer.Add(panel)

        # 指定第1列是可扩展的
        grid_sizer.AddGrowableRow(1)

        # 将wx.FlexGridSizer.Frame关联起来
        self.SetSizer(grid_sizer)
        # 使框架自适应内部控件
        self.Fit()



if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

wx.FlexGridSizer布局控件，相比于wx.GridSizer，更加灵活的地方在于，可以在某行或者某列上进行设置，即每一行或者每一列它的尺寸大小是独立的。

每一行（列）会以这一行（列）的最大的“长”（“宽”）作为这一行（列）的尺寸大小。

当窗口大小发生变化时，默认整个wx.FlexGridSizer是固定不变的，除非使用wx.FlexGridSizer的方法AddGrowableRow（AddGrowableCol）指定某行（列）是可以跟着窗口大小的扩展而扩展。

使用Add方法在网格中添加控件时，只能按顺序添加控件。

**进阶的网格布局控件：**wx.GridBagSizer

![](/assets/gridbagsizer.png)

```py
import wx


class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='GridBagSizer Test', size=(300, 400))
        # 创建一个wx.GridBagSizer, 行间隔5， 列间隔5
        grid_sizer = wx.GridBagSizer(5, 5)
        for row in range(3):
            for col in range(3):
                panel = wx.Panel(self)
                panel.SetMinSize((100, 100))
                # wx.GridBagSizer的Add方法需要指定添加的位置
                grid_sizer.Add(panel, pos=(row, col))

        # 在第0行第3列，添加一个控件，占3行和1列的空间，wx.EXPAND表示控件扩展至填满整个“格子”的空间
        panel_right = wx.Panel(self)
        panel_right.SetMinSize((100, 100))
        grid_sizer.Add(panel_right, pos=(0, 3), span=(3, 1), flag=wx.EXPAND)

        # 在第3行第0列，添加一个控件，占1行和4列的空间，wx.EXPAND表示控件扩展至填满整个“格子”的空间
        panel_bottom = wx.Panel(self)
        panel_bottom.SetMinSize((100, 100))
        grid_sizer.Add(panel_bottom, pos=(3, 0), span=(1, 4), flag=wx.EXPAND)

        # 将wx.GridBagSizer.Frame关联起来
        self.SetSizer(grid_sizer)
        # 使框架自适应内部控件
        self.Fit()



if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

wx.GridBagSizer布局控件，相比于wx.FlexGridSizer和wx.GridSizer，多了两个特性，一是可以初始化时不用指定“格子”数量，在往“格子”中添加控件时可以不按顺序添加，直接指定控件添加的位置即可，二是一个“格子”可以跨越多个单元格显示。

当窗口大小变化时，与wx.FlexGridSizer一样，默认是不变的，需要自己指定哪行（列）是可扩展的。

此控件在使用Add方法时，必须指明添加的位置，即pos参数。

**横向（纵向）布局控件：**wx.BoxSizer

![](/assets/boxsizer_h.png)

![](/assets/boxsizer_v.png)

```py
import wx


class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='BoxSizer Test', size=(300, 400))
        # 创建一个wx.BoxSizer, 默认是wx.HORIZONTAL，即横向的
        box_sizer = wx.BoxSizer()
        # 也可指定为wx.VERTICAL，即纵向的
        # grid_sizer = wx.BoxSizer(wx.VERTICAL)

        for row in range(3):
            panel = wx.Panel(self)
            panel.SetMinSize((100, 100))
            # 从左到右（从上到下）依次添加控件
            # 第二个参数为0时，窗口扩展时，格子尺寸不会变化，不为0时会跟着扩展变化
            # wx.ALL表示所有边框（此处边框为5）
            # wx.EXPAND表示控件填满整个“格子”
            box_sizer.Add(panel, 1, wx.ALL | wx.EXPAND, 5)

        # 将wx.BoxSizer.Frame关联起来
        self.SetSizer(box_sizer)
        # 使框架自适应内部控件
        self.Fit()



if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

wx.BoxSizer布局控件只能横向或者二纵向添加控件，默认是横向的，可以使用wx.BoxSizer\(wx.VERTICAL\)使之变为纵向的，并且没有特定的“格子”数，可以一直添加控件。

在窗口大小发生变化时，wx.BoxSizer只能横向或者纵向扩展。

在使用Add方法添加控件时，如果第二个参数proportion指定为0（默认），则“格子”在窗口大小变化时会保持不变，如果需要格子跟着扩展，一般可以指定为1。

**带有标签和边框线的横向（纵向布局控件）：**wx.StaticBoxSizer

![](/assets/staticboxsizer_h.png)

![](/assets/staticboxsizer_v.png)

```py
import wx

class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, -1, title='StaticBoxSizer Test', size=(500, 700))
        # 新建一个wx.StaticBox（带有标签和边框线）
        staticbox = wx.StaticBox(self, label="StaticBoxSizer Test")
        # wx.StaticBoxSizer初始化时需要传入一个wx.StaticBox，方向默认是横向
        staticbox_sizer = wx.StaticBoxSizer(staticbox)
        # 也可指定为wx.VERTICAL，即纵向的
        # staticbox_sizer = wx.StaticBoxSizer(staticbox, wx.VERTICAL)

        for row in range(3):
            panel = wx.Panel(self)
            panel.SetMinSize((100, 100))
            # 从左到右（从上到下）依次添加控件
            # 第二个参数为0时，窗口扩展时，格子尺寸不会变化，不为0时会跟着扩展变化
            # wx.ALL表示所有边框（此处边框为5）
            # wx.EXPAND表示控件填满整个“格子”
            staticbox_sizer.Add(panel, 1, wx.ALL | wx.EXPAND, 5)

        # 这里单独在新建一个wx.BoxSizer是为了在界面中显示出wx.StaticBoxSizer的边框线
        box_sizer = wx.BoxSizer()
        box_sizer.Add(staticbox_sizer, 1, wx.ALL | wx.EXPAND, 5)

        # 将wx.BoxSizer.Frame关联起来
        self.SetSizer(box_sizer)
        # 使框架自适应内部控件
        self.Fit()


if __name__ == '__main__':
    app = wx.App()
    myframe = MyFrame()
    myframe.Show()
    app.MainLoop()
```

wx.StaticBoxSizer布局控件其实就是相当于在wx.BoxSizer外面套了一个wx.StaticBox，用法与wx.BoxSizer是相同的。

