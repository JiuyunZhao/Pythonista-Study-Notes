### 一、for循环

#### 语法一

```bash
for 变量 in 值1 值2 值3...
    do
        程序
    done
```

**注：**多个值之间只要“有空”，不只是空格，换行符、制表符等都行，比如读取文件时，可以自动遍历每一行。

**示例1：**遍历固定的某些项

```bash
#!/bin/bash

for time in morning noon afternoon evening
    do
        echo "This time is $time !"
    done
```

**示例2：**遍历某个变量中的值

```bash
#!/bin/bash

#解压lamp目录下的所有tar.gz压缩包
cd /lamp
ls *.tar.gz > ls.log
for i in $(cat ls.log)
    do
        tar -zxf $i $> /dev/null
    done
rm -rf /lamp/ls.log
```

#### 语法二

```bash
for (( 初始值;循环控制条件;变量变化 ))
    do
        程序
    done
```

**注：**执行流程为，先运行“初始值”语句，然后执行循环体一次，然后执行“变量变化”语句，然后判断是否符合“循环控制条件”，如果符合则继续执行，否则退出for循环。

**示例：**

```bash
#!/bin/bash

s=0
for (( i=1;i<=100;i=i+1 ))
    do
        s=$(( $s+$i ))
    done
echo "The sum of 1+2+...+100 is: $s"
```



### 二、while循环

**语法：**

```bash
while [ 条件判断式 ]
    do
        程序
    done
```

只要判断式成立，循环就会一直继续运行。

执行顺序为，先判断“条件判断式”是否成立，如果成立则执行循环体，执行完后再次判断“条件判断式”，依次类推，直到”条件判断式“不成立时退出循环。

**示例：**

```bash
#!/bin/bash

i=1
s=0
while [ $i -le 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "The sum is: $s"
```

### 

### 三、until循环

**语法：**

```bash
until [ 条件判断式 ]
    do
        程序
    done
```

与while循环正好相反，当条件判断式成立时则退出循环。

执行顺序为，先判断“条件判断式”是否成立，如果不成立则执行循环体，执行完后再次判断“条件判断式”，依次类推，直到”条件判断式“成立时退出循环。

**示例：**

```bash
#!/bin/bash

i=1
s=0
while [ $i -gt 100 ]
    do
        s=$(( $s+$i ))
        i=$(( $i+1 ))
    done
echo "The sum is: $s"
```



