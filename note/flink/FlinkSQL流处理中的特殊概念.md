## 一、前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一篇文章，为大家介绍了关于 FlinkSQL 的背景，常见使用以及一些小技巧。学完之后，对于FlinkSQL只能算是简单入了个门。不过不用担心，本篇文章，博主将为大家带来关于 **FlinkSQL中流处理的特殊概念**，喜欢的话，记得看完点个赞|ू･ω･` )

***

## 二、流处理中的特殊概念
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Tomato'>Table API和SQL，本质上还是基于关系型表的操作方式；而关系型表、关系代数，以及SQL本身，一般是有界的，更适合批处理的场景。这就导致在进行流处理的过程中，理解会稍微复杂一些，需要引入一些特殊概念</font>

### 2.1 流处理和关系代数（表，及SQL）的区别
|  | 关系代数（表）/SQL |流处理
|--|--|--|
| 处理的数据对象 | 字段元组的有界集合 |字段元组的无限序列
查询（Query）对数据的访问|可以访问到完整的数据输入|无法访问所有数据，必须持续“等待”流式输入|
查询终止条件|生成固定大小的结果集后终止|永不停止，根据持续收到的数据不断更新查询结果|

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以看到，其实关系代数（主要就是指关系型数据库中的表）和SQL，主要就是针对批处理的，这和流处理有天生的隔阂。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
### 2.2 动态表（Dynamic Tables）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为流处理面对的数据，是连续不断的，这和我们熟悉的关系型数据库中保存的“表”完全不同。所以，**如果我们把流数据转换成Table，然后执行类似于table的 select 操作，结果就不是一成不变的，而是随着新数据的到来，会不停更新**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='		Tomato'>我们可以随着新数据的到来，不停地在之前的基础上更新结果。这样得到的表，在Flink Table API  概念里，就叫做 “动态表” （Dynamic Tables）</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**动态表是 Flink 对流数据的 Table API 和 SQL 支持的核心概念**。与表示批处理数据的静态表不同，动态表是随时间变化的。动态表可以像静态的批处理表一样进行查询，查询一个动态表会产生持续查询（Continuous Query）。**连续查询永远不会终止，并会生成另一个动态表。查询（Query）会不断更新其动态结果表，以反映其动态输入表上的更改**。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
### 2.3 流式持续查询的过程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下图显示了流、动态表和连续查询的关系：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117101717553.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;流式持续查询的过程为：

 1. 流被转换为动态表
 2. 对动态表计算连续查询，生成新的动态表
 3. 生成的动态表被转换回流

#### 2.3.1 将流转换成表（Table）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**为了处理带有关系查询的流，必须先将其转换为表**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从概念上讲，流的每个数据记录，都被解释为对结果表的插入（Insert）修改。因为流是持续不断的，而且之前的输出结果无法改变。本质上，我们其实是从一个、只有插入操作的 changelog（更新日志）流，来构建一个表

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了更好地说明动态表和持续查询的概念，我们来举一个具体的例子

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如，我们现在的输入数据，就是用户在网站上的访问行为，数据类型（Schema）如下：

