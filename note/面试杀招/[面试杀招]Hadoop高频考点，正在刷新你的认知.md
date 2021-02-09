## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一篇文章为大家总结了一些关于Hive的热门考点，得到了一些朋友的肯定与转发，菌菌就觉得花时间去做这些知识整合是非常有价值，有意义的一件事。本篇文章，让我们有幸一起来阅读一下，该怎么准备Hadoop的内容，才有机会在面试过程占据上风。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016092720496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

***
## 一、什么是Hadoop？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是一个看着不起眼，实则“送命题”的典型。往往大家关于大数据的其他内容准备得非常充分，反倒问你什么是Hadoop却有点猝不及防，回答磕磕绊绊，给面试官的印象就很不好。另外，回答这个问题，一定要从事物本身上升到广义去介绍。面试官往往通过这个问题来判断你是否具有最基本的认知能力。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>Hadoop是一个能够对大量数据进行分布式处理的软件框架。以一种可靠、高效、可伸缩的方式进行数据处理</font>。主要包括三部分内容：*Hdfs，MapReduce，Yarn*

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>Hadoop在广义上指一个生态圈，泛指大数据技术相关的开源组件或产品</font>，如HBase，Hive，Spark，Zookeeper，Kafka，flume....

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

![Hadoop生态圈](https://img-blog.csdnimg.cn/20201016100136760.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## 二、能跟我介绍下Hadoop和Spark的差异吗？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;被问到也不要惊讶，面试官往往通过你对于不同技术的差异描述，就能看出你是不是真的具有很强的学习能力。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
|  |  Hadoop|Spark|
|--|--|--|
| 类型 | 基础平台，包含计算，存储，调度 |分布式计算工具|
|场景|大规模数据集上的批处理|迭代计算，交互式计算，流计算|
|价格|对机器要求低，便宜|对内存有要求，相对较贵
|编程范式|MapReduce，API 较为底层，算法适应性差|RDD组成DAG有向无环图，API较为顶层，方便使用
|数据存储结构|MapReduce中间计算结果存在HDFS磁盘上，延迟大|RDD中间运算结果存在内存中，延迟小
|运行方式|Task以进程方式维护，任务启动慢|Task以线程方式维护，任务启动快|
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 三、Hadoop常见的版本有哪些，分别有哪些特点，你一般是如何进行选择的?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个完全就是基于个人的经验之谈的，如果平时没有细致研究过这些，这个问题一定是答不好的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于Hadoop的飞速发展，功能不断更新和完善，Hadoop的版本非常多，同时也显得杂乱。目前市面上，主流的是以下几个版本：

 - Apache 社区版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>Apache</font> 社区版本 <font color='blue'>完全开源，免费，是非商业版本</font>。Apache社区的Hadoop版本分支较多，而且部分Hadoop存在Bug。在选择Hadoop、Hbase、Hive等时，需要考虑兼容性。同时，<font color='blue'>这个版本的Hadoop的部署对Hadoop开发人员或运维人员的技术要求比较高</font>。

 - Cloudera版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='	DeepPink'>Cloudera</font> 版本<font color='DeepPink'> 开源，免费，有商业版和非商业版本</font>，是在Apache社区版本的Hadoop基础上，选择相对稳定版本的Hadoop，进行开发和维护的Hadoop版本。由于此版本的Hadoop在开发过程中对其他的框架的集成进行了大量的兼容性测试，因此<font color='DeepPink'>使用者不必考虑Hadoop、Hbase、Hive等在使用过程中版本的兼容性问题，大大节省了使用者在调试兼容性方面的时间成本</font>。

 - Hortonworks版本

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>Hortonworks</font> 版本 的 Hadoop <font color='red'>开源、免费，有商业和非商业版本</font>，其在 Apache 的基础上修改，对相关的组件或功能进行了二次开发，其中<font color='red'>商业版本的功能是最强大，最齐全的</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以基于以上特点进行选择，我们一般刚接触大数据用的就是CDH，在工作中大概率用 Apache 或者 Hortonworks。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 四、能简单介绍Hadoop1.0，2.0，3.0的区别吗？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一般能问出这种问题的面试官都是“狠人”，基本技术都不差，他们往往是更希望应聘者能在这些“细节”问题上脱颖而出。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>Hadoop1.0由分布式存储系统HDFS和分布式计算框架MapReduce组成</font>，其中HDFS由一个NameNode和多个DateNode组成，MapReduce由一个JobTracker和多个TaskTracker组成。在Hadoop1.0中容易导致单点故障，拓展性差，性能低，支持编程模型单一的问题。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>Hadoop2.0即为克服Hadoop1.0中的不足</font>，提出了以下关键特性：

 - <font color='		OrangeRed'>Yarn</font>：它是Hadoop2.0引入的一个全新的通用资源管理系统，完全代替了Hadoop1.0中的JobTracker。在MRv1 中的 JobTracker 资源管理和作业跟踪的功能被抽象为 ResourceManager 和 AppMaster 两个组件。Yarn 还支持多种应用程序和框架，提供统一的资源调度和管理功能
 - <font color='	OrangeRed'>NameNode 单点故障得以解决</font>：Hadoop2.2.0 同时解决了 NameNode 单点故障问题和内存受限问题，并提供 NFS，QJM 和 Zookeeper 三种可选的共享存储系统
 - <font color='	OrangeRed'>HDFS 快照</font>：指 HDFS（或子系统）在某一时刻的只读镜像，该只读镜像对于防止数据误删、丢失等是非常重要的。例如，管理员可定时为重要文件或目录做快照，当发生了数据误删或者丢失的现象时，管理员可以将这个数据快照作为恢复数据的依据
 - <font color='	OrangeRed'>支持Windows 操作系统</font>：Hadoop 2.2.0 版本的一个重大改进就是开始支持 Windows 操作系统
 - <font color='	OrangeRed'>Append</font>：新版本的 Hadoop 引入了对文件的追加操作

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同时，新版本的Hadoop对于HDFS做了两个非常重要的**增强**，分别是<font color='RoyalBlue'>支持异构的存储层次</font>和<font color='RoyalBlue'>通过数据节点为存储在HDFS中的数据提供内存缓冲功能</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>相比于Hadoop2.0，Hadoop3.0 是直接基于 JDK1.8 发布的一个新版本</font>，同时，Hadoop3.0引入了一些重要的功能和特性

 - <font color='	BlueViolet'>HDFS可擦除编码</font>：这项技术使HDFS在不降低可靠性的前提下节省了很大一部分存储空间
 - <font color='BlueViolet'>多NameNode支持</font>：在Hadoop3.0中，新增了对多NameNode的支持。当然，处于Active状态的NameNode实例必须只有一个。也就是说，从Hadoop3.0开始，在同一个集群中，支持一个 ActiveNameNode 和 多个 StandbyNameNode 的部署方式。
 - <font color='BlueViolet'>MR Native Task优化</font>
 - <font color='BlueViolet'>Yarn基于cgroup 的内存和磁盘 I/O 隔离</font>
 - <font color='BlueViolet'>Yarn container resizing</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;限于篇幅原因，这还都只是部分特性，大家多注意菌哥标记颜色的部分，就足以应对面试了。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## 五、说下Hadoop常用的端口号
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Hadoop常用的端口号总共就那么几个，大家选择好记的几个就OK了

 - dfs.namenode.http-address:50070
 - dfs.datanode.http-address:50075
 - SecondaryNameNode：50090
 - dfs.datanode.address:50010
 - fs.defaultFS:8020 或者9000
 - yarn.resourcemanager.webapp.address:8088
 - 历史服务器web访问端口：19888

## 六、简单介绍一下搭建Hadoop集群的流程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个问题实在基础，这里也简单概述下。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在正式搭建之前，我们需要准备以下6步：

**准备工作**

 1. 关闭防火墙
 2. 关闭SELINUX
 3. 修改主机名
 4. ssh无密码拷贝数据
 5. 设置主机名和IP对应
 6. jdk1.8安装

**搭建工作:**

 - 下载并解压Hadoop的jar包
 - 配置hadoop的核心文件
 - 格式化namenode
 - 启动....

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## 七、介绍一下HDFS读写流程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个问题非常基础，同时出现的频率也是异常的高，但是大家也不要被HDFS的读写流程吓到。相信看到这里的朋友，应该不是第一次背HDFS读写繁多的步骤了，菌哥在这里也不建议大家去背那些文字，这里贴上两张图，大家要学会做到心中有图，万般皆易。

 - HDFS读数据流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016180905938.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

 - HDFS的写数据流程

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016181106661.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

 - &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 八、介绍一下MapReduce的Shuffle过程，并给出Hadoop优化的方案(包括：压缩、小文件、集群的优化)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MapReduce数据读取并写入HDFS流程实际上是有10步



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016182235471.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​       其中最重要，也是最不好讲的就是 shuffle 阶段，当面试官着重要求你介绍 Shuffle 阶段时，可就不能像上边图上写的那样简单去介绍了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;你可以这么说：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1) Map方法之后Reduce方法之前这段处理过程叫**Shuffle**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2) Map方法之后，数据首先进入到分区方法，把数据标记好分区，然后把数据发送到环形缓冲区；<font color='DarkViolet'>环形缓冲区默认大小100m，环形缓冲区达到80%</font>时，进行溢写；<font color='RoyalBlue'>溢写前对数据进行排序，排序按照对key的索引进行字典顺序排序</font>，排序的手段**快排**；溢写产生大量溢写文件，需要对溢写文件进行**归并排序**；<font color='red'>对溢写的文件也可以进行Combiner操作，前提是汇总操作，求平均值不行</font>。最后将文件按照分区存储到磁盘，等待Reduce端拉取。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3）每个Reduce拉取Map端对应分区的数据。拉取数据后先存储到内存中，内存不够了，再存储到磁盘。拉取完所有数据后，采用归并排序将内存和磁盘中的数据都进行排序。在进入Reduce方法前，可以对数据进行分组操作。

