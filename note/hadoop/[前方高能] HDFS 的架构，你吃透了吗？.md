## 前言

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HDFS 是 Hadoop 中存储数据的基石，存储着所有的数据，具有高可靠性，高容错性，高可扩展性，高吞吐量等特征，能够部署在大规模廉价的集群上，极大地降低了部署成本。有意思的是，**其良好的架构特征使其能够存储海量的数据**。本篇文章，我们就来系统学习一下，Hadoop HDFS的架构！

![](https://img-blog.csdnimg.cn/20210228110745899.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## HDFS架构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HDFS采用 `Master/Slave` 架构存储数据，且支持 NameNode 的 HA。HDFS架构主要包含客户端，`NameNode`，`SecondaryNameNode` 和 `DataNode` 四个重要组成部分，如图所示：

![](https://img-blog.csdnimg.cn/20210228145111743.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）客户端向NameNode发起请求，获取元数据信息，这些元数据信息包括命名空间、块映射信息及 DataNode 的位置信息等。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）NameNode 将元数据信息返回给客户端。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）客户端获取到元数据信息后，到相应的 DataNode 上读/写数据

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）相关联的 DataNode 之间会相互复制数据，以达到 DataNode 副本数的要求

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（5）DataNode 会定期向 NameNode 发送心跳信息，将自身节点的状态信息报告给 NameNode。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（6）SecondaryNameNode 并不是 NameNode 的备份。SecondaryNameNode 会定期获取 NameNode 上的 `fsimage `和 `edits log` 日志，并将二者进行合并，产生 `fsimage.ckpt` 推送给 NameNode。

### 1、NameNode
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='DarkCyan'>NameNode 是整个 Hadooop 集群中至关重要的组件，它维护着整个 HDFS 树，以及文件系统树中所有的文件和文件路径的元数据信息</font>。这些元数据信息包括**文件名**，**命令空间**，**文件属性**（文件生成的时间、文件的副本数、文件的权限）、**文件数据块**、**文件数据块与所在 DataNode 之间的映射关系**等。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='tomato'>一旦 NameNode 宕机或 NameNode 上的元数据信息损坏或丢失，基本上就会丢失 Hadoop 集群中存储的所有数据，整个 Hadoop 集群也会随之瘫痪</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 Hadoop 运行的过程中， NameNode 的主要功能如下图所示：
![](https://img-blog.csdnimg.cn/20210228151714899.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

### 2、SecondaryNameNode
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='DarkOrchid'>SecondaryNameNode 并不是 NameNode 的备份，在NameNode 发生故障时也不能立刻接管 NameNode 的工作</font>。SecondaryNameNode 在 Hadoop 运行的过程中具有两个作用：一个是**备份数据镜像**，另一个是**定期合并日志与镜像**，因此可以称其为 Hadoop 的**检查点**（checkpoint）。<font color='RoyalBlue'>SecondaryNameNode 定期合并 NameNode 中的 fsimage 和 edits log，能够防止 NameNode 重启时把整个 fsimage 镜像文件加载到内存，耗费过长的启动时间</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SecondaryNameNode 的工作流程如图所示：

![](https://img-blog.csdnimg.cn/20210228175857945.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​        **SecondaryNameNode的工作流程如下：**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）`SecondaryNameNode` 会通知 `NameNode` 生成新的 `edits log` 日志文件。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）`NameNode` 生成新的 `edits log` 日志文件，然后将新的日志信息写到新生成的 `edits log` 日志文件中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）`SecondaryNameNode` 复制 `NameNode` 上的 `fsimage` 镜像和 `edits log` 日志文件，此时使用的是 **http get** 方式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）`SecondaryNameNode` 将`fsimage `将镜像文件加载到内存中，然后执行 `edits log` 日志文件中的操作，生成新的镜像文件 `fsimage.ckpt`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（5）`SecondaryNameNode` 将 `fsimage.ckpt` 文件发送给 `NameNode`，此时使用的是 **http post** 方式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（6）`NameNode` 将 `edits log` 日志文件替换成新生成的 `edits.log` 日志文件，同样将 fsimage文件替换成 `SecondaryNameNode` 发送过来的新的 `fsimage` 文件。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（7）`NameNode` 更新 `fsimage` 文件，将此次执行 `checkpoint` 的时间写入 fstime 文件中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;经过 `SecondaryNameNode` 对 `fsimage` 镜像文件和 `edits log` 日志文件的复制和合并操作之后，`NameNode` 中的 `fsimage` 镜像文件就保存了最新的 `checkpoint` 的元数据信息， `edits log` 日志文件也会重新写入数据，两个文件中的数据不会变得很大。因此，当重启 `NameNode` 时，不会耗费太长的启动时间。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**SecondaryNameNode 周期性地进行 checkpoint 操作需要满足一定的前提条件，这些条件如下**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）`edits log` 日志文件的大小达到了一定的阈值，此时会对其进行合并操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）每隔一段时间进行 **checkpoint** 操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这些条件可以在`core-site.xml`文件中进行配置和调整，代码如下所示：

```bash
<property>
         <name>fs.checkpoint.period</name>
         <value>3600</value>
</property>
<property>
         <name>fs.checkpoint.size</name>
         <value>67108864</value>
</property>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述代码配置了 `checkpoint` 发生的时间周期和 `edits log`日志文件的大小阈值，说明如下。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）**fs.checkpoint.period**：表示触发 `checkpoint`发生的时间周期，这里配置的时间周期为 1 h。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）**fs.checkpoint.size**：表示 `edits log` 日志文件大小达到了多大的阈值时会发生 `checkpoint`操作，这里配置的 `edits log`大小阈值为 64 MB。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上述代码中配置的 `checkpoint`操作发生的情况如下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）如果 `edits log` 日志文件经过 1 h 未能达到 64 MB，但是满足了 `checkpoint`发生的周期为 1 h 的条件，也会发生 `checkpoint` 操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）如果 `edits log`日志文件大小在 1 h 之内达到了 64MB，满足了 `checkpoint` 发生的 `edits log`日志文件大小阈值的条件，则会发生 `checkpoint`操作。

> **注意**：如果 NameNode 发生故障或 NameNode 上的元数据信息丢失或损坏导致 NameNode 无法启动，此时就需要人工干预，将 NameNode 中的元数据状态恢复到 SecondaryNameNode 中的元数据状态。此时，如果 SecondaryNameNode 上的元数据信息与 NameNode 宕机时的元数据信息不同步，则或多或少地会导致 Hadoop 集群中丢失一部分数据。出于此原因，<font color='tomato'>**应尽量避免将 NameNode 和 SecondaryNameNode 部署在同一台服务器上**</font>。

### 3、DataNode
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='SlateBlue'>DataNode 是真正存储数据的节点</font>，这些数据以**数据块**的形式存储在 DataNode 上。一个数据块包含两个文件：一个是**存储数据本身的文件**，另一个是**存储元数据的文件**（这些元数据主要包括数据块的长度、数据块的检验和、时间戳）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataNode 运行时的工作机制如图所示：

![](https://img-blog.csdnimg.cn/20210301233255910.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如图所示，DataNode 运行时的工作机制如下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）DataNode启动之后，向 NameNode **注册**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）NameNode **返回**注册成功的消息给 DataNode。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）DataNode 收到 NameNode 返回的注册成功的信息之后，会**周期性**地向 NameNode **上报**当前 DataNode 的所有块信息，默认发送所有数据块的时间周期是 **1h**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）DataNode **周期性**地向NameNode 发送心跳信息；NameNode **收到** DataNode 发来的心跳信息后，会将DataNode 需要执行的命令放入到 心跳信息的 返回数据中，**返回**给 DataNode。DataNode 向 NameNode 发送心跳信息的默认时间周期是 **3s**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（5）NameNode **超过一定的时间**没有收到 DataNode 发来的心跳信息，则 NameNode 会认为对应的 DataNode **不可用**。默认的超时时间是 10 min。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（6）**在存储上相互关联的 DataNode 会同步数据块，以达到数据副本数的要求**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当 DataNode 发生故障导致 DataNode 无法与 NameNode 通信时，NameNode 不会立即认为 DataNode 已经 “死亡”。要经过一段**短暂的超时时长**后才会认为 DataNode 已经 “死亡”。HDFS 中默认的超时时长为 10 min + 30 s，可以用如下公式来表示这个超时时长：


```bash
timeout = 2 * dfs.namenode.heartbeat.recheck-interval +10 * dfs.heartbeat.interval
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中，各参数的含义如下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）`timeout`：超时时长。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）`dfs.namenode.heartbeat.recheck-interval`：检查过期 DataNode 的时间间隔，与 `dfs.heartbeat.interval` 结合使用，默认的单位是 ms，默认时间是 5 min。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）`dfs.heartbeat.interval`：检测数据节点的时间间隔，默认的单位为 s，默认的时间是 3 s。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以，可以得出 DataNode 的默认超时时长为 630s，如下所示：

```bash
timeout = 2 * 5 * 60 + 10 * 3 = 630s
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DataNode 的超时时长也可以在 `hdfs-site.xml `文件中进行配置，代码如下所示：

```xml
<property>
     <name>dfs.namenode.heartbeat.recheck-interval</name>
     <value>3000</value>
</property>
<property>
     <name>dfs.heartbeat.interval</name>
     <value>2</value>
</property>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据上面的公式可以得出，在配置文件中配置的超时时长为：

```bash
timeout = 2 * 3000 / 1000 + 10 * 2 = 26s
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当 DataNode 被 NameNode 判定为 “**死亡**”时，HDFS 就会马上自动进行**数据块的容错复制**。此时，当被 NameNode 判定为 “死亡” 的 DataNode 重新加入集群中时，如果其存储的数据块并没有损坏，就会造成 **HDFS 上某些数据块的备份数超过系统配置的备份数目**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HDFS上**删除多余的数据块**需要的时间长短和数据块报告的时间间隔有关。该参数可以在 `hdfs-site.xml `文件中进行配置，代码如下所示：

```xml
<property>
     <name>dfs.blockreport.intervalMsec</name>
     <value>21600000</value>
     <description>Determines block reporting interval in milliseconds.</description>
</property>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据块报告的时间间隔默认为 `21600000`ms，即 6h，可以通过调整此参数的大小来调整数据块报告的时间间隔。 

***
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇文章算是对 **HDFS的架构**解释的比较透彻，相信不论是刚入门的小白，还是已经有了一定基础的大数据学者，看完都会有一定的收获，希望大家平时学习也能够多学会总结，用输出倒逼自己输入！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 巨人的肩膀
> 1、《海量数据处理与大数据技术实战》
> 2、《大数据平台架构与原型实现》
> 3、https://baike.baidu.com/item/hdfs/4836121?fr=aladdin 
> 4、http://hadoop.apache.org/docs/r1.0.4/cn/hdfs_user_guide.html

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，本篇文章就到这里，更多干货文章请关注我的公众号。**你知道的越多，你不知道的也越多**。我是梦想家，点个关注，我们下一期见！
![](https://img-blog.csdnimg.cn/2021030301520171.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)