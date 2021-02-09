&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近有点小忙，心细的朋友们可能已经看出菌已经好久没更新博客了。但是不慌，该掌握的知识，咋们也不能落下。这一期博客，我也不搞那些花里胡哨了，专心写一篇总结Shell精华的博客，也算是为像Alice一样的“小小白”谋点福利吧…φ(๑˃∀˂๑)♪ 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817192928253.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
@[TOC]
***

## 1、Shell 概述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先，博主作为一个大数据开发人士，先问大家一个问题：程序员为什么需要学习Shell呢?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;原因也很简单，最明显的无非就是下面两点：

 - 需要看懂运维人员编写的 Shell 程序
 - 偶尔会编写一些简单 Shell 程序来管理集群，提高开发效率。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200817193731785.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
## 2、Shell解析器

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）Linux提供的Shell解析器有：

```powershell
[root@node01 hive-1.1.0-cdh5.14.0]# cat /etc/shells
/bin/sh
/bin/bash
/sbin/nologin
/bin/dash
/bin/tcsh
/bin/csh
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）bash和sh的关系

```powershell
[root@node01 bin]$ ll | grep bash
-rwxr-xr-x. 1 root root 941880 5月  11 2016 bash
lrwxrwxrwx. 1 root root      4 5月  27 2017 sh -> bash
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）Centos默认的解析器是bash

```powershell
[root@node01 bin]$ echo $SHELL
/bin/bash
```

## 3、Shell脚本入门
### 3.1 脚本格式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;脚本以 <font color='red'>#!/bin/bash </font>开头（指定解析器）

### 3.2  编写第一个Shell脚本:helloworld
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）需求：创建一个Shell脚本，输出helloworld

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）案例实操：

```powershell
[root@node01 datas] touch helloworld.sh
[root@node01 datas] vi helloworld.sh

在helloworld.sh中输入如下内容
#!/bin/bash
echo "helloworld"
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）脚本的常用执行方式

 - 第一种：采用bash或sh+脚本的相对路径或绝对路径<font color='blue'>（不用赋予脚本+x权限）</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sh+脚本的相对路径

```powershell
[root@node01 datas] sh helloworld.sh 
Helloworld
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sh+脚本的绝对路径

```powershell
[root@node01 datas] sh /home/atguigu/datas/helloworld.sh 
helloworld
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bash+脚本的相对路径

```powershell
[root@node01 datas] bash helloworld.sh 
Helloworld
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bash+脚本的绝对路径

```powershell
[root@node01 datas] bash /home/atguigu/datas/helloworld.sh 
Helloworld
```

 - 第二种：采用输入脚本的绝对路径或相对路径执行脚本<font color='red'>（必须具有可执行权限+x）</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（a）首先要赋予helloworld.sh 脚本的+x权限

```powershell
[root@node01 datasdatas]$ chmod 777 helloworld.sh
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（b）执行脚本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相对路径

```powershell
[root@node01 datas datas]$ ./helloworld.sh 
Helloworld
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;绝对路径

```powershell
[root@node01 datas datas]$ /home/atguigu/datas/helloworld.sh 
Helloworld
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>注意：第一种执行方法，本质是bash解析器帮你执行脚本，所以脚本本身不需要执行权限。第二种执行方法，本质是脚本需要自己执行，所以需要执行权限。</font>

 - 3．第二个Shell脚本：多命令处理

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）需求：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在/home/datas目录下创建一个banzhang.txt,在banzhang.txt文件中增加“I love cls”。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）案例实操：

```powershell
[root@node01 datas] touch batch.sh
[root@node01 datas] vi batch.sh

在batch.sh中输入如下内容
#!/bin/bash

cd /home/datas
touch cls.txt
echo "I love cls" >>cls.txt
```

## 4、Shell中的变量
### 4.1 系统变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1.  常用系统变量

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`$HOME`、`$PWD`、`$SHELL`、`$USER`等


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2.  案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）查看系统变量的值

```powershell
[root@node01 datas] echo $HOME
/home/node01
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）显示当前Shell中所有变量：set

```powershell
[root@node01 datas] set
BASH=/bin/bash
BASH_ALIASES=()
BASH_ARGC=()
BASH_ARGV=()
....
```

### 4.2 自定义变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）定义变量：变量=值 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）撤销变量：unset 变量

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）声明静态变量：<font color='blue'>readonly变量，注意：不能unset</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．变量定义规则

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）变量名称可以由`字母`、`数字`和`下划线`组成，但是不能以数字开头，<font color='red'>环境变量名建议大写</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>（2）等号两侧不能有空格</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）在bash中，变量默认类型都是字符串类型，无法直接进行数值运算。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）变量的值如果有空格，需要使用双引号或单引号括起来。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）定义变量A