> **讲到这里你可能已经口干舌燥，想缓一缓。
> 但面试官可能对你非常欣赏：**
> <font color='black'>**小伙几，看来你对MapReduce的Shuffle阶段掌握很透彻啊，那你跟我再介绍一下你是如何基于MapReduce做Hadoop的优化的，可以给你个提示，可以从<font color='red'>压缩，小文件，集群优化</font>层面去考虑哦~**</font>
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016185355738.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)**可能你心里仿佛有一万只草泥马在奔腾，但是为了顺利拿下本轮面试，你还是不得不开始思考，如何回答比较好：**


### 1）HDFS小文件影响

 - 影响NameNode的寿命，因为文件元数据存储在NameNode的内存中
 - 影响计算引擎的任务数量，比如每个小的文件都会生成一个Map任务

### 2）数据输入小文件处理

 - 合并小文件：对小文件进行归档（Har）、自定义Inputformat将小文件存储成SequenceFile文件。
 - 采用ConbinFileInputFormat来作为输入，解决输入端大量小文件场景
 - 对于大量小文件Job，可以开启JVM重用


### 3）Map阶段

 - 增大环形缓冲区大小。由100m扩大到200m
 - 增大环形缓冲区溢写的比例。由80%扩大到90%
 - 减少对溢写文件的merge次数。（10个文件，一次20个merge）
 - 不影响实际业务的前提下，采用Combiner提前合并，减少 I/O

