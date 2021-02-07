## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;众所周知，Hadoop 提供了命令行接口，对HDFS中的文件进行管理操作，如<font color='	Tomato'>**读取文件**、**新建目录**、**移动文件**、**复制文件**、**删除目录**、**上传文件**、**下载文件**、**列出目录**</font>等。本期文章，菌哥打算为大家详细介绍 Hadoop 的命令行接口！希望大家看完之后，能够有所收获
|ू･ω･` )
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119134752287.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HDFS命令行的格式如下所示：

```bash
Hadoop fs -cmd <args>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中，cmd是要执行的具体命令；<args>是要执行命令的参数，但不限于一个参数。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要查看命令行接口的帮助信息，只需在命令行中输入如下命令：

```bash
hadoop fs
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;即不添加任务具体的执行命令，Hadoop 就会列出命令行接口的帮助信息，如下所示：

```bash
[root@node01 ~]# hadoop fs
Usage: hadoop fs [generic options]
        [-appendToFile <localsrc> ... <dst>]
        [-cat [-ignoreCrc] <src> ...]
        [-checksum <src> ...]
        [-chgrp [-R] GROUP PATH...]
        [-chmod [-R] <MODE[,MODE]... | OCTALMODE> PATH...]
        [-chown [-R] [OWNER][:[GROUP]] PATH...]
        [-copyFromLocal [-f] [-p] [-l] <localsrc> ... <dst>]
        [-copyToLocal [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-count [-q] [-h] [-v] [-x] <path> ...]
        [-cp [-f] [-p | -p[topax]] <src> ... <dst>]
        [-createSnapshot <snapshotDir> [<snapshotName>]]
        [-deleteSnapshot <snapshotDir> <snapshotName>]
        [-df [-h] [<path> ...]]
        [-du [-s] [-h] [-x] <path> ...]
        [-expunge]
        [-find <path> ... <expression> ...]
        [-get [-p] [-ignoreCrc] [-crc] <src> ... <localdst>]
        [-getfacl [-R] <path>]
        [-getfattr [-R] {-n name | -d} [-e en] <path>]
        [-getmerge [-nl] <src> <localdst>]
        [-help [cmd ...]]
        [-ls [-C] [-d] [-h] [-q] [-R] [-t] [-S] [-r] [-u] [<path> ...]]
        [-mkdir [-p] <path> ...]
        [-moveFromLocal <localsrc> ... <dst>]
        [-moveToLocal <src> <localdst>]
        [-mv <src> ... <dst>]
        [-put [-f] [-p] [-l] <localsrc> ... <dst>]
        [-renameSnapshot <snapshotDir> <oldName> <newName>]
        [-rm [-f] [-r|-R] [-skipTrash] <src> ...]
        [-rmdir [--ignore-fail-on-non-empty] <dir> ...]
        [-setfacl [-R] [{-b|-k} {-m|-x <acl_spec>} <path>]|[--set <acl_spec> <path>]]
        [-setfattr {-n name [-v value] | -x name} <path>]
        [-setrep [-R] [-w] <rep> <path> ...]
        [-stat [format] <path> ...]
        [-tail [-f] <file>]
        [-test -[defsz] <path>]
        [-text [-ignoreCrc] <src> ...]
        [-touchz <path> ...]
        [-usage [cmd ...]]

Generic options supported are
-conf <configuration file>     specify an application configuration file
-D <property=value>            use value for given property
-fs <local|namenode:port>      specify a namenode
-jt <local|resourcemanager:port>    specify a ResourceManager
-files <comma separated list of files>    specify comma separated files to be copied to the map reduce cluster
-libjars <comma separated list of jars>    specify comma separated jar files to include in the classpath.
-archives <comma separated list of archives>    specify comma separated archives to be unarchived on the compute machines.

The general command line syntax is
bin/hadoop command [genericOptions] [commandOptions]
```
## 1、文件准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在服务器本地创建 data.txt 文件用于测试，文件的内容如下所示：

```bash
hello hadoop
```

## 2、-appendToFile
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将服务器本地的文件追加到HDFS指定的文件中，如果多次运行相同的参数，则会在 HDFS 的文件中追加多行相同的内容。实例代码如下所示：

```bash
hadoop fs -appendToFile data.txt /data/data.txt
```

## 3、-cat
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要用来查看 HDFS 中的非压缩文件的内容。实例代码如下所示：

```bash
[root@node01 ~]# hadoop fs -cat  /data/data.txt
hello hadoop
hello hadoop
```
## 4、-checksum
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 HDFS 中文件的校验和。实例代码如下所示：
```bash
[root@node01 ~]# hadoop fs -checksum  /data/data.txt
/data/data.txt  MD5-of-0MD5-of-512CRC32C        000002000000000000000000c8e21d30c9ed5817cd5ff40768a34389
```
## 5、-chgrp
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;改变 HDFS 中文件或目录的所属组，-R 选项可以改变目录下所有子目录的所属组，执行此命令的用户必须是文件或目录的所有者或超级用户。实例代码如下所示：

```bash
hadoop fs -chgrp hadoop /data/data.txt
```

## 6、-chmod
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改 HDFS 中文件或目录的访问权限，-R 选项可以修改目录下的所有子目录的访问权限，执行此命令的用户必须是文件或目录的所有者或超级用户。实例代码如下所示：

```bash
hadoop fs -chmod 700 /data/data.txt
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此时，data.txt 文件当前的访问权限已经被修改为“ -rwx------”

## 7、chown
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;修改文件或目录的所有者，-R选项可以修改目录下所有子目录的所有者，此命令的用户必须是超级用户。实例代码如下所示：

```bash
hadoop fs -chown alice:alice /data/data.txt
```

## 8、-copyFromLocal
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将本地服务器上的文件复制到HDFS中。实例代码如下所示：

```bash
hadoop fs -copyFromLocal a.txt /data/
```
## 9、-copyToLocal
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将 HDFS 中的文件复制到服务器本地。实例代码如下所示：

```bash
hadoop fs -copyToLocal /data/data.txt /home/hadoop/input
```

## 10、-count
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显示目录下的子目录数、文件数、占用字节数、所有文件和目录名，-q 选项显示目录和空间的配额信息。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -count /data/
           4            9                456 /data
```
## 11、-cp
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;复制文件或目录，如果源文件或目录有多个，则目标必须为目录。实例代码如下所示：

```bash
hadoop fs -cp /data/data.txt /data/data.tmp
```
## 12、-createSnapshot
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为HDFS中的文件创建快照，实例代码如下：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先在 HDFS 中创建目录 /sn，并将 /sn 目录设置为可快照，如下所示：

```bash
[root@node01 zwj]# hadoop fs -mkdir /sn
[root@node01 zwj]# hdfs dfsadmin -allowSnapshot /sn
Allowing snaphot on /sn succeeded
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来执行创建快照操作，如下所示：

```bash
[root@node01 zwj]# hadoop fs -createSnapshot /sn s1
Created snapshot /sn/.snapshot/s1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说明创建快照成功。

## 13、-deleteSnapshot
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;删除 HDFS 中的文件快照，实例代码如下所示：

```bash
hadoop fs -deleteSnapshot /sn sn1
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;删除 /sn 目录的快照sn1

## 14、-df 
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 HDFS 中目录空间的使用情况。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -df -h /data
Filesystem             Size    Used  Available  Use%
hdfs://node01:8020  130.1 G  13.7 G     57.8 G   11%
```

## 15、-du
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 HDFS 或目录中的文件大小。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -du -h -s -x /data
456  1.3 K  /data
```

## 16、-expunge
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;清空HDFS中的回收站，实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -expunge
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: TrashPolicyDefault#deleteCheckpoint for trashRoot: hdfs://node01:8020/user/root/.Trash
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: TrashPolicyDefault#deleteCheckpoint for trashRoot: hdfs://node01:8020/user/root/.Trash
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: Deleted trash checkpoint: /user/root/.Trash/201028063715
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: Deleted trash checkpoint: /user/root/.Trash/201031181139
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: TrashPolicyDefault#createCheckpoint for trashRoot: hdfs://node01:8020/user/root/.Trash
20/12/27 20:41:48 INFO fs.TrashPolicyDefault: Created trash checkpoint: /user/root/.Trash/201227204148
```
## 17、-find
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查找 HDFS 中指定目录下的文件。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -find /data /data/data.txt
/data
/data/a.txt
/data/data.txt
```
## 18、-get
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将 HDFS 中的文件复制到本地服务器。实例代码如下所示：

```bash
hadoop fs -get /data/data.txt  /home/hadoop/input
```
## 19、-getfacl
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看HDFS中指定目录下的文件的访问控制列表，-R 选项可以查看所有子目录下的文件访问控制列表。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -getfacl /data
# file: /data
# owner: root
# group: supergroup
```
## 20、-getfattr
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 HDFS 上的文件扩展属性信息，-R 选项可以查看当前目录下所有子目录中的文件扩展属性信息或子目录下文件的扩展属性信息。实例代码如下所示：
```bash
[root@node01 zwj]# hadoop fs -getfattr -R -d /data
# file: /data
# file: /data/a.txt
# file: /data/data.txt
# file: /data/input
```
## 21、-getmerge
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将 HDFS 中的多个文件合并为一个文件，复制到本地服务器。实例代码如下所示：

```bash
hadoop fs -getmerge /data/a.txt /data/b.txt /home/hadoop/input/data.local
```

## 22、-head
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以head方式查看 HDFS 中的文件，此命令后面的文件只能为文件，不能为目录，实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -head /data/data.txt
hello hadoop
hello hadoop
```
## 23、-help
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 Hadoop 具体命令的帮助信息。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -help cat
-cat [-ignoreCrc] <src> ... :
  Fetch all files that match the file pattern <src> and display their content on
  stdout.
```
## 24、-ls
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;列出 HDFS 中指定目录下的信息，实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -ls /data
Found 3 items
-rw-r--r--   3 root supergroup          6 2020-12-27 20:11 /data/a.txt
-rw-r--r--   3 root supergroup         26 2020-12-27 18:59 /data/data.txt
drwxr-xr-x   - root supergroup          0 2020-09-18 19:16 /data/input
```
## 25、-mkdir
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 HDFS 上创建目录，实例代码如下所示：

```bash
hadoop fs -mkdir /test/data
```
## 26、-moveFromLocal
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;移动本地服务器上的某个文件到 HDFS 中。实例代码如下所示：

```bash
hadoop fs -moveFromLocal /home/hadoop/input/data.local /data/
```
## 27、-moveToLocal
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;移动 HDFS 中的文件到本地服务器的某个目录下。

```bash
hadoop fs -moveToLocal  /data/data.txt  /home/hadoop/input/
```

> 注意：| 此命令在 Hadoop3.2.0 版本中尚未实现

## 28、-mv
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;移动 HDFS 中的目录到 HDFS 中的另一个目录下。实例代码如下所示：

```bash
hadoop fs -mv /data/data.local /test
```
## 29、-put
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;复制本地文件到 HDFS 中的某个目录下。实例代码如下所示：

```bash
hadoop fs -put /home/hadoop/input/data.local /data
```
## 30、-renameSnapshot
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;重命名 HDFS 上的文件快照。实例代码如下：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先在 HDFS 中创建目录 /sn，并将 /sn 目录设置为可快照，如下所示：

```bash
[root@node01 zwj]# hadoop fs -mkdir /sn
[root@node01 zwj]# hdfs dfsadmin -allowSnapshot /sn
Allowing snaphot on /sn succeeded
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来执行创建快照操作，如下所示：

```bash
[root@node01 zwj]# hadoop fs -createSnapshot /sn s1
Created snapshot /sn/.snapshot/s1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;说明创建快照成功。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来将 /sn 目录的快照名称 sn1 重命名为 sn2，如下所示：

```bash
hadoop fs -renameSnapshot /sn sn1 sn2
```

## 31、-rm
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;删除文件或目录。实例代码如下所示：

```bash
hadoop fs -rm /data/data.local
```

## 32、-rmkdir
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;删除HDFS上的目录，此目录必须是空目录。实例代码如下所示：

```bash
hadoop fs -mkdir /test
```

## 33、-setrep
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;设置 HDFS 上的文件的目标副本数量，-R 选项可以对子目录逐级进行相同的操作， -w 选项等待副本达到设置值。实例代码如下所示：

```bash
hadoop fs -setrep 5 /data/data.txt
```

## 34、-stat
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看 HDFS 上文件或目录的统计信息，以 format 的格式列出。可选的 format 格式如下：

 1.  %b：文件所占的块数
 2. %g：文件所属的用户组
 3. %n：文件名
 4. %o：文件块大小
 5. %r：备份数
 6. %u：文件所属用户
 7. %y：文件修改时间

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例代码如下所示：

```bash
[root@node01 zwj]$ hadoop fs -stat %b,%g,%n,%o,%r,%u,%y /data
0,hive,data,0,0,hive,2020-11-16 07:54:04
```
## 35、-tail
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;显示一个文件的末尾数据，通常是显示文件最后的 1KB 的数据。-f 选项可以监听文件的变化，当有内容追加到文件中时，-f 选项能够实时显示追加的内容。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -tail /data/data.txt
hello hadoop
hello hadoop
```
## 36、-test
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;检测文件的信息，参数选项如下：

 1. -d：如果路径为目录则返回0
 2. -e：如果路径存在则返回0
 3. -f：如果路径为文件则返回0
 4. -s：如果路径中的文件大于0字节则返回0
 5. -w：如果路径存在并且具有写权限则返回0
 6. -r：如果路径存在并且具有读权限则返回0
 7. -z：如果路径中的文件为0字节则返回0，否则返回1 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;实例代码如下所示：

```bash
hadoop fs -test -d /data
```

## 37、-text
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看文件内容。text 命令除了能够查看非压缩的文本文件内容之外，也能查看压缩后的文本文件内容；cat命令只能查看非压缩的文本文件内容。实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -text /data/data.txt
hello hadoop
hello hadoop
```

## 38、touch
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 HDFS 上创建文件，如果文件不存在则不报错，实例代码如下所示：

```bash
hadoop fs -touch /data/data.touch
```
## 39、-truncate
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;切断 HDFS 上的文件，实例代码如下所示：

```bash
[root@node01 zwj]# hadoop fs -truncate 26 /data/data.txt
Truncate /data/data.txt to length: 26
```

## 40、-usage
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;列出指定命令的使用格式，实例代码如下所示：

```bash
[[root@node01 zwj]# hadoop fs -usage cat
Usage: hadoop fs [generic options] -cat [-ignoreCrc] <src> ...
```

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期内容为大家介绍了 40 个 HDFS常用的命令，还有一些不常用的命令我就没有列出来，等着感兴趣的小伙伴们自行去探索。之后的文章，我会先把**FlinkSQL**的内容更完，然后会根据自己平时做的笔记，出一些**硬核**的知识总结，等到复习的差不多了，开始更一个**实时数仓**的项目，感兴趣的小伙伴们记得及时关注，第一时间获取技术干货！你知道的越多，你不知道的也越多，我是**Alice**，我们下一期见！！！

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> 文章持续更新，可以微信搜一搜「 **猿人菌** 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…期待您的关注!



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210119135940253.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)