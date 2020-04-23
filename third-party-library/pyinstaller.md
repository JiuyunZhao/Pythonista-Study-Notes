# PyInstaller（exe程序打包）

PyInstaller可以将Python程序打包成一个exe程序来独立运行，用户使用时只需要执行这个exe文件即可，不需要在机器上再安装Python及其他包就可运行了。另外，PyInstaller相较于其他打包程序，比如py2exe，大多时候使用起来更加方便，可以通过命令行的一些简单命令即可进行打包，当然，当你需要打包的程序比较大且复杂时，使用哪个打包程序可能差别都不会太大了，这时候就看个人的习惯和爱好了。

**pip安装PyInstaller：**pip install pyinstaller

**pip更新PyInstaller：**pip install --upgrade pyinstaller

**pypi下载PyInstaller：**https://pypi.org/project/PyInstaller/\#files

**注意：**windows安装PyInstaller时需要额外安装另外两个模块，pywin32或pypiwin32，以及pefile，如果使用pip进行安装，但是还没有安装pywin32时，会自动安装pypiwin32，pefile没有时也会自动安装。



#### 简单打包命令

##### pyinstaller \[-F/-D\] \[-w/-c\] \[-i xxx.ico\] xxx.py/xxx.spec

* **xxx.py/xxx.spec：**需要打包的程序main文件或者spec文件。spec文件在使用py文件进行打包时会在相同路径下自动生成，spec中的内容也是根据命令行中输入的内容来生成的，也可以使用命令pyi-makespec \[options\] xxx.py来生成一个纯粹的spec文件，而不会去执行打包的操作。
* **-F/--onefile：**将整个程序打包为一个exe文件，需要注意的是，与-D模式生成的exe程序相比，在启动速度上会慢一点，原因是它需要先解压exe文件并生成一个唯一的临时环境来运行程序，关闭环境时也会自动删除这个临时环境，-D模式的程序本身就是解压好的，运行完也不需要执行删除操作，当程序比较大时，这个差别就很明显了。
* **-D/--onedir：**默认选项，与F/--onefile参数作用相反，将程序打包为一个文件夹，文件夹中包含启动程序的exe文件和其他依赖的资源文件和DLL文件等。
* **-w：**表示程序运行后隐藏命令行窗口，当你不需要使用命令行窗口作为程序的I/O时，比如GUI程序，可以使用这个参数选项。
* **-c：**默认选项，与-w相反，提供一个命令行窗口进行I/O。
* **-i/--icon：**指定exe程序图标。



一般来说，大多数的程序，特别是一些小程序，使用这个命令就可以顺利的打包了，也用不到其他复杂的参数选项。如果需要其他的知识点，以下的一些点可以参考下（这些点是自己在官方文档上找的，有些并没有经过验证，有错欢迎指正）：

#### 命令行指定参数选项方式打包

1. **build文件夹：**运行后会在同路径下生成一个build文件夹，这个文件夹的作用相当于PyInstaller的工作空间，PyInstaller运行相关的文件和日志都在这个文件夹中，打包完成后可以直接删除。
2. **dist文件夹：**运行完成后会在同路径下生成一个dist文件夹，这个文件夹下有一个跟程序同名的文件夹，打包好的exe程序就在这个文件夹下。
3. **多py文件：**如果命令行中指定的py文件不止一个，比如“pyinstaller xxx1.py xxx2.py”，pyinstaller会依次分析并执行，并把第一个py名称作为spec和dist文件下的文件夹和程序的名称。
4. **其他常用参数选项：**
   * **--specpath DIR：**指定生成spec文件的路径，默认为当前路径。
   * **-n NAME/--name NAME：**指定spec文件和程序的名称，默认就是传入的py脚本或spec文件的名称。
   * **-h/--help：**显示PyInstaller帮助信息。
   * **-v/--version：**显示PyInstaller版本信息。
   * **--distpath DIR：**指定生成dist的目录，默认为“./dist”。
   * **--workpath WORKPATH：**指定pyinstaller的工作目录，即build文件夹，默认为“./build”。
   * **-y/--noconfirm：**替换输出目录时不询问，默认输出目录是“SPECPATH/dist/SPECNAME”。
   * **--upx-dir UPX\_DIR：**指定UPX程序的路径，默认为“程序执行路径”，即双击某个文件时，系统自动查找对应程序的路径。UPX为一个压缩程序，需要自行下载，可以将exe压缩为zip格式的文件，并且压缩效率非常高，如果打包后的exe程序非常大的话，为了避免客户下载时文件太大的问题，可以使用这个UPX工具。
   * **--noupx：**不需要UPX（即便可用）。
   * **-a/--ascii：**不支持Unicode，默认为支持（如果可用的话）。
   * **--clean：**在pyinstaller开始执行之前清除缓存并删除临时文件（一般存储在C:\Users\Administrator\AppData\Roaming\pyinstaller）。
   * **--log-level LEVEL：**指定打印的日志等级，默认为INFO，日志等级有：TRACE，DEBUG，INFO，WARN，ERROR，CRITICAL。如果在打包时遇到了问题，为了方便定位问题，可以使用这个参数来查看特定级别的日志信息。