### 4）Reduce阶段

 - 合理设置Map和Reduce数：两个都不能设置太少，也不能设置太多。太少，会导致Task等待，延长处理时间；太多，会导致 Map、Reduce任务间竞争资源，造成处理超时等错误。
 - 设置Map、Reduce共存：调整 `slowstart.completedmaps` 参数，使Map运行到一定程度后，Reduce也开始运行，减少Reduce的等待时间
 - 规避使用Reduce，因为Reduce在用于连接数据集的时候将会产生大量的网络消耗。
 - 增加每个Reduce去Map中拿数据的并行数
 - 集群性能可以的前提下，增大Reduce端存储数据内存的大小

### 5) IO 传输

 - 采用数据压缩的方式，减少网络IO的的时间
 - 使用SequenceFile二进制文件

### 6) 整体

 - MapTask默认内存大小为1G，可以增加MapTask内存大小为4
 - ReduceTask默认内存大小为1G，可以增加ReduceTask内存大小为4-5g
 - 可以增加MapTask的cpu核数，增加ReduceTask的CPU核数
 - 增加每个Container的CPU核数和内存大小
 - 调整每个Map Task和Reduce Task最大重试次数

### 7) 压缩
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;压缩，可以参考这张图

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016192615132.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

**提示**：如果面试过程问起，我们一般回答压缩方式为<font color='red'>Snappy，特点速度快，缺点无法切分</font>（可以回答在链式MR中，Reduce端输出使用bzip2压缩，以便后续的map任务对数据进行split）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 九、介绍一下 Yarn 的 Job 提交流程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里一共也有两个版本，分别是详细版和简略版，具体使用哪个还是分不同的场合。正常情况下，将简略版的回答清楚了就很OK，详细版的最多做个内容的补充：

 - 详细版



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016193521350.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



 - 简略版



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016193647210.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

