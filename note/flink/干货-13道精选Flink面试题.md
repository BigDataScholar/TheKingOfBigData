&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信小伙伴们对于Flink一定不会感到陌生，作为**连续三年蝉联第一，荣膺全球最活跃的 Apache 开源项目**，Flink在中国的热度也一直是居高不下。近几年，在社区的推动下，Flink 技术栈在越来越多的公司开始得到应用，因此**在大数据的求职招聘中，对于Flink的着重考察也变得越来越重要**。本期文章，菌哥就带大家来总结一下，在面试过程中，Flink常被问到的知识点有哪些？如果本文对你有帮助，记得在看完之后，一键三连(✧◡✧)
![](https://img-blog.csdnimg.cn/20210126133106608.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

***
## 1、应用架构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：公司怎么提交的实时任务，有多少 Job Manager？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1. 我们使用 `yarn session` 模式提交任务。每次提交都会创建一个新的 Flink 集群，为每一个 job 提供一个 `yarn-session`，任务之间互相独立，互不影响， 方便管理。任务执行完成之后创建的集群也会消失。线上命令脚本如下：

```bash
bin/yarn-session.sh -n 7 -s 8 -jm 3072 -tm 32768 -qu root.*.* -nm *-* -d
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中申请 7 个 taskManager，每个 8 核，每个 taskmanager 有 32768M 内存。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2. 集群默认只有一个 Job Manager。但为了防止单点故障，我们配置了高可用。 我们公司一般配置一个主 Job Manager，两个备用 Job Manager，然后结合 ZooKeeper 的使用，来达到高可用。


## 2、压测和监控
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：怎么做压力测试和监控？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：我们一般碰到的压力来自以下几个方面：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一，产生数据流的速度如果过快，而下游的算子消费不过来的话，会产生**背压**。 背压的监控可以使用 **Flink Web UI**(localhost:8081) 来**可视化监控**，一旦报警就能知道。一般情况下背压问题的产生可能是由于 **sink** 这个 操作符没有优化好，做一下 优化就可以了。比如如果是写入 ElasticSearch， 那么可以改成**批量写入**，可以调 大 ElasticSearch **队列的大小**等等策略。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;二，设置 watermark 的**最大延迟时间**这个参数，如果设置的过大，可能会造成**内存的压力**。可以设置最大延迟时间小一些，然后把迟到元素发送到**侧输出流**中去。 晚一点更新结果。或者使用类似于 **RocksDB** 这样的状态后端， RocksDB 会开辟堆外存储空间，但 IO 速度会变慢，需要权衡。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;三，还有就是**滑动窗口的长度**如果过长，而滑动距离很短的话，Flink 的性能会下降的很厉害。我们主要通过**时间分片**的方法，将每个元素只存入一个“重叠窗 口”，这样就可以减少窗口处理中状态的写入。（详情链接：[Flink 滑动窗口优化](https://www.infoq.cn/article/sIhs_qY6HCpMQNblTI9M)）

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;四，**状态后端**使用 RocksDB，还没有碰到被撑爆的问题

## 3、为什么用 Flink
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：为什么使用 Flink 替代 Spark？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：主要考虑的是 flink 的**低延迟**、**高吞吐量**和对**流式数据**应用场景更好的支持；另外，flink 可以很好地处理**乱序**数据，而且可以保证 **exactly-once** 的状态一致性。

## 4、checkpoint 的理解
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：如何理解Flink的**checkpoint**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：Checkpoint是Flink实现**容错机制**最核心的功能，它能够根据配置周期性地基于Stream中各个Operator/task的状态来生成**快照**，从而将这些状态数据**定期持久化存储**下来，当Flink程序一旦意外崩溃时，重新运行程序时可以有选择地从这些快照进行恢复，从而修正因为故障带来的程序数据异常。他可以存在内存，文件系统，或者 RocksDB。


## 5、exactly-once 的保证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：如果下级存储不支持事务，Flink 怎么保证 exactly-once？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：端到端的 exactly-once 对 sink 要求比较高，具体实现主要有**幂等写入**和 **事务性**写入两种方式。幂等写入的场景依赖于业务逻辑，更常见的是用事务性写入。 而事务性写入又有`预写日志（WAL）`和`两阶段提交（2PC）`两种方式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果外部系统不支持事务，那么可以用**预写日志**的方式，把结果数据先当成状态保存，然后在收到 **checkpoint** 完成的通知时，一次性写入 sink 系统。

## 6、状态机制
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：说一下 Flink 状态机制？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：Flink 内置的很多算子，包括源 source，数据存储 sink 都是有状态的。在 Flink 中，**状态始终与特定算子相关联**。Flink 会以 **checkpoint** 的形式对各个任务的 状态进行快照，用于保证故障恢复时的**状态一致**性。Flink 通过状态后端来管理状态 和 checkpoint 的存储，状态后端也可以有不同的配置选择。

## 7、海量 key 去重
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：怎么去重？考虑一个实时场景：双十一场景，滑动窗口长度为 1 小时， 滑动距离为 10 秒钟，亿级用户，怎样计算 UV？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：使用类似于 scala 的 set 数据结构或者 redis 的 set 显然是不行的， 因为可能有上亿个 Key，内存放不下。所以可以考虑使用**布隆过滤器**（Bloom Filter） 来去重。

## 8、checkpoint 与 spark 比较

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;问题：Flink 的 **checkpoint** 机制对比 spark 有什么不同和优势？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答：** spark streaming 的 checkpoint 仅仅是针对 driver 的故障恢复做了**数据和元数据的checkpoint**。而 flink 的 checkpoint 机制要复杂了很多，它采用的是轻量级的分布式快照，实现了**每个算子的快照，及流动中的数据的快照**。

## 9、watermark 机制
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：请详细解释一下 Flink 的 Watermark 机制。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：在使用 EventTime 处理 Stream 数据的时候会遇到数据乱序的问题，流处理从 Event（事 件）产生，流经 Source，再到 Operator，这中间需要一定的时间。虽然大部分情况下，传输到 Operator 的数据都是按照**事件产生的时间**顺序来的，但是也不排除由于**网络延迟**等原因而导致乱序的产生，特别是使用 Kafka 的时候，多个分区之间的数据无法保证有序。因此， <font color='RoyalBlue'>**在进行 Window 计算的时候，不能无限期地等下去，必须要有个机制来保证在特定的时间后， 必须触发 Window 进行计算**</font>，这个特别的机制就是 **Watermark（水位线）**。Watermark是用于**处理乱序事件**的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 Flink 的窗口处理过程中，如果确定全部数据到达，就可以对 Window 的所有数据做窗口计算操作（如汇总、分组等），如果数据没有全部到达，则**继续等待该窗口中的数据全部到达才开始处理**。这种情况下就需要用到**水位线(WaterMarks**)机制，它能够<font color='RoyalBlue'>**衡量数据处理进度（表达数据到达的完整性）**</font>，保证事件数据（全部）到达 Flink 系统，或者在乱序及延迟到达时，也能够像预期一样计算出正确并且连续的结果。


## 10、exactly-once 如何实现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：Flink 中 `exactly-once` 语义是如何实现的，状态是如何存储的？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;解答：Flink 依靠 **checkpoint** 机制来实现 **exactly-once** 语义，如果要实现端到端 的 exactly-once，还需要外部 source 和 sink 满足一定的条件。状态的存储通过状态 后端来管理，Flink 中可以配置不同的状态后端。

## 11、CEP
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：Flink CEP 编程中当状态没有到达的时候会将数据保存在哪里？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：在流式处理中，CEP 当然是要支持 **EventTime** 的，那么相对应的也要支持数据的迟到现象，也就是 watermark的处理逻辑。CEP对未匹配成功的事件序 列的处理，和迟到数据是类似的。在 Flink CEP 的处理逻辑中，**状态没有满足的和迟到的数据，都会存储在一个 Map 数据结构中**，也就是说，如果我们限定判断事件 序列的时长为5 分钟，那么内存中就会存储 5 分钟的数据，这在我看来，也是对内存的极大损伤之一。

## 12、三种时间语义
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：Flink 三种时间语义是什么，分别说出应用场景？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：

- **Event Time**：这是实际应用最常见的时间语义，指的是事件创建的时间，**往往跟watermark结合使用**

- **Processing Time**：指每一个执行基于时间操作的算子的本地系统时间，与机器相关。适用场景：没有事件时间的情况下，或者**对实时性要求超高**的情况
- **Ingestion Time**：指数据进入Flink的时间。适用场景：**存在多个 Source Operator** 的情况下，每个 Source Operator 可以使用自己本地系统时钟指派 Ingestion Time。后续基于时间相关的各种操作， 都会使用数据记录中的 Ingestion Time


## 13、数据高峰的处理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**问题**：Flink 程序在面对数据高峰期时如何处理？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**解答**：使用大容量的 Kafka 把数据先放到消息队列里面作为数据源，再使用 Flink 进行消费，不过这样会影响到一点实时性。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本次分享的Flink面试题难度不大，但足够考验一个数据工程师对于Flink的基本认知水平。要想职业道路好走，就得跟上技术发展的潮流！本期文章就到这里，后期会为大家分享更多干货内容，敬请期待！你知道的越多，你不知道的也越多，我是Alice，我们下一期见！


 ## 彩蛋
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了能鼓励大家多学会总结，菌在这里贴上自己平时做的思维导图，需要的朋友，可以关注博主个人微信公众号【猿人菌】，后台回复“**思维导图**”即可获取。

![](https://img-blog.csdnimg.cn/20201015101722118.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;更有海量面经奉送
![](https://img-blog.csdnimg.cn/20210126132608386.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
![](https://img-blog.csdnimg.cn/20200920214945468.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center =200x200)
<center>
<marquee>
<b>
     <font color="#4169E1">关注即可获取高质量思维导图，互联网一线大厂面经，大数据珍藏精品书籍...期待您的关注!</font>





