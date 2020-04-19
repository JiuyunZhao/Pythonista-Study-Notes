### 一、if语句

####  单分支if语句

语法（中括号首尾的空格不能省略）：

```bash
if [ 条件判断式 ];then
    程序
fi
#或者
if [ 条件判断式 ]
    then
        程序
fi
```

示例：

```bash
#!/bin/bash

#根分区的使用率如果达到80则发出警告，向屏幕输出一条提示信息。
rate=$(df -h | grep /dev/sda5 | awk '{print $5}' | cut -d "%" -f 1)

if [ $rate -ge 80 ]
    then
        echo "/dev/sda5 is full!!!"
fi
```

#### 

#### 双分支if语句

语法：

```bash
if [ 条件判断式 ]
    then
        程序1
    else
        程序2
fi
```

示例1：对数据进行备份

```bash
#!/bin/bash
#获取当前系统时间，并以年月日的格式显示
date=$(date +%y%m%d)
#获取目录/etc的大小
size=$(du -sh /etc)

#如果存在目录
if [ -d /tmp/dbback ]
    then
        echo "Date is: $date" > tmp/dbback/db.txt
        echo "Size is: $size" >> /tmp/dbback/db.txt
        #在脚本中也是可以使用cd这样的命令的
        cd /tmp/dbback
        #打包压缩文件进行备份，并且将命令执行后的信息丢弃
        tar -zcf etc_$date.tar.gz /etc db.txt &>/dev/null
        rm -rf /tmp/dbback/db.txt
    else
        mkdir /tmp/dbback
        echo "Date is: $date" > tmp/dbback/db.txt
        echo "Size is: $size" >> /tmp/dbback/db.txt
        cd /tmp/dbback
        tar -zcf etc_$date.tar.gz /etc db.txt &>/dev/null
        rm -rf /tmp/dbback/db.txt
fi
```

示例2：检查某个服务是否正常运行

```bash
#!/bin/bash
port=$(nmap -sT 192.168.1.159 | grep tcp | grep http | awk '{print $2}')
#使用nmap命令扫描服务器，并截取Apache服务的状态
if [ "$port"=="open" ]
    then
        echo "$(date) httpd is ok!" >> /tmp/autostart-acc.log
    else
        #重启Apache服务
        /etc/rc.d/init.d/httpd start $>/dev/null
        echo "$(date) restart httpd!!" >> /tmp/autostart-err.log
fi
```



#### 多分支if语句

语法：

```bash
if [ 条件判断式1 ]
    then
        程序1
elif [ 条件判断式2 ]
    then
        程序2
...
else
    程序n
fi
```

示例：

```bash
#!/bin/bash

# 从键盘输入读取值并赋予变量file
read -p "Please input a filename: " file

#判断变量file是否为空
if [ -z "$file" ]
    then
        echo "Error, ase input a filename!"
        #退出并设置返回码
        exit 1
#判断文件是否存在
elif [ ! -e "$file" ]
    then
        echo "Error, your input is not a file!"
        exit 2
#判断file的值是否为普通文件
elif [ -f "$file" ] 
    then
        echo "$file is a regulare file!"
#判断file的值是否为目录文件
elif [ -d "$file" ]
    then
        echo "$file is a directory!"
else
    echo "$file is an other file!"
fi
```



### 二、case语句

语法：

```bash
case $变量名 in
	"值1")
		程序1
		;;
	"值2")
		程序2
		;;
	...
	*)
		程序n
		;;
esac
```

示例：

```bash
#!/bin/bash

read -p "Please choose yes/no: " -t 30 cho
case $cho in
	"yes")
		echo "Your choose is yes!"
		;;
	"no")
		echo "Your choose is no!"
		;;
	*)
		echo "Your choose is error!"
		;;
esac
```