```json
[
  user:  VARCHAR,   // 用户名
  cTime: TIMESTAMP, // 访问某个URL的时间戳
  url:   VARCHAR    // 用户访问的URL
]
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下图显示了如何将访问URL事件流，或者叫点击事件流（左侧）转换为表（右侧）。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117101916237.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;随着插入更多的访问事件流记录，生成的表将不断增长。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
#### 2.3.2 持续查询（Continuous Query）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;持续查询，会在动态表上做计算处理，并作为结果生成新的动态表。与批处理查询不同，连续查询从不终止，并根据输入表上的更新更新其结果表。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在任何时间点，连续查询的结果在语义上，等同于在输入表的快照上，以批处理模式执行的同一查询的结果。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在下面的示例中，我们展示了对点击事件流中的一个持续查询。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个Query很简单，是一个分组聚合做 count 统计的查询。它将用户字段上的 clicks 表分组，并统计访问的 url 数。图中显示了随着时间的推移，当 clicks 表被其他行更新时如何计算查询。

![](https://img-blog.csdnimg.cn/20210117102140746.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
#### 2.3.3 将动态表转换成流
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;与常规的数据库表一样，动态表可以通过插入（Insert）、更新（Update）和删除（Delete）更改，进行持续的修改。<font color='	Tomato'>将动态表转换为流或将其写入外部系统时，需要对这些更改进行编码</font>。Flink的Table API和SQL支持三种方式对动态表的更改进行编码：

 - **仅追加（Append-only）流**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;仅通过插入（Insert）更改，来修改的动态表，可以直接转换为“仅追加”流。这个流中发出的数据，就是动态表中新增的每一行。

 - **撤回（Retract）流**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Retract流是包含两类消息的流，添加（Add）消息和撤回（Retract）消息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;动态表通过将 INSERT 编码为 add 消息、DELETE 编码为retract消息、UPDATE 编码为被更改行（前一行）的 retract 消息和更新后行（新行）的 add 消息，转换为 retract 流。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下图显示了将动态表转换为 Retract 流的过程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210117102643561.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
- **Upsert（更新插入）流**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Upsert 流包含两种类型的消息：Upsert 消息和 delete 消息。转换为 upsert 流的动态表，需要有唯一的键（key）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过将 INSERT 和 UPDATE 更改编码为 upsert 消息，将DELETE更改编码为DELETE消息，就可以将具有唯一键（Unique Key）的动态表转换为流。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下图显示了将动态表转换为 upsert 流的过程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/2021011710302610.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这些概念我们之前都已提到过。需要注意的是，在代码里将动态表转换为DataStream时，仅支持 Append 和Retract流 。而向外部系统输出动态表的TableSink接口，则可以有不同的实现，比如之前我们讲到的ES，就可以有Upsert模式。

### 2.4 时间特性
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于时间的操作（比如 Table API 和 SQL 中窗口操作），需要定义相关的时间语义和时间数据来源的信息。所以，**Table可以提供一个逻辑上的时间字段，用于在表处理程序中，指示时间和访问相应的时间戳**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时间属性，可以是每个表 schema 的一部分。一旦定义了时间属性，它就可以作为一个字段引用，并且可以在基于时间的操作中使用。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;时间属性的行为类似于常规时间戳，可以访问，并且进行计算。

#### 2.4.1 处理时间（Processing Time）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;处理时间语义下，允许表处理程序根据机器的本地时间生成结果。它是时间的最简单概念。它既不需要提取时间戳，也不需要生成watermark。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;定义处理时间属性有三种方法：在 DataStream 转化时直接指定；在定义 Table Schema 时指定；在创建表的 DDL 中指定。

##### 2.4.1.1 DataStream转化成Table时指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由 DataStream 转换成表时，可以在后面指定字段名来定义Schema。在定义 Schema 期间，可以使用 `.proctime` ，定义处理时间字段。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意，这个 proctime 属性只能通过附加逻辑字段，来扩展物理schema 。因此，只能在 schema 定义的末尾定义它。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
// 定义好 DataStream
val inputStream: DataStream[String] = env.readTextFile("\\sensor.txt")
val dataStream: DataStream[SensorReading] = inputStream
  .map(data => {
    val dataArray = data.split(",")
    SensorReading(dataArray(0), dataArray(1).toLong, dataArray(2).toDouble)
  })

// 将 DataStream转换为 Table，并指定时间字段
val sensorTable = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'timestamp, 'pt.proctime)
```

##### 2.4.1.2 定义Table Schema 时指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种方法其实也很简单，只要在定义 Schema 的时候，加上一个新的字段，并指定成 proctime 就可以了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
tableEnv.connect(
  new FileSystem().path("..\\sensor.txt"))
  .withFormat(new Csv())
  .withSchema(new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
    .field("temperature", DataTypes.DOUBLE())
    .field("pt", DataTypes.TIMESTAMP(3))
      .proctime()    // 指定 pt字段为处理时间
  ) // 定义表结构
  .createTemporaryTable("inputTable") // 创建临时表