```powershell
[root@node01 datas] A=5
[root@node01 datas] echo $A
5
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）给变量A重新赋值

```powershell
[root@node01 datas] A=8
[root@node01 datas] echo $A
8
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）撤销变量A

```powershell
[root@node01 datas] unset A
[root@node01 datas] echo $A
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）声明静态的变量B=2，不能unset

```powershell
[root@node01 datas] readonly B=2
[root@node01 datas] echo $B
2
[root@node01 datas] B=9
-bash: B: readonly variable
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（5）<font color='blue'>在bash中，变量默认类型都是字符串类型</font>，无法直接进行**数值**运算

```powershell
[atguigu@hadoop102 ~] C=1+2
[atguigu@hadoop102 ~] echo $C
1+2
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（6）变量的值如果有空格，需要使用双引号或单引号括起来

```powershell
[root@node01 ~] D=I love banzhang
-bash: world: command not found
[root@node01 ~] D="I love banzhang"
[root@node01 ~] echo $A
I love banzhang
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（7）可把变量提升为全局环境变量，可供其他Shell程序使用

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用法： <font color='red'>export  变量名</font>

```powershell
[root@node01 datas] vim helloworld.sh 

#!/bin/bash

echo "helloworld"
echo $B

[root@node01 datas] ./helloworld.sh 
Helloworld
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;发现并没有打印输出变量B的值

```powershell
[root@node01 datas]export B
[root@node01 datas]./helloworld.sh 
helloworld
2
```
### 4.3 特殊变量：$n
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
$n	（功能描述：n为数字，$0代表该脚本名称，$1-$9代表第一到第九个参数，十以上的参数需要用大括号包含，如${10}）
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）输出该脚本文件名称、输入参数1和输入参数2 的值

```powershell
[root@node01 datas] touch parameter.sh 
[root@node01 datas] vim parameter.sh

#!/bin/bash
echo "$0  $1   $2"

[atguigu@hadoop101 datas] chmod 777 parameter.sh

[atguigu@hadoop101 datas] ./parameter.sh cls  xz
./parameter.sh  cls   xz
```

### 4.4 特殊变量：$#
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```python
$#	（功能描述：获取所有输入参数个数，常用于循环）。
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）获取输入参数的个数

```powershell
[root@node01 datas] vim parameter.sh

#!/bin/bash
echo "$0  $1   $2"
echo $#

[atguigu@hadoop101 datas] chmod 777 parameter.sh

[atguigu@hadoop101 datas] ./parameter.sh cls  xz
parameter.sh cls xz 
2
```

### 4.5 特殊变量：$ *、$@
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
$*	（功能描述：这个变量代表命令行中所有的参数，$*把所有的参数看成一个整体）
$@	（功能描述：这个变量也代表命令行中所有的参数，不过$@把每个参数区分对待）
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）打印输入的所有参数

```powershell
[root@node01 datas]# bash paramter.sh 1 2 3
paramter.sh 1 2
3
1 2 3
1 2 3
```

### 4.6 特殊变量: $?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
$？	（功能描述：最后一次执行的命令的返回状态。如果这个变量的值为0，证明上一个命令正确执行；如果这个变量的值为非0（具体是哪个数，由命令自己来决定），则证明上一个命令执行不正确了。）
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）判断helloworld.sh脚本是否正确执行

```powershell
[atguigu@hadoop101 datas] ./helloworld.sh 
hello world
[atguigu@hadoop101 datas] echo $?
0
```

## 5、运算符

 1. 基本语法:

（1）`“$((运算式))”或“$[运算式]”`

（2）`expr  + , - , \*,  /,  %    加，减，乘，除，取余`

 2. 案例实操:

&nbsp;&nbsp;&nbsp;&nbsp;□  计算3+2的值

```powershell
[atguigu@hadoop101 datas] expr 2 + 3
5
```

&nbsp;&nbsp;&nbsp;&nbsp;□  计算3-2的值

```powershell
[atguigu@hadoop101 datas] expr 3 - 2 
1
```

&nbsp;&nbsp;&nbsp;&nbsp;□  计算（2+3）X4的值

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;■  expr一步完成计算

```powershell
[atguigu@hadoop101 datas] expr `expr 2 + 3` \* 4
20
```


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;■ 采用$[运算式] 方式

```powershell
[atguigu@hadoop101 datas]# S=$[(2+3)*4]
[atguigu@hadoop101 datas]# echo $S
```

