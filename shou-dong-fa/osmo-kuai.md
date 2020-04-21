# os模块

**os.path.driname\(path\)：**返回当前路径的上一级路径字符串。

**os.path.basename\(path\)：**返回当前路径的目录名（文件夹名）或文件名（全称）。

**os.path.split\(path\)：**返回一个路径以最后一个路径分割符分割后的元组。

**os.path.splitext\(file\_name\)：**返回文件名和其后缀组成的元组（后缀包含点号，比如“.txt”）。

**os.path.isdir\(path\)：**判断一个路径是否是一个目录（文件夹）。

**os.path.isfile\(path\)：**判断一个路径是否是一个文件。

**os.path.join\(path\_str1, path\_str2\)：**将两个及以上的字符串使用当前系统的路径分隔符连接起来。

**os.path.abspath\(path\)：**返回一个路径的绝对路径。

**os.listdir\(dir\_path\)：**以列表的形式返回一个目录（dir\_path只能是目录，不能是文件名路径）下的所有文件（全称）和文件夹名称。

**os.remove\(file\_path\)：**删除指定文件。

**os.rmdir\(dir\_path\)：**删除一个空目录。

**os.removedirs\(dir\_path\)：**递归删除指定空目录（空文件夹）。

**os.path.exists\(path\)：**判断一个路径是否存在。

**os.mkdir\(dir\_path\)：**新建一个目录（文件夹）。

**os.makedirs\(dir\_path\)：**递归创建目录（文件夹）。

**os.getcwd\(\)：**获取当前工作目录。

**os.chdir\(path\)：**改变当前工作目录为新的目录path。

**os.system\(command\)：**调用dos命令并运行，例如：os.system\('python D:\test.py arg1 arg2'\)，即在DOS界面运行Python文件test.py，并传入参数“arg1”和“arg2”。

**os.\_exit\(status\)：**以指定状态退出Python解释器，并不做任何处理，即运行完这条语句后就会直接退出了，后面的代码都不会执行了。退出Python解释器还有一个sys.exit\(\)方法，详细见sys模块。

**注：**Windows的路径分隔符为“\”，所以写路径字符串的时候一般都是要写成“\”的，但是在Python中，无论什么平台，只需要写“/”就OK了，避免了不同平台的路径分隔符不同的问题。

##### 简单示例：

```text
>>> os.path.dirname('D:\\Games')
'D:\\'
>>> os.path.basename('D:\\Games\\9yin_632\\蜗牛整包\\0x0804.ini')
'0x0804.ini'
>>> os.path.splitext('0x0804.ini')
('0x0804', '.ini')
>>> os.path.abspath('Games')  # 随意写的字符串（相对路径），返回的路径字符串加上了当前的工作路径（绝对路径）
'C:\\Python27\\Games'
```