5. **数据绑定和搜索相关的参数选项：**
   * **--add-data SRC;DEST：**指定需要添加非二进制文件路径或者文件夹路径，比如图片和pdf文件等，这个选项可以使用多次。这个命令其实就是将需要的文件或者文件夹拷贝到指定的路径下，在-D模式下，可以看情况在程序打包完成后自己手动拷贝过去。
   * **--add-binary SRC;DEST：**指定需要添加的二进制文件路径，比如DLL文件、动态链接库或者共享文件对象等，这个选项可以使用多次。同-add-data命令一样，是一个拷贝数据的功能。
   * **-p DIR/--paths DIR：**指定import语句的查找路径（与PYTHONPATH一样），多个路径之间可以使用分号“;”连接，或者多次使用这个选项来进行指定。
   * **--hidden-import MODULENAME/--hiddenimport MODULENAME：**指定脚本中需要隐式导入的模块，比如在\_\_import\_\_、imp.find\_module\(\)、exec、eval等语句中导入的模块，这些模块PyInstaller是找不到的，需要手动指定导入，这个选项可以使用多次。
   * **--additional-hooks-dir HOOKSPATH：**指定额外hook文件（可以是py文件）的查找路径，这些文件的作用是在PyInstaller运行时改变一些Python或者其他库原有的函数或者变量的执行逻辑（并不会改变这些库本身的代码），以便能顺利的打包完成，这个选项可以使用多次。
   * **--runtime-hook RUNTIME\_HOOKS：**指定自定义的运行时hook文件路径（可以是py文件），在打好包的exe程序中，在运行这个exe程序时，指定的hook文件会在所有代码和模块之前运行，包括main文件，以满足一些运行环境的特殊要求，这个选项可以使用多次。
   * **--exclude-module EXCLUDES：**指定可以被忽略的可选的模块或包，因为某些模块只是PyInstaller根据自身的逻辑去查找的，这些模块对于exe程序本身并没有用到，但是在日志中还是会提示“module not found”，这种日志可以不用管，或者使用这个参数选项来指定不用导入，这个选项可以使用多次。
   * **--key KEY：**指定用于Python字节码加密的key，key是一个16个字符的字符串。