## 6、条件判断
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[ condition ] （<font color='red'>注意condition前后要有空格</font>）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：条件非空即为true，[ alice ]返回true，[] 返回false。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 常用判断条件

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<1>  两个整数之间比较

```powershell
= 字符串比较
-lt 小于（less than）			-le 小于等于（less equal）
-eq 等于（equal）				-gt 大于（greater than）
-ge 大于等于（greater equal）	-ne 不等于（Not equal）
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<2>  按照文件权限进行判断

```powershell
-r 有读的权限（read）			-w 有写的权限（write）
-x 有执行的权限（execute）
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<3> 按照文件类型进行判断

```powershell
-f 文件存在并且是一个常规的文件（file）
-e 文件存在（existence）		-d 文件存在并是一个目录（directory）
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）23是否大于等于22

```powershell
[atguigu@hadoop101 datas][ 23 -ge 22 ]
[atguigu@hadoop101 datas]echo $?
0
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）helloworld.sh是否具有写权限

```powershell
[atguigu@hadoop101 datas][ -w helloworld.sh ]
[root@node01 datas] echo $?
0
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）/home/atguigu/cls.txt目录中的文件是否存在

```powershell
[root@node01 datas] [ -e /home/atguigu/cls.txt ]
[root@node01 datas] echo $?
1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）多条件判断（&& 表示前一条命令执行成功时，才执行后一条命令，|| 表示上一条命令执行失败后，才执行下一条命令）

```powershell
[root@node01 ~][ condition ] && echo OK || echo notok
OK
[root@node01 datas] [ condition ] && [ ] || echo notok
notok
```

## 7、流程控制（重点）
### 7.1 if 判断

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
if [ 条件判断式 ];then 
  程序 
fi 
或者 
if [ 条件判断式 ] 
  then 
    程序 
fi
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意事项：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）[ 条件判断式 ]，中括号和条件判断式之间必须有空格。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）<font color='red'>if后要有空格</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）输入一个数字，如果是1，则输出alice zhen shuai，如果是2，则输出cls zhen mei，如果是其它，什么也不输出。

```powershell
[root@node01 datas] touch if.sh
[root@node01 datas] vim if.sh

#!/bin/bash

if [ $1 -eq "1" ]
then
        echo "alice zhen shuai"
elif [ $1 -eq "2" ]
then
        echo "alice zhen mei"
fi

[root@node01 datas] chmod 777 if.sh 
[root@node01 datas] ./if.sh 1
alice zhen shuai
```

### 7.2 case 语句
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
case $变量名 in 
  "值1"） 
    如果变量的值等于值1，则执行程序1 
    ;; 
  "值2"） 
    如果变量的值等于值2，则执行程序2 
    ;; 
  …省略其他分支… 
  *） 
    如果变量的值都不是以上的值，则执行此程序 
    ;; 
esac
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意事项:

```powershell
# !/bin/bash

case $1 in
"1")
   echo "alice"
;;
"2")
   echo "TomWhite"
;;
*)
   echo "Cindy"
;;
esac

[root@node01 datas] chmod 777 case.sh
[root@node01 datas] ./case.sh 1
1
```

### 7.3 for 循环
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法1


```powershell
for (( 初始值;循环控制条件;变量变化 )) 
  do 
    程序 
  done
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

（1）从1加到100

```powershell
[root@node01 datas] touch for1.sh
[root@node01 datas] vim for1.sh

#!/bin/bash

s=0
for((i=0;i<=100;i++))
do
        s=$[$s+$i]
done
echo $s

[root@node01 datas] chmod 777 for1.sh 
[root@node01 datas] ./for1.sh 
5050
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3．基本语法2

```powershell
for 变量 in 值1 值2 值3… 
  do 
    程序 
  done
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）打印所有输入参数

```powershell
[root@node01 datas] touch for2.sh
[root@node01 datas] vim for2.sh

#!/bin/bash
#打印数字

for i in $*
 do
   echo "alice is beautiful"
 done

[root@node01 datas] chmod 777 for2.sh 
[root@node01 datas] bash for2.sh cls xz bd
alice is beautiful
alice is beautiful
alice is beautiful
```


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）比较 $* 和 $@ 区别

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>（a）`$*`  和 `$@` 都表示传递给函数或脚本的所有参数，不被双引号“”包含时，都以 `$1 $2 … $n` 的形式输出所有参数。</font>

```powershell
#!/bin/bash

for i in $*
do
  echo "alice is beautiful " + $i
done