&nbsp;       其中简略版对应的步骤分别如下：

 1. <font color='red'>client向RM提交应用程序</font>，其中包括启动该应用的ApplicationMaster的必须信息，例如ApplicationMaster程序、启动ApplicationMaster的命令、用户程序等
 2. <font color='red'>ResourceManager启动一个container用于运行ApplicationMaster</font>
 3. 启动中的ApplicationMaster向ResourceManager<font color='red'>注册</font>自己，启动成功后与RM保持心跳
 4. ApplicationMaster向ResourceManager发送请求,<font color='red'>申请</font>相应数目的container
 5. 申请成功的container，由ApplicationMaster进行<font color='red'>初始化</font>。container的启动信息初始化后，AM与对应的NodeManager通信，要求NM启动container
 6. NM<font color='red'>启动</font>container
 7. container运行期间，ApplicationMaster对container进行<font color='red'>监控</font>。container通过RPC协议向对应的AM汇报自己的进度和状态等信息
 8. 应用运行结束后，ApplicationMaster向ResourceManager<font color='red'>注销</font>自己，并允许属于它的container被收回


## 十、介绍下Yarn默认的调度器，调度器分类，以及它们之间的区别

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;关于Yarn的知识点考察实际上在面试中占的比重并的不多，像面试中常问的无非就Yarn的Job执行流程或者调度器的分类，答案往往也都差不多，以下回答做个参考：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）Hadoop调度器主要分为三类：

 - FIFO Scheduler：先进先出调度器：优先提交的，优先执行，后面提交的等待<font color='	OrangeRed'>【生产环境不会使用】</font>
 - Capacity Scheduler：容量调度器：允许看创建多个任务对列，多个任务对列可以同时执行。但是一个队列内部还是先进先出。<font color='	DarkViolet'>【Hadoop2.7.2默认的调度器】</font>
 - Fair Scheduler：公平调度器：第一个程序在启动时可以占用其他队列的资源（100%占用），当其他队列有任务提交时，占用资源的队列需要将资源还给该任务。还资源的时候，效率比较慢。<font color='Chocolate'>【CDH版本的yarn调度器默认】</font>

## 十一、了解过哪些Hadoop的参数优化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;前面刚回答完Hadoop基于压缩，小文件，IO的集群优化，现在又要回答参数优化，真的好烦啊(Ｔ▽Ｔ)如果你把自己放在实习生这个level，你 duck 不必研究这么多关于性能调优这块的内容，毕竟对于稍有工作经验的工程师来说，<font color='Tomato'>调优这块是非常重要的</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们常见的**Hadoop参数调优**有以下几种：

 - <font color='	IndianRed'>在hdfs-site.xml文件中配置多目录</font>，最好提前配置好，否则更改目录需要重新启动集群
 - <font color='	IndianRed'>NameNode有一个工作线程池</font>，用来处理不同DataNode的并发心跳以及客户端并发的元数据操作

