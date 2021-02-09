## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在之前的一篇博客👉[《零基础小白如何入门Shell，快来看看(收藏)这篇大总结!!》](https://blog.csdn.net/weixin_44318830/article/details/108062253)中，博主已经为大家介绍了Shell常见的入门级操作，本篇博客，我们就来学习一些进阶的内容，并且还附带一些对应的测试题。感兴趣的小伙伴们记得点个赞以表支持哟~

***
## 常用的Shell工具
### 1、cut
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cut的工作就是“剪”，具体的说就是在文件中负责剪切数据用的。<font color='blue'>**cut命令从文件的每一行剪切字节，字符和字段并将这些字节，字符和字段输出**</font>。

#### 1.1 基本用法

> **cut[选项参数]   filename**
> 说明： 默认分隔符是制表符
#### 1.2 选项参数说明
| 选项参数 | 功能             |
| -------- | ---------------- |
| -f       | 列号，提前第几列 |
|-d|分隔符，按照指定分隔符分割列

#### 1.3 案例实操
 (0) 数据准备

```powershell
[root@node01 datas]# touch cut.txt
[root@node01 datas]# vim cut.txt
dong shen
guan zhen
wo  wo
lai  lai
le  le
```
(1)切割 cut.txt 第一列

```powershell
[root@node01 datas]# cut -d " " -f 1 cut.txt
dong
guan
wo
lai
le
```
(2)切割cut.txt第二，三列

```powershell
[root@node01 datas]# cut -d " " -f 2,3 cut.txt
shen
zhen
 wo
 lai
 le
```
(3)在cut.txt文件中切割出guan

```powershell
[root@node01 datas]# cat cut.txt | grep "guan" | cut -d " " -f 1
guan
```
(4)选取系统PATH变量值，第2个“：”开始后的所有路径：

```powershell
[root@node01 datas]# echo $PATH
/usr/lib64/qt-3.3/bin::/export/servers/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3bin::/export/servers/jdk1.8.0_144/bin:::/export/servers/hbase-1.1.1/bin::/exportrvers/hadoop-2.6.0-cdh5.14.0/bin:/export/servers/hadoop-2.6.0-cdh5.14.0/sbin:/usocal/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/export/servers/hive-1.1.dh5.14.0/bin:/export/servers/kafka_2.11-1.0.0//bin:/export/servers/pig/bin:/exposervers/spark/bin:/export/servers/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/bin:/exporervers/zookeeper-3.4.5-cdh5.14.0/bin:/root/bin
[root@node01 datas]# echo $PATH | cut -d : -f 2-
:/export/servers/kafka-eagle-bin-1.3.2/kafka-eagle-web-1.3.2/bin::/export/servers/jdk1.8.0_144/bin:::/export/servers/hbase-1.1.1/bin::/export/servers/hadoop-2.6.0-cdh5.14.0/bin:/export/servers/hadoop-2.6.0-cdh5.14.0/sbin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/export/servers/hive-1.1.0-cdh5.14.0/bin:/export/servers/kafka_2.11-1.0.0//bin:/export/servers/pig/bin:/export/servers/spark/bin:/export/servers/sqoop-1.4.6.bin__hadoop-2.0.4-alpha/bin:/export/servers/zookeeper-3.4.5-cdh5.14.0/bin:/root/bin
```
(5)切割 ifconfig 后打印的IP地址

```powershell

[root@node01 datas]# ifconfig eth0 | grep "inet addr" | cut -d: -f 2 | cut -d" " -f1
192.168.100.100
```

### 2、sed

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sed是一种流编辑器，**它一次处理一行内容**。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”，接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。<font color='red'>文件内容并没有改变</font>，除非你使用重定向存储输出。

#### 2.1 基本用法

> **sed[选项参数] 'command' filename**

#### 2.2 选项参数说明
| 选项参数 | 功能                              |
| -------- | --------------------------------- |
| -e       | 直接在指令模式上进行sed的动作编辑 |

#### 2.3 命令功能描述
| 命令 | 功能描述                                |
| ---- | --------------------------------------- |
| a    | 新增，a的后面可以接字符串，在下一行出现 |
d|删除
s|查找并替换|


#### 2.4 案例实操
(0) 数据准备

```powershell
[root@node01 datas]# touch sed.txt
[root@node01 datas]# vim sed.txt
dong shen
guan zhen
wo  wo
lai  lai

le  le
```
(1) 将“mei nv”这个单词插入到sed.txt第二行下，打印
```powershell
[root@node01 datas]# sed '2a mei nv' sed.txt
dong shen
guan zhen
mei nv
wo  wo
lai  lai

le  le
```
<font color='red'>注意：文件并没有改变</font>

(2) 删除 sed.txt 文件所有包含 wo 的行

```powershell
[root@node01 datas]# sed '/wo/d' sed.txt
dong shen
guan zhen
lai  lai

le  le
```
(3) 将sed.txt文件中wo替换为ni

```powershell
[root@node01 datas]# sed 's/wo/ni/g' sed.txt
dong shen
guan zhen
ni  ni
lai  lai

le  le
```
<font color='red'>注意：‘g’表示global，全部替换</font>

(4) 将sed.txt文件中的第二行删除并将wo替换为ni

```powershell
[root@node01 datas]# sed -e '2d' -e 's/wo/ni/g' sed.txt
dong shen
ni  ni
lai  lai

le  le
```

### 3、awk
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>一个强大的文本分析工具</font>，把文件逐行的读入，以空格为默认分隔符将每行切片，切开的部分再进行分析处理。

