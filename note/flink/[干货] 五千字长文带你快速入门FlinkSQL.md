## 一、前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最近几天因为工作比较忙，已经几天没有及时更新文章了，在这里先给小伙伴们说声抱歉...临近周末，再忙再累，我也要开始发力了。接下来的几天，菌哥将为大家带来关于**FlinkSQL**的教程，之后还会更新一些大数据实时数仓的内容，和一些热门的组件使用！希望小伙伴们能点个关注，第一时间关注技术干货！

***

## 二、FlinkSQL出现的背景
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='	Tomato'>**Flink SQL 是 Flink 实时计算为简化计算模型，降低用户使用实时计算门槛而设计的一套符合标准 SQL 语义的开发语言。**</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;自 2015 年开始，阿里巴巴开始调研开源流计算引擎，最终决定基于 Flink 打造新一代计算引擎，针对 Flink 存在的不足进行优化和改进，并且在 2019 年初将最终代码开源，也就是我们熟知的 Blink。Blink 在原来的 Flink 基础上最显著的一个贡献就是 Flink SQL 的实现。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink SQL 是面向用户的 API 层，在我们传统的流式计算领域，比如 **<font color='BlueViolet'>Storm、Spark Streaming 都会提供一些 Function 或者 Datastream API，用户通过 Java 或 Scala 写业务逻辑，这种方式虽然灵活，但有一些不足，比如具备一定门槛且调优较难</font>**，随着版本的不断更新，API 也出现了很多**不兼容**的地方。

