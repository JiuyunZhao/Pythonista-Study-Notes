# **虚拟环境**

使Python框架的不同版本可以在同一台电脑上运行。如果在电脑上全局（C盘或者其他目录）安装Flask（或其他Python框架），当你使用其他版本的Flask（比如有新版本了！），那有可能这个版本和之前的版本就不兼容，你就不能在同一台电脑上运行不同版本的Flask。

## **一、virtualenv**

**安装：**pip install virtualenv（即“virtual environment”的简写）。

**创建并激活/退出虚拟环境：**

1. **mkdir Virtualenv：**创建一个目录用于存放所有的虚拟环境（目录名可以自定义）；
2. **cd Virtualenv：**进入创建的Virtualenv目录；
3. **virtualenv flask-env：**使用命令virtualenv（virtualenv此为创建虚拟环境的命令名称）创建属于Flask（flask-env为虚拟环境名称，可以自定义）的虚拟环境。创建虚拟环境时指定Python解释器，比如：virtualenv -p C:\Python36\python.exe python36\_env\_test，即在上面第三步的命令中间加了“-p”参数和Python解释器的绝对路径；
4. **cd flask-env：**进入创建的虚拟环境（即进入该目录）；
5. **cd Scripts：**进入Scripts目录；
6. **activate/deactivate：**激活/退出该虚拟环境（激活成功后不再是以盘符开头，而是以“\(flask-env\)”虚拟环境的名称开头），如图：

![](/assets/flask-1.png)

## **二、virtualenvwrapper**

可以安装一个对虚拟环境操作更加方便和强大的工具virtualenvwrapper，可以安装virtualenv后再安装virtualenvwrapper，也可以直接安装virtualenvwrapper（没有安装virtualenv时，virtualenvwrapper会先自动安装virtualenv）。

* **pip install virtualenvwrapper-win：**在windows系统上安装virtualenvwrapper。
* **mkvirtualenv new\_env: **在一个默认的路径下创建虚拟环境（C:\Users\Administrator\Envs）。如果不想使用默认路径，可以在环境变量中配置WORKON\_HOME来指定创建虚拟环境的路径。如果想为虚拟环境指定Python解释器，则使用如下命令：mkvirtualenv --python==C:\Python27\python.exe new\_env。
* **workon new\_env: **进入某个虚拟环境（不用再cd到虚拟环境的路径，也不用使用命令activate来激活虚拟环境）。
* **lsvirtualenv: **列出所有虚拟环境。
* **rmvirtualenv new\_env: **删除某个虚拟环境。
* **cdvirtualenv new\_env: **cd到某个虚拟环境的路径（已在虚拟环境中），如果还没进入虚拟环境，则cd到该虚拟环境对应的Python解释器路径下。