6. **-D模式程序的运行：**运行程序的时候，其实开始运行的是一个pyinstaller生成的引导加载程序bootloader，bootloader是根据不同的操作系统生成的，运行bootloader时，会创建一个临时的Python环境以便运行Python程序，所以使用exe程序时不用安装Python也能运行这个程序。
7. **-F模式程序的运行：**也会有一个BootLoader，但是会根据操作系统创建一个名为\_MEIxxxxxx的文件夹，用作这个程序的临时运行环境（不只是Python环境），这个xxxxxx是一个随机的数字。-F模式程序启动的时候因为需要解压并拷贝依赖和资源文件到临时运行环境\_MEIxxxxxx，所以启动速度是比-D模式程序要慢的，运行结束后会删除临时运行环境的文件夹。在Linux和相关系统中，可能有“no-execution”选项，但是对于-F模式程序是不兼容的。由于\_MEIxxxxxx是唯一的，所以可以同时运行多个程序，多个程序时互不干涉的。如果程序崩溃了，或者强行结束了（比如在Windows的任务管理器中杀死了进程），\_MEIxxxxxx文件夹是不会被删除的，所以频繁崩溃或者结束进程会导致有多个\_MEIxxxxxx文件夹，会非常占用磁盘空间，可以使用--runtime-temdir指定\_MEIxxxxxx的存放位置。-F模式程序如果在运行时遇到了权限问题，可以使用-D模式程序替代。
8. **shell脚本/批处理脚本：**使用命令行打包时，可能需要指定的参数选项很多，这时候可以将需要执行的全部命令信息，包括这些参数选项的指定，都放在一个shell脚本或者批处理文件中来执行。
9. **运行时信息：**
   * **\_\_file\_\_： **所有基于模块的使用到\_\_file\_\_属性的代码，在源码运行时表示的是当前脚本的绝对路径，但是打包后就是当前模块的模块名（即文件名xxx.py）。
   * **sys.frozen：**源码运行时没有这个属性，打包后的程序添加了这个属性，值为True。
   * **sys.\_MEIPASS：** 源码运行时没有这个属性，打包后的程序添加了这个属性，表示程序运行的绝对路径。对于-D模式程序，表示的是这个exe程序所在文件夹的绝对路径，对于-F模式程序表示的是\_MEIxxxxxx的文件夹绝对路径，\_MEIxxxxxx为exe解压后创建的临时运行环境的文件夹名称，对于exe程序每一次运行来说，它是唯一的。
   * **sys.excutable：**代码运行时表示运行的解释器绝对路径，如C:\Python36\python.exe，在打包的程序中就是exe程序文件的绝对路径，这个是用来定位用户运行该程序的真实位置。
   * **sys.argv\[0\]：**一般来说就是运行程序的绝对路径，但是在不同平台或者不同的方式启动程序时，会有所不同，比如通过符号链接运行的程序sys.argv\[0\]就是符号名称，而不是真实的程序路径。
10. **数据文件的修改：**--add-data这种通过拷贝形式的数据文件，在-D模式下如果在运行时修改了，那么对应的数据文件是真的被修改了，但是在-F模式下，由于每次运行会单独创建一个临时运行环境，修改的内容也是临时运行环境中的内容，并且运行完后会自动删除临时运行环境，所以这种数据文件是无法直接更改exe中的数据文件的，也就意味着每次运行程序，数据文件都会是exe程序中原来的那一份，修改的内容会随着临时运行环境的关闭而删除了，不会同步到exe程序中的。
11. **发生错误时：**
    * 当发生“module not found”警告时，其实很多找不到对应模块的消息都不用管，只是PyInstaller根据自身的逻辑去查找的，因为它们并不是跟你的最终程序有关的。
    * 当发生导入失败时，这才是真正的错误，需要去关注和解决的。
    * 当打包成功，且中间没有发生任何警告提示，但是运行程序时提示某个模块找不到，可能就是“--hidden-import=”的问题了，当使用\_\_import\_\_、imp.find\_module\(\)、exec、eval或者Python/C API时，pyinstaller不会自动去导入这些里面涉及的导包，所以这些包就需要使用“--hidden-import=”来指定具体需要导入的包了。
    *  --runtime-hook=xxx.py中指定的py会依次在打包的主程序main脚本之前运行，可以用来改变一些函数或者变量，以满足特殊场景的要求。

#### 

#### spec配置文件方式打包

1. **生成spec文件：**pyi-makespec \[options\] xxx.py \[other scripts...\]，生成spec文件时可以什么都不指定，然后在生成的spec文件中单独配置，默认为-D模式下的spec文件，也可以指定生成-F模式的spec文件。当然也可以在第一次就将参数选项指定好，以后就只维护spec文件。
2. **参数选项：**生成spec文件的参数选项和命令行模式下执行PyInstaller打包是完全一样的。
3. **spec文件类型：**spec文件其实就是一个py文件，在编辑时可以直接将它当成一个py文件来使用。
4. **spec文件优势：**一般情况而言，直接使用PyInstaller命令行直接打包即可，但是以下情况使用spec文件的话会方便一些：
   * 程序需要绑定一些数据文件，可以在spec文件中单独用一个列表变量来指定，可读性和可维护性会高很多。
   * 需要include一些PyInstaller不知道的动态链接库，如：.dll/.so文件，同样可以在spec文件中单独用一个列表变量来指定。
   * 需要往可执行文件中添加一些运行时选项，如hook文件。

##### 关于spec文件