![](https://img-blog.csdnimg.cn/20210116105810976.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;



​        在这个背景下，毫无疑问，SQL 就成了我们最佳选择，之所以选择将 SQL 作为核心 API，是因为其具有几个非常重要的特点：

 - **SQL 属于设定式语言，用户只要表达清楚需求即可，不需要了解具体做法**；
 - **SQL 可优化，内置多种查询优化器，这些查询优化器可为 SQL 翻译出最优执行计划**；
 - **SQL 易于理解，不同行业和领域的人都懂，学习成本较低**；
 - **SQL 非常稳定，在数据库 30 多年的历史中，SQL 本身变化较少**；
 - **流与批的统一，Flink 底层 Runtime 本身就是一个流与批统一的引擎，而 SQL 可以做到 API 层的流与批统一**。


## 三、整体介绍
### 3.1 什么是 Table API 和 Flink SQL?
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink本身是批流统一的处理框架，所以**Table API和SQL，就是批流统一的上层处理API**。目前功能尚未完善，处于活跃的开发阶段。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Table API是一套内嵌在Java和Scala语言中的查询API**，它允许我们以非常直观的方式，组合来自一些关系运算符的查询（比如select、filter和join）。而对于**Flink SQL，就是直接可以在代码中写SQL，来实现一些查询（Query）操作**。Flink的SQL支持，基于实现了SQL标准的Apache Calcite（Apache开源SQL解析工具）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;无论输入是批输入还是流式输入，在这两套API中，指定的查询都具有相同的语义，得到相同的结果。

### 3.2 需要引入的依赖
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table API 和 SQL 需要引入的依赖有两个：`planner`  和 `bridge`

```xml
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-planner_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-table-api-scala-bridge_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**flink-table-planner**：<font color='Tomato'>**planner计划器，是table API最主要的部分，提供了运行时环境和生成程序执行计划的planner**</font>；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**flink-table-api-scala-bridge**：<font color='Tomato'>**bridge桥接器，主要负责table API和 DataStream/DataSet API的连接支持，按照语言分java和scala**</font>；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里的两个依赖，是IDE环境下运行需要添加的；如果是生产环境，lib目录下默认已经有了planner，就只需要有bridge就可以了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然，如果想使用用户自定义函数，或是跟 kafka 做连接，需要有一个SQL client，这个包含在 `flink-table-common` 里。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
### 3.3 两种planner（old & blink）的区别
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1、**批流统一：Blink将批处理作业，视为流式处理的特殊情况**。所以，blink不支持表和DataSet之间的转换，批处理作业将不转换为DataSet应用程序，而是跟流处理一样，转换为DataStream程序来处理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2、**因为批流统一，Blink planner也不支持BatchTableSource，而使用有界的StreamTableSource代替**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3、Blink planner只支持全新的目录，不支持已弃用的ExternalCatalog。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4、旧 planner 和 Blink planner 的FilterableTableSource实现不兼容。旧的planner会把PlannerExpressions下推到filterableTableSource中，而blink planner则会把Expressions下推。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5、基于字符串的键值配置选项仅适用于Blink planner。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6、PlannerConfig在两个planner中的实现不同。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;7、Blink planner会将多个sink优化在一个DAG中（仅在TableEnvironment上受支持，而在StreamTableEnvironment上不受支持）。而旧 planner 的优化总是将每一个sink放在一个新的DAG中，其中所有DAG彼此独立。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;8、旧的planner不支持目录统计，而Blink planner支持。

## 四、API 调用
### 4.1 基本程序结构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Table API 和 SQL 的程序结构，与流式处理的程序结构类似**；也可以近似地认为有这么几步：首先创建执行环境，然后定义source、transform和sink。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;具体操作流程如下：

```scala
val tableEnv = ...     // 创建表的执行环境

// 创建一张表，用于读取数据
tableEnv.connect(...).createTemporaryTable("inputTable")
// 注册一张表，用于把计算结果输出
tableEnv.connect(...).createTemporaryTable("outputTable")

// 通过 Table API 查询算子，得到一张结果表
val result = tableEnv.from("inputTable").select(...)
// 通过 SQL查询语句，得到一张结果表
val sqlResult  = tableEnv.sqlQuery("SELECT ... FROM inputTable ...")

// 将结果表写入输出表中
result.insertInto("outputTable")
```

### 4.2 创建表环境
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;创建表环境最简单的方式，就是基于流处理执行环境，调create方法直接创建：

```scala
val tableEnv = StreamTableEnvironment.create(env)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表环境（**TableEnvironment**）是flink中集成 Table API & SQL 的**核心概念**。它负责:

 - 注册catalog
 - 在内部 catalog 中注册表
 - 执行 SQL 查询
 - 注册用户自定义函数
 - 将 DataStream 或 DataSet 转换为表
 - 保存对 ExecutionEnvironment 或 StreamExecutionEnvironment  的引用

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在创建TableEnv的时候，可以多传入一个EnvironmentSettings 或者 TableConfig 参数，可以用来配置 TableEnvironment 的一些特性。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如，配置老版本的流式查询（Flink-Streaming-Query）： 

```scala
val settings = EnvironmentSettings.newInstance()
  .useOldPlanner()      // 使用老版本planner
  .inStreamingMode()    // 流处理模式
  .build()
val tableEnv = StreamTableEnvironment.create(env, settings)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于老版本的批处理环境（Flink-Batch-Query）：

```scala
val batchEnv = ExecutionEnvironment.getExecutionEnvironment
val batchTableEnv = BatchTableEnvironment.create(batchEnv)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于 blink 版本的流处理环境（Blink-Streaming-Query）：

```scala
val bsSettings = EnvironmentSettings.newInstance()
.useBlinkPlanner()
.inStreamingMode().build()
val bsTableEnv = StreamTableEnvironment.create(env, bsSettings)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于blink版本的批处理环境（Blink-Batch-Query）：

```scala
val bbSettings = EnvironmentSettings.newInstance()
.useBlinkPlanner()
.inBatchMode().build()
val bbTableEnv = TableEnvironment.create(bbSettings)
```

### 4.3 在Catalog中注册表
#### 4.3.1 表(Table)的概念
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TableEnvironment 可以注册目录 Catalog ，并可以基于Catalog注册表。它会维护一个 Catalog-Table 表之间的map。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表（Table）是由一个“标识符”来指定的，由3部分组成：**Catalog名、数据库（database）名和对象名（表名**）。如果没有指定目录或数据库，就使用当前的默认值。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表可以是常规的（Table，表），或者虚拟的（View，视图）。**常规表（Table）一般可以用来描述外部数据，比如文件、数据库表或消息队列的数据，也可以直接从 DataStream转换而来。视图可以从现有的表中创建，通常是 table API 或者SQL查询的一个结果**。 

#### 4.3.2 连接到文件系统（Csv格式）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;连接外部系统在Catalog中注册表，直接调用 **tableEnv.connect()** 就可以，里面参数要传入一个 ConnectorDescriptor ，也就是connector描述器。对于文件系统的 connector 而言，flink内部已经提供了，就叫做**FileSystem()**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
tableEnv
.connect( new FileSystem().path("sensor.txt"))  // 定义表数据来源，外部连接
  .withFormat(new OldCsv())    // 定义从外部系统读取数据之后的格式化方法
  .withSchema( new Schema()
    .field("id", DataTypes.STRING())
    .field("timestamp", DataTypes.BIGINT())
    .field("temperature", DataTypes.DOUBLE())
  )    // 定义表结构
  .createTemporaryTable("inputTable")    // 创建临时表
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是旧版本的csv格式描述器。由于它是非标的，跟外部系统对接并不通用，所以将被弃用，以后会被一个符合RFC-4180标准的新format描述器取代。**新的描述器就叫Csv()，但flink没有直接提供，需要引入依赖flink-csv**：

```scala
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-csv</artifactId>
    <version>1.10.0</version>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码非常类似，只需要把 withFormat 里的 OldCsv 改成Csv就可以了。

#### 4.3.3 连接到Kafka
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;kafka的连接器 flink-kafka-connector 中，1.10 版本的已经提供了 Table API 的支持。我们可以在 connect方法中直接传入一个叫做Kafka的类，这就是kafka连接器的描述器ConnectorDescriptor。

```scala
tableEnv.connect(
  new Kafka()
    .version("0.11") // 定义kafka的版本
    .topic("sensor") // 定义主题
    .property("zookeeper.connect", "localhost:2181") 
    .property("bootstrap.servers", "localhost:9092")
)
  .withFormat(new Csv())
  .withSchema(new Schema()
  .field("id", DataTypes.STRING())
  .field("timestamp", DataTypes.BIGINT())
  .field("temperature", DataTypes.DOUBLE())
)
  .createTemporaryTable("kafkaInputTable")
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然也可以连接到 ElasticSearch、MySql、HBase、Hive等外部系统，实现方式基本上是类似的。感兴趣的 小伙伴可以自行去研究，这里就不详细赘述了。

### 4.4 表的查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过上面的学习，我们已经利用外部系统的连接器connector，我们可以读写数据，并在环境的Catalog中注册表。接下来就可以对表做**查询转换**了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink给我们提供了两种查询方式：Table API和 SQL。

####  4.4.1 Table API的调用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table API是集成在Scala和Java语言内的查询API。与SQL不同，Table API的查询不会用字符串表示，而是在宿主语言中一步一步调用完成的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Table API基于代表一张“表”的Table类，并提供一整套操作处理的方法API**。这些方法会返回一个新的Table对象，这个对象就表示对输入表应用转换操作的结果。有些关系型转换操作，可以由多个方法调用组成，构成链式调用结构。例如`table.select(…).filter(…)`，其中 select（…）表示选择表中指定的字段，filter(…)表示筛选条件。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码中的实现如下：

```scala
val sensorTable: Table = tableEnv.from("inputTable")

val resultTable: Table = senorTable
.select("id, temperature")
.filter("id ='sensor_1'")
```

#### 4.4.2 SQL查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**Flink的SQL集成，基于的是ApacheCalcite，它实现了SQL标准**。在Flink中，用常规字符串来定义SQL查询语句。SQL 查询的结果，是一个新的 Table。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码实现如下：

```scala
val resultSqlTable: Table = tableEnv.sqlQuery("select id, temperature from inputTable where id ='sensor_1'")
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;或者：

```scala
val resultSqlTable: Table = tableEnv.sqlQuery(
  """
    |select id, temperature
    |from inputTable
    |where id = 'sensor_1'
  """.stripMargin)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然，也可以加上聚合操作，比如我们统计每个sensor温度数据出现的个数，做个count统计：

```scala
val aggResultTable = sensorTable
.groupBy('id)
.select('id, 'id.count as 'count)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SQL的实现：

```scala
val aggResultSqlTable = tableEnv.sqlQuery("select id, count(id) as cnt from inputTable group by id")
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这里Table API里指定的字段，前面加了一个单引号’，这是Table API中定义的Expression类型的写法，可以很方便地表示一个表中的字段。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;字段可以直接全部用双引号引起来，也可以用**半边单引号+字段名**的方式。以后的代码中，一般都用后一种形式。

### 4.5 将DataStream 转换成表
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='DeepPink'>Flink允许我们把Table和DataStream做转换</font>：我们可以基于一个DataStream，先流式地读取数据源，然后map成样例类，再把它转成Table。Table的列字段（column fields），就是样例类里的字段，这样就不用再麻烦地定义schema了。

#### 4.5.1 代码表达
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码中实现非常简单，直接用 tableEnv.fromDataStream() 就可以了。默认转换后的 Table schema 和 DataStream 中的字段定义一一对应，也可以单独指定出来。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这就允许我们更换字段的顺序、重命名，或者只选取某些字段出来，相当于做了一次map操作（或者Table API的 select操作）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码具体如下：

```scala
val inputStream: DataStream[String] = env.readTextFile("sensor.txt")
val dataStream: DataStream[SensorReading] = inputStream
  .map(data => {
    val dataArray = data.split(",")
    SensorReading(dataArray(0), dataArray(1).toLong, dataArray(2).toDouble)
  })

val sensorTable: Table = tableEnv.fromDataStreama(datStream)

val sensorTable2 = tableEnv.fromDataStream(dataStream, 'id, 'timestamp as 'ts)
```

#### 4.5.2 数据类型与 Table schema的对应
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上节的例子中，DataStream 中的数据类型，与表的 Schema 之间的对应关系，是按照样例类中的字段名来对应的（name-based mapping），所以还可以用as做重命名。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外一种对应方式是，直接按照字段的位置来对应（position-based mapping），对应的过程中，就可以直接指定新的字段名了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于名称的对应：

```scala
val sensorTable = tableEnv.fromDataStream(dataStream, 'timestamp as 'ts, 'id as 'myId, 'temperature)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于位置的对应：

```scala
val sensorTable = tableEnv.fromDataStream(dataStream, 'myId, 'ts)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink的 DataStream 和 DataSet API 支持多种类型。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;组合类型，比如元组（内置Scala和Java元组）、POJO、Scala case类和Flink的Row类型等，允许具有多个字段的嵌套数据结构，这些字段可以在Table的表达式中访问。其他类型，则被视为原子类型。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;元组类型和原子类型，一般用位置对应会好一些；如果非要用名称对应，也是可以的：元组类型，默认的名称是 “_1”, “_2”；而原子类型，默认名称是 ”f0”。

### 4.6 创建临时视图（Temporary View）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;创建临时视图的第一种方式，就是直接从DataStream转换而来。同样，可以直接对应字段转换；也可以在转换的时候，指定相应的字段。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
tableEnv.createTemporaryView("sensorView", dataStream)
tableEnv.createTemporaryView("sensorView", dataStream, 'id, 'temperature, 'timestamp as 'ts)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外，当然还可以基于Table创建视图：

```scala
tableEnv.createTemporaryView("sensorView", sensorTable)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;View和Table的Schema完全相同。事实上，在Table API中，可以认为View 和 Table 是等价的。

### 4.7 输出表
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表的输出，是通过将数据写入 TableSink 来实现的。**TableSink 是一个通用接口，可以支持不同的文件格式、存储数据库和消息队列**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;具体实现，输出表最直接的方法，就是**通过 Table.insertInto() 方法将一个 Table 写入注册过的 TableSink 中**。

#### 4.7.1 输出到文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
// 注册输出表
tableEnv.connect(
  new FileSystem().path("…\\resources\\out.txt")
) // 定义到文件系统的连接
  .withFormat(new Csv()) // 定义格式化方法，Csv格式
  .withSchema(new Schema()
  .field("id", DataTypes.STRING())
  .field("temp", DataTypes.DOUBLE())
) // 定义表结构
  .createTemporaryTable("outputTable") // 创建临时表

resultSqlTable.insertInto("outputTable")
```
#### 4.7.2 更新模式（Update Mode）
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>在流处理过程中，表的处理并不像传统定义的那样简单。</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于流式查询（Streaming Queries），**需要声明如何在（动态）表和外部连接器之间执行转换**。与外部系统交换的消息类型，由<font color='purple'>**更新模式（update mode）**</font>指定。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink Table API中的更新模式有以下三种：

 - **追加模式（Append Mode）**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在追加模式下，表（动态表）和外部连接器只交换插入（Insert）消息。

- **撤回模式（Retract Mode）**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在撤回模式下，表和外部连接器交换的是：添加（Add）和撤回（Retract）消息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中：

> - <font color='black'>**插入（Insert）会被编码为添加消息；**</font>
> - <font color='black'>**删除（Delete）则编码为撤回消息；**
> - <font color='black'>**更新（Update）则会编码为，已更新行（上一行）的撤回消息，和更新行（新行）的添加消息。**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在此模式下，不能定义key，这一点跟upsert模式完全不同。

 - **Upsert（更新插入）模式**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在Upsert模式下，动态表和外部连接器交换Upsert和Delete消息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个模式需要一个唯一的key，通过这个key可以传递更新消息。为了正确应用消息，外部连接器需要知道这个唯一key的属性。

>- <font color='black'>**插入（Insert）和更新（Update）都被编码为Upsert消息；**</font>
>- <font color='black'>**删除（Delete）编码为Delete信息**</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这种模式和 Retract 模式的主要区别在于，Update操作是用单个消息编码的，所以效率会更高。

#### 4.7.3 输出到Kafka
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;除了输出到文件，也可以输出到Kafka。我们可以结合前面Kafka作为输入数据，构建数据管道，kafka进，kafka出。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
// 输出到 kafka
tableEnv.connect(
  new Kafka()
    .version("0.11")
    .topic("sinkTest")
    .property("zookeeper.connect", "localhost:2181")
    .property("bootstrap.servers", "localhost:9092")
)
  .withFormat( new Csv() )
  .withSchema( new Schema()
    .field("id", DataTypes.STRING())
    .field("temp", DataTypes.DOUBLE())
  )
  .createTemporaryTable("kafkaOutputTable")

resultTable.insertInto("kafkaOutputTable")
```
#### 4.7.4 输出到ElasticSearch
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ElasticSearch的connector可以在upsert（update+insert，更新插入）模式下操作，这样就可以使用Query定义的键（key）与外部系统交换UPSERT/DELETE消息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外，对于“仅追加”（append-only）的查询，connector还可以在 append 模式下操作，这样就可以与外部系统只交换 insert 消息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;es目前支持的数据格式，只有Json，而 flink 本身并没有对应的支持，所以还需要引入依赖：

```scala
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-json</artifactId>
    <version>1.10.0</version>
</dependency>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码实现如下：

```scala
// 输出到es
tableEnv.connect(
  new Elasticsearch()
    .version("6")
    .host("localhost", 9200, "http")
    .index("sensor")
    .documentType("temp")
)
  .inUpsertMode()           // 指定是 Upsert 模式
  .withFormat(new Json())
  .withSchema( new Schema()
    .field("id", DataTypes.STRING())
    .field("count", DataTypes.BIGINT())
  )
  .createTemporaryTable("esOutputTable")

aggResultTable.insertInto("esOutputTable")
```

#### 4.7.5 输出到MySql
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flink专门为Table API的jdbc连接提供了flink-jdbc连接器，我们需要先引入依赖：

```scala
<dependency>
    <groupId>org.apache.flink</groupId>
    <artifactId>flink-jdbc_2.11</artifactId>
    <version>1.10.0</version>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;jdbc连接的代码实现比较特殊，因为没有对应的java/scala类实现 `ConnectorDescriptor`，所以不能直接 `tableEnv.connect()`。不过Flink SQL留下了执行DDL的接口：`tableEnv.sqlUpdate()`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于jdbc的创建表操作，天生就适合直接写DDL来实现，所以我们的代码可以这样写：

```scala
// 输出到 Mysql
val sinkDDL: String =
  """
    |create table jdbcOutputTable (
    |  id varchar(20) not null,
    |  cnt bigint not null
    |) with (
    |  'connector.type' = 'jdbc',
    |  'connector.url' = 'jdbc:mysql://localhost:3306/test',
    |  'connector.table' = 'sensor_count',
    |  'connector.driver' = 'com.mysql.jdbc.Driver',
    |  'connector.username' = 'root',
    |  'connector.password' = '123456'
    |)
  """.stripMargin

tableEnv.sqlUpdate(sinkDDL)
aggResultSqlTable.insertInto("jdbcOutputTable")
```
#### 4.7.6 将表转换成DataStream
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**表可以转换为DataStream或DataSet**。这样，自定义流处理或批处理程序就可以继续在 Table API或SQL查询的结果上运行了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**将表转换为DataStream或DataSet时，需要指定生成的数据类型**，即要将表的每一行转换成的数据类型。通常，**最方便的转换类型就是Row**。当然，因为结果的所有字段类型都是明确的，我们也经常会用元组类型来表示。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表作为流式查询的结果，是**动态更新**的。所以，将这种动态查询转换成的数据流，同样需要对表的更新操作进行编码，进而有不同的转换模式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table API 中表到 DataStream 有两种模式：


 - **追加模式（Append Mode）**


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用于表只会被插入（Insert）操作更改的场景

- **撤回模式（Retract Mode）**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用于任何场景。有些类似于更新模式中Retract模式，它只有 Insert 和 Delete 两类操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='DarkOrchid'>得到的数据会增加一个Boolean类型的标识位（返回的第一个字段），用它来表示到底是新增的数据（Insert），还是被删除的数据（老数据， Delete）</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码实现如下：

```scala
val resultStream: DataStream[Row] = tableEnv.toAppendStream[Row](resultTable)

val aggResultStream: DataStream[(Boolean, (String, Long))] = 
tableEnv.toRetractStream[(String, Long)](aggResultTable)

resultStream.print("result")
aggResultStream.print("aggResult")
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='HotPink'>所以，没有经过groupby之类聚合操作，可以直接用 toAppendStream 来转换；而如果经过了聚合，有更新操作，一般就必须用 toRetractDstream。
</font>

####  4.7.7 Query的解释和执行
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Table API提供了一种机制来解释（Explain）**计算表的逻辑和优化查询计划**。这是通过TableEnvironment.explain（table）方法或TableEnvironment.explain（）方法完成的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;explain方法会返回一个字符串，描述三个计划：

 - 未优化的逻辑查询计划
 - 优化后的逻辑查询计划
 - 实际执行计划

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们可以在代码中查看执行计划：

```scala
val explaination: String = tableEnv.explain(resultTable)
println(explaination)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Query的解释和执行过程，老planner和 blink planner 大体是一致的，又有所不同。整体来讲，Query都会表示成一个逻辑查询计划，然后分两步解释：

 1. 优化查询计划
 2. 解释成 DataStream 或者 DataSet程序

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而 Blink 版本是批流统一的，所以所有的Query，只会被解释成DataStream程序；另外在批处理环境 TableEnvironment 下，Blink版本要到 **tableEnv.execute()** 执行调用才开始解释。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;

![](https://img-blog.csdnimg.cn/20210116195814196.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center =300x250)

## 巨人的肩膀

> 1、http://www.atguigu.com/    
> 2、https://www.bilibili.com/video/BV12k4y1z7LM?from=search&seid=953051020130358915
> 3、https://blog.csdn.net/u013411339/article/details/93267838

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇文章主要用五千多字，为大家带来迅速入门并掌握 FlinkSQL 的技巧，包含FlinkSQL出现的背景介绍以及与 Table API 的区别，API调用方式更是介绍的非常详细全面，希望小伙伴们在看了之后能够及时复习总结，尤其是初学者。好了，本篇文章 over，大家看了之后有任何的疑惑都可以私信作者，我看到都会一一解答。下一篇我会在本篇的基础上为大家介绍一些**流处理中的特殊概念**，敬请期待|ू･ω･` )，你知道的越多，你不知道的也越多，我是Alice，我们下一期见！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> **文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…关注这个在大数据领域冉冉升起的新星！**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![](https://img-blog.csdnimg.cn/20210119222335538.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