```
##### 2.4.1.3 创建表的DDL中指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在创建表的DDL中，增加一个字段并指定成 proctime，也可以指定当前的时间字段。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
val sinkDDL: String =
  """
    |create table dataTable (
    |  id varchar(20) not null,
    |  ts bigint,
    |  temperature double,
    |  pt AS PROCTIME()
    |) with (
    |  'connector.type' = 'filesystem',
    |  'connector.path' = 'file:///D:\\..\\sensor.txt',
    |  'format.type' = 'csv'
    |)
  """.stripMargin

tableEnv.sqlUpdate(sinkDDL) // 执行 DDL
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>注意：运行这段DDL，必须使用Blink Planner</font>


#### 2.4.2 事件时间（Event Time）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**事件时间语义，允许表处理程序根据每个记录中包含的时间生成结果**。这样即使在有乱序事件或者延迟事件时，也可以获得正确的结果。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了处理无序事件，并区分流中的准时和迟到事件；Flink需要从事件数据中，提取时间戳，并用来推进事件时间的进展（watermark）。

##### 2.4.2.1 DataStream转化成Table时指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在 DataStream 转换成 Table，schema 的定义期间，使用 .rowtime 可以定义事件时间属性。注意，<font color='	Tomato'>必须在转换的数据流中分配时间戳和watermark</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在将数据流转换为表时，有两种定义时间属性的方法。根据指定的 `.rowtime` 字段名是否存在于数据流的架构中，timestamp 字段可以：

 - 作为新字段追加到schema
 - 替换现有字段

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在这两种情况下，定义的事件时间戳字段，都将保存 DataStream 中事件时间戳的值。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
val inputStream: DataStream[String] = env.readTextFile("\\sensor.txt")
val dataStream: DataStream[SensorReading] = inputStream
    .map(data => {
        val dataArray = data.split(",")
        SensorReading(dataArray(0), dataArray(1).toLong, dataArray(2).toDouble)
      })
    .assignAscendingTimestamps(_.timestamp * 1000L)

// 将 DataStream转换为 Table，并指定时间字段
val sensorTable = tableEnv.fromDataStream(dataStream, 'id, 'timestamp.rowtime, 'temperature)
// 或者，直接追加字段
val sensorTable2 = tableEnv.fromDataStream(dataStream, 'id, 'temperature, 'timestamp, 'rt.rowtime)
```

##### 2.4.2.2 定义Table Schema时指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种方法只要在定义 Schema 的时候，将事件时间字段，并指定成 `rowtime` 就可以了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
tableEnv.connect(new FileSystem().path("sensor.txt"))
      .withFormat(new Csv())
      .withSchema(new Schema()
        .field("id", DataTypes.STRING())
        .field("timestamp", DataTypes.BIGINT())
        .rowtime(new Rowtime()
          .timestampsFromField("timestamp") // 从字段中提取时间戳
          .watermarksPeriodicBounded(1000) // watermark 延迟 1 秒 )
        )
        .field("temperature", DataTypes.DOUBLE())) // 定义表结构
      .createTemporaryTable("inputTable") // 创建临时表
```
##### 2.4.3 创建表的 DDL 中指定
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Tomato'>事件时间属性，是使用 watermark 语句，定义现有事件时间字段上的 watermark 生成表达式，该表达式将事件时间字段标记为事件时间属性</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
val sinkDDL: String =
"""
|create table dataTable (
|  id varchar(20) not null,
|  ts bigint,
|  temperature double,
|  rt AS TO_TIMESTAMP( FROM_UNIXTIME(ts) ),
|  watermark for rt as rt - interval '1' second
|) with (
|  'connector.type' = 'filesystem',
|  'connector.path' = 'file:///D:\\..\\sensor.txt',
|  'format.type' = 'csv'
|)
""".stripMargin
tableEnv.sqlUpdate(sinkDDL) // 执行 DDL
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里 `FROM_UNIXTIME` 是系统内置的时间函数，用来将一个整数（秒数）转换成“`YYYY-MM-DD hh:mm:ss`”格式（默认，也可以作为第二个String参数传入）的日期时间字符串（date time string）；然后再用`TO_TIMESTAMP`将其转换成`Timestamp`

## 巨人的肩膀
> 1、http://www.atguigu.com/
> 2、https://www.bilibili.com/video/BV12k4y1z7LM?from=search&seid=953051020130358915
> 3、https://blog.csdn.net/u013411339/article/details/93267838

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇文章在前一篇关于FlinkSQL的基础之上，引入了流处理中的一些特殊概念，如果没有Flink基础的同学可能会理解起来比较吃力，建议去看看菌哥之前写的文章或者私信笔者具体的疑惑。**学习时间语义，要配合窗口操作才能发挥作用**，下一篇文章，将为大家带来关于FlinkSQL窗口的具体内容，敬请期待 |ू･ω･` )你知道的越多，你不知道的也越多，我是Alice，我们下一期见！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

>**文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经等你来领取，顺便关注下这个在大数据领域冉冉升起的新星！**

 



<img src="https://img-blog.csdnimg.cn/20210119135940253.png" width = 99% height = 50% />