```java
dfs.namenode.handler.count=20 * log2(Cluster Size)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如集群规模为10台时，此参数设置为60

 - <font color='	IndianRed'>编辑日志存储路径dfs.namenode.edits.dir设置与镜像文件存储路径dfs.namenode.name.dir尽量分开</font>，达到最低写入延迟
 - <font color='	IndianRed'>服务器节点上YARN可使用的物理内存总量</font>，默认是8192（MB），注意，如果你的节点内存资源不够8GB，则需要调减小这个值，而YARN不会智能的探测节点的物理内存总量
 - <font color='	IndianRed'>单个任务可申请的最多物理内存量</font>，默认是8192（MB）


## 十二、了解过Hadoop的基准测试吗?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个完全就是基于项目经验的面试题了，暂时回答不上来的朋友可以留意一下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们搭建完Hadoop集群后需要对HDFS读写性能和MR计算能力测试。测试jar包在hadoop的share文件夹下。

## 十三、你是怎么处理Hadoop宕机的问题的?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信被问到这里，一部分的小伙伴已经坚持不下去了



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201016230036696.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

​         但言归正传，被问到了，我们总不能说俺不知道，洒家不会之类的吧٩(๑❛ᴗ❛๑)۶下面展示一种回答，给大家来个Demo。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Tomato'>如果MR造成系统宕机。此时要控制Yarn同时运行的任务数，和每个任务申请的最大内存</font>。调整参数：yarn.scheduler.maximum-allocation-mb（单个任务可申请的最多物理内存量，默认是8192MB）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Tomato'>如果写入文件过量造成NameNode宕机。那么调高Kafka的存储大小，控制从Kafka到HDFS的写入速度。高峰期的时候用Kafka进行缓存</font>，高峰期过去数据同步会自动跟上。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 十四、你是如何解决Hadoop数据倾斜的问题的，能举个例子吗?

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**性能优化**和**数据倾斜**，如果在面试前不好好准备，那就准备在面试时吃亏吧~其实掌握得多了，很多方法都有相通的地方。下面贴出一种靠谱的回答，大家可以借鉴下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）<font color='Chocolate'>提前在map进行combine，减少传输的数据量</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Mapper加上combiner相当于提前进行reduce，即<font color='Chocolate'>把一个Mapper中的相同key进行了聚合，减少shuffle过程中传输的数据量，以及Reducer端的计算量</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Salmon'>如果导致数据倾斜的key 大量分布在不同的mapper的时候，这种方法就不是很有效了</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）数据倾斜的key 大量分布在不同的mapper

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这种情况，大致有如下几种方法：


 - <font color='	Goldenrod'>**局部聚合加全局聚合**</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第一次在map阶段对那些导致了数据倾斜的key 加上1到n的随机前缀，这样本来相同的key 也会被分到多个Reducer 中进行局部聚合，数量就会大大降低。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;第二次mapreduce，去掉key的随机前缀，进行全局聚合。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**思想**：<font color='red'>二次mr，第一次将key随机散列到不同 reducer 进行处理达到负载均衡目的。第二次再根据去掉key的随机前缀，按原key进行reduce处理</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个方法进行两次mapreduce，性能稍差。

 - <font color='Goldenrod'>**增加Reducer，提升并行度**</font>

```java
JobConf.setNumReduceTasks(int)
```

 - <font color='Goldenrod'>**实现自定义分区**</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;根据数据分布情况，自定义散列函数，将key均匀分配到不同Reducer

***
## 彩蛋
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了能鼓励大家多学会总结，菌在这里贴上自己平时做的思维导图，需要的朋友，可以关注博主个人微信公众号【猿人菌】，后台回复“思维导图”即可获取。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201015101722118.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 结语
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很高兴能看到这里的朋友，有任何好的想法或者建议都可以在评论区留言，或者直接私信我也ok，后期会考虑出一些大数据面试的场景题，<font color='BlueViolet'>在最美的年华，做最好的自己</font>，我是00后Alice，我们下一期见~~

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='RoyalBlue'>**一键三连，养成习惯~**</font>

> 文章持续更新，可以微信搜一搜「 **猿人菌** 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经....期待您的关注!

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201116102452301.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)