#### 3.1 基本用法

> **awk [选项参数] ‘pattern1{action1}  pattern2{action2}...’ filename**
> pattern : 表示AWK在数据中查找的内容，就是匹配模式
> action：在找到匹配内容时所执行的一系列命令

#### 3.2 选项参数说明
| 选项参数 | 功能                 |
| -------- | -------------------- |
| -F       | 指定输入文件折分隔符 |
-v|赋值一个用户定义变量|

#### 3.3 案例实操
(0) 数据准备

```powershell
[root@node01 datas]# cp /etc/passwd ./
```

(1) 搜索passwd文件以root关键字开头的所有行，并输出该行的第7列

```powershell
[root@node01 datas]# awk -F : '/^root/{print $7}' passwd
/bin/bash
```
(2) 搜索passwd文件以root关键字开头的所有行，并输出该行的第1列和第7列，中间以“，”号分割

```powershell
[root@node01 datas]# awk -F : '/^root/{print $1","$7}' passwd
root,/bin/bash
```
<font color='red'>注意：只有匹配了pattern的行才会执行action</font>

(3) 只显示 passwd 文件的第一列和第七列，以逗号分割，且在第一行内容前面添加列名`user，shell`在最后一行添加内容`dahaige，/bin/zuishuai`

```powershell
[root@node01 datas]# awk -F : 'BEGIN{print "user,shell"}{print $1","$7} END{print "dahaige,/bin/zuishuani"}' passwd
user,shell
root,/bin/bash
bin,/sbin/nologin
......
hadoop,/bin/bash
dahaige,/bin/zuishuani
```

<font color='red'>注意：BEGIN 在所有数据读取行之前执行；END 在所有数据执行之后执行。</font>

(4)将passwd文件中的用户id增加数值1并输出

```powershell
[root@node01 datas]# awk -v i=1 -F : '{print $3 + i}' passwd
1
2
3
4
......
```

#### 3.4 awk的内置变量
| 变量     | 说明   |
| -------- | ------ |
| FILENAME | 文件名 |
|NR|已读的记录数
|NF|浏览记录的域的个数(切割后，列的个数)

#### 3.5 案例实操
(1) 统计 passwd 文件名，每行的行号，每行的列数

```powershell
[root@node01 datas]# awk -F : '{print "filename:" FILENAME ",linenumber:" NR ",columns:" NF}' passwd
filename:passwd,linenumber:1,columns:7
filename:passwd,linenumber:2,columns:7
filename:passwd,linenumber:3,columns:7
......
```

### 4、sort
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='BlueViolet'> **sort** 命令在Linux里非常有用，它将文件进行排序，并将排序结果标准输出。</font>

#### 4.1 基本语法
> sort(选项)(参数)

| 选项 | 说明               |
| ---- | ------------------ |
| -n   | 依照数值的大小排序 |
|-r|以相反的顺序来排序
|-t|设置排序时所用的分隔字符
-k|指定需要排序的列|

<font color='gray'>**参数：指定待排序的文件列表**</font>

#### 4.2 案例实操
(0) 数据准备

```powershell
[root@node01 datas]# touch sort.sh
[root@node01 datas]# vim sort.sh
bb:40:5.4
bd:20:4.2
xz:50:2.3
cls:10:3.5
ss:30:1.6
```
(1) 按照 " : " 分割后的第三列倒序排序。

```powershell
[root@node01 datas]# sort -t : -nrk 3 sort.sh
bb:40:5.4
bd:20:4.2
cls:10:3.5
xz:50:2.3
ss:30:1.6
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200906200830178.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;看到这里的朋友，一定对于Shell有了新的认知，但是我们了解得再多，终归还是需要通过实践来测试我们的能力。下面菌哥放上几道关于Shell的<font color='blue'>企业真实面试题</font>，感兴趣的朋友可以测试一下~

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020090620063560.jpg#pic_center)


## 企业真实面试题
### 1、京东
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题1：使用Linux命令查询 sed.txt 中空行所在的行号

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`awk '/^$/{print NR}' sed.txt`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题2：有文件 chengji.txt 内容如下:

```powershell
张三 40
李四 50
王五 60
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用Linux命令计算第二列的和并输出

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`cat chengji.txt | awk -F " " '{sum+=$2} END{print sum}'`

### 2、搜狐&和讯网
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题1：Shell脚本里如何检查一个文件是否存在？如果不存在该如何处理？

```powershell
#!/bin/bash

if [ -f file.txt ]; then
   echo "文件存在!"
else
   echo "文件不存在!"
fi
```

### 3、新浪
问题1：用shell写一个脚本，对文本中无序的一列数字排序
```powershell
[root@node01 datas]# cat demo.txt
9
8
7
6
5
4
3
2
10
1
[root@node01 datas]# sort -n demo.txt | awk '{a+=$0;print $0}END{print "SUM="a}'
1
2
3
4
5
6
7
8
9
10
SUM=55
```

### 3、金和网络
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题1：请用shell脚本写出查找当前文件夹下所有的文本文件内容中包含有字符”shen”的文件名称

```powershell
[root@node01 datas]# grep -r "shen" .
./sed.txt:dong shen
./cut.txt:dong shen
[root@node01 datas]# grep -r "shen" . | cut -d ":" -f 1
./sed.txt
./cut.txt
```

## 小结

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇博客介绍了Shell常用的几种工具：cut，sed，awk，sort。这些工具不论是在Linux的开发，还是在大数据运维环境下，使用的频率都很高，热爱学习的小伙伴们记得勤加练习哟~



