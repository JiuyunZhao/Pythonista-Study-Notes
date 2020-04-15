Vim在Linux中是一个简单却又强大的文本编辑器，可以用来创建、编辑和查看一个文本。本文只是简单介绍下，更多用法还得个人多使用练习才行。



Vim通常分为三种模式：

**命令模式：**刚打开Vim时就默认进入命令模式，此时所有的键盘敲击都会被识别为命令而不是字符输入，而编辑器则处于等待用户输入命令的状态。

**输入模式：**即插入模式，通常使用此模式来编辑文本，当按下Esc键时自动退出输入模式，并进入命令模式。

**底线命令模式：**在命令模式中输入冒号:就可以进入底线命令模式了，输入命令后回车即可执行对应的命令并退出底线命令模式，当按下Esc键时也会自动退出底线命令模式，并进入命令模式。



**命令模式中常用的命令有：

* **a/A：**a表示在光标所在字符后插入，A表示在光标所在行尾插入，此时进入输入模式。
* **i/I：**i表示在光标所在字符前插入，I表示在光标所在行首插入，此时进入输入模式。
* **o/O：**o表示在光标所在行下插入新行，O表示在光标所在行上插入新行，此时进入输入模式。
* **gg：**定位到第一行。
* **G：**定位到最后一行。
* **\[n\]G：**表示定位到第n行，如先按下数字88（并不会在屏幕上显示出来你的按键），再按下G就会跳转到第88行，效果同底线命令模式的命令“:88”。
* **$：**光标移动至行尾。
* **0：**光标移动至行首。
* **x：**删除光标所在处的字符。
* **\[n\]x：**删除光标所在处后的n个字符。
* **dd：**删除（剪切）光标所在行。
* **\[n\]dd：**删除（剪切）光标所在行及之后的共n行。
* **dG：**删除光标所在行到文件末尾的所有内容。
* **D：**删除光标所在处到文件末尾。
* **yy：**复制当前行。
* **\[n\]yy：**复制当前行及以下n行。
* **p/P：**粘贴在光标所在行的下面或上面。
* **r：**替代光标所在处的字符。
* **R：**进入替换状态，从光标所在处开始替换字符，按Esc结束。
* **u：**取消上一步操作。
* **/\[string\]：**搜索指定的字符串，然后回车，按n可以查看下一个搜索结果。但默认是区分大小写的，想要不区分大小写，需要执行一个底线命令模式的命令“:set ic”，反之，又想区分大小写了，执行“:set noic”。
* **ZZ：**快捷键，保存修改并退出。



**底线命令模式中常用的命令有（省略了冒号:）：

* **set nu：**设置行号。
* **set nonu：**取消行号。
* **\[n\]：**定位到第n行，如“:50”表示定位到50行。
* **\[n1\],\[n2\]d：**删除n1到n2行的所有内容。
* **set \[ic/noic\]：**不区分大小写和区分大小写。
* **%s/\[old\]/\[new\]/\[g/c\]：**在全文中将old字符串替换为指定的new字符串，g表示执行时不询问，c表示执行时询问。
* **\[n1\],\[n2\]s/\[old\]/\[new\]/\[g/c\]：**在指定范围n1到n2行之间将old字符串替换为指定的new字符串，g表示执行时不询问，c表示执行时询问。
* **w：**保存修改。
* **w newfilename：**另存为指定文件。
* **wq：**保存修改并退出。
* **q!：**不保存修改并退出。
* **wq!：**保存修改并退出（文件所有者和root可使用），当修改了权限为只读的文件时，只使用:wq是不能保存的，这时候可以使用:wq!强行保存修改。



**Vim更多技巧

最前面的冒号表示底线命令模式：

* **:r \[filename\]：**将其他文件的内容从光标所在处导入到本文件中。
* **:!\[命令\]：**在不退出Vim的情况下执行命令。
* **:r !\[命令\]：**将一个命令的执行结果导入从光标所在处导入到本文件中。
* **:\[n1\],\[n2\]s/^/\#/g：**连续多行注释（即将所有行的行首都替换为\#，^表示行首，其他语言的注释同理替换即可）。
* **:\[n1\],\[n2\]s/^\#//g：**取消多行的行首注释（即将所有行的行首的\#替换为空字符，^表示行首，其他语言的注释同理替换即可）。