如果有需要，可以通过PyCharm、eclipse等工具打开安装目录中的PyInstaller文件夹来查看源码信息

* **a：**Analysis类的实例，要求传入各种脚本用于分析程序的导入和依赖。a中内容主要包括以下四部分：scripts，即可以在命令行中输入的Python脚本；pure，程序代码文件中的纯Python模块，包括程序的代码文件本身；binaries，程序代码文件中需要的非Python模块，包括--add-binary参数指定的内容；datas，非二进制文件，包括--add-data参数指定的内容。
* **pyz：**PYZ的实例，是一个.pyz文件，包含了所有pure中的所有Python模块。
* **exe：**EXE类的实例，这个类是用来处理Analysis和PYZ的结果的，也是用来生成最后的exe可执行程序。
* **coll：**COLLECT类的实例，用于创建输出目录。在-F模式下，是没有COLLECT实例的，并且所有的脚本、模块和二进制文件都包含在了最终生成的exe文件中。

1. **Analysis参数scripts：**也是第一个参数，它是一个脚本列表，可以传入多个py脚本，效果与命令行中指定多py文件相同，即py文件不止一个时，比如“pyinstaller xxx1.py xxx2.py”，pyinstaller会依次分析并执行，并把第一个py名称作为spec和dist文件下的文件夹和程序的名称。
2. **Analysis参数pathex：**同命令“-p DIR/--paths DIR”，其实默认就有一个spec的目录，如果使用命令添加的话，会首先添加命令中指定的目录，再添加默认的路径。
3. **Analysis参数datas：**即添加数据文件，命令是--add-data，spec文件中是Analysis的datas=\[\]参数，datas是一个元素为元组的列表，每个元组有两个元素，都必须是字符串类型，元组的第一个元素为数据文件或文件夹，元组的第二个元素为运行时这些文件或文件夹的位置。例如：datas=\[\('src/README.txt', '.'\), \]，也可以在命令行中这样写：pyinstaller --add-data 'src/README.txt;.' myscript.py，表示打包时将文件src/README.txt添加（copy）到相对于spec文件的根目录下，指定文件时是相对于spec来进行寻找的，而不是要打包的exe程序路径。也可以使用通配符：datas= \[ \('/mygame/sfx/\*.mp3', 'sfx' \) \]，表示将/mygame/sfx/目录下的所有.mp3文件都copy到sfx文件夹中。也可以添加整个文件夹：datas= \[ \('/mygame/data', 'data' \) \]，表示将/mygame/data文件夹下所有的文件都copy到data文件夹下。
4. **Analysis参数binaries：**添加二进制文件，效果同命令--add-binary，也是一个列表，定义方式与datas参数一样。
5. **Analysis参数hiddenimports：**同命令“--hidden-import MODULENAME/--hiddenimport MODULENAME”。
6. **Analysis参数hookspath：**同命令“--additional-hooks-dir HOOKSPATH”。
7. **Analysis参数runtime\_hooks：**同命令“--runtime-hook RUNTIME\_HOOKS”。
8. **Analysis参数excludes：**同命令“--exclude-module EXCLUDES”。
9. **exe参数console：**设置是否显示命令行窗口，和命令-w/-c作用一样。
10. **exe参数icon：**设置程序图标，和命令-i/--icon作用一样。某些情况，直接执行“pyinstaller xxx.py”时生成的spec中没有这个参数，需要手动添加，参数值就是图片路径的字符串。
11. **PyInstaller全局变量：**这些全局变量可以在spec文件使用。
    * **DISTPATH：**相对于dist文件夹的相对路径，如果--distpath参数选项被指定了，则使用被指定的参数值。
    * **HOMEPATH：**pyinstaller查找的绝对路径，一般是Python解释器的site-packages文件夹的绝对路径。
    * **SPEC：**在命令行中指定的spec文件路径。
    * **SPECPATH：**os.path.split\(SPEC\)的第一个值。
    * **specnm：**spec文件的文件名，不含文件类型后缀。
    * **workpath：**相对于build文件夹的相对路径，如果workpath=参数选项被指定了，这使用被指定的值。WARNFILE：在build文件夹中警告文件的全路径，一般是warn-myscript.txt
12. **指定了相同的参数：**当命令行和spec中指定了相同的参数选项，那么命令行的参数选项会被忽略。