for j in $@
do echo "TomWhite wrote the first book about hadoop" + $j
done

[root@node01 datas]# bash for.sh cls cz bd
alice is beautiful  + cls
alice is beautiful  + cz
alice is beautiful  + bd
TomWhite wrote the first book about hadoop + cls
TomWhite wrote the first book about hadoop + cz
TomWhite wrote the first book about hadoop + bd
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>（b）当它们被双引号“”包含时，“$*”会将所有的参数作为一个整体，以`“$1 $2 …$n”`的形式输出所有参数；“$@”会将各个参数分开，以`“$1” “$2”…”$n”`的形式输出所有参数。</font>

```powershell
#!/bin/bash

for i in "$*"
#$*中的所有参数看成是一个整体，所以这个for循环只会循环一次
do
  echo "alice is beautiful " + $i
done

for j in "$@"
#$@中的每个参数都看成是独立的，所以“$@”中有几个参数，就会循环几次
do echo "TomWhite wrote the first book about hadoop" + $j
done


[root@node01 datas]# bash for.sh cls cz bd
alice is beautiful  + cls cz bd
TomWhite wrote the first book about hadoop + cls
TomWhite wrote the first book about hadoop + cz
TomWhite wrote the first book about hadoop + bd
```

### 7.4 While循环
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
while [ 条件判断式 ] 
  do 
    程序
  done
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）从1加到100

```powershell
[root@node01 datas] touch while.sh
[root@node01 datas] vim while.sh

#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done

echo $s

[root@node01 datas] chmod 777 while.sh 
[root@node01 datas] ./while.sh 
5050
```

## 8、read读取控制台输入

 1. 基本语法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>read(选项)(参数)</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选项:

 - <font color='	BlueViolet'>-p：指定读取值时的提示符；</font>
 - <font color='	BlueViolet'>-t：指定读取值时等待的时间（秒）</font>


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;参数

 - <font color='SlateBlue'>变量：指定读取值的变量名</font>


 2. 案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）提示7秒内，读取控制台输入的名称

```powershell
[root@node01 datas] touch read.sh
[root@node01 datas] vim read.sh

#!/bin/bash

read -t 7 -p "Enter your name in 7 seconds " NAME
echo $NAME

[root@node01 datas] bash ./read.sh 
Enter your name in 7 seconds alice
alice
```
## 9、函数
### 9.1 系统函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．<font color='red'>**basename**</font> 基本语法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>**basename [string / pathname] [suffix]**  	</font>（功能描述：basename命令会删掉所有的前缀包括最后一个（‘/’）字符，然后将字符串显示出来。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;选项：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='BlueViolet'>suffix为后缀，如果suffix被指定了，basename会将pathname或string中的suffix去掉</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）截取该 /home/atguigu/banzhang.txt 路径的文件名称

```powershell
[root@node01 datas] basename /home/atguigu/banzhang.txt 
banzhang.txt
[root@node01 datas] basename /home/atguigu/banzhang.txt .txt
banzhang
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3.	<font color='red'>**dirname**</font> 基本语法

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>**dirname 文件绝对路径**		</font>（功能描述：从给定的包含绝对路径的文件名中去除文件名（非目录的部分），然后返回剩下的路径（目录的部分））


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）获取banzhang.txt文件的路径

```powershell
[root@node01 ~]dirname /home/atguigu/banzhang.txt 
/home/atguigu
```

### 9.2 自定义函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1．基本语法

```powershell
[ function ] funname[()]
{
	Action;
	[return int;]
}
funname
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. <font color='blue'>经验技巧</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）<font color='blue'>必须在调用函数地方之前，先声明函数，shell脚本是逐行运行。不会像其它语言一样先编译。</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）<font color='BlueViolet'>函数返回值，只能通过 `$?` 系统变量获得，可以显示加：return返回，如果不加，将以最后一条命令运行结果，作为返回值。return后跟数值n(0-255)</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3．案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）计算两个输入参数的和

```powershell
[root@node01 datas] touch fun.sh
[root@node01 datas] vim fun.sh

#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo "$s"
}

read -p "Please input the number1: " n1;
read -p "Please input the number2: " n2;
sum $n1 $n2;

[root@node01 datas] chmod 777 fun.sh
[root@node01 datas] ./fun.sh 
Please input the number1: 2
Please input the number2: 5
7
```

***

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好啦，到这里，Shell最基础，最入门的部分已经介绍完了，感兴趣的小伙伴们记得勤加练习😀下一篇博客，菌哥将为大家带来Shell的进阶——常用工具的使用💪。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​        ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201116102452301.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)