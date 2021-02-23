## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一篇文章已经为大家介绍了 MySQL 在用户画像的标签数据存储中的具体应用场景，本篇我们来谈谈 HBase 的使用！

![](https://img-blog.csdnimg.cn/20210222222026959.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
>**原著作者：赵宏田
来源：《用户画像方法论与工程化解决方案》**

## HBase存储

### 1. HBase简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='Tomato'>**HBase是一个高性能、列存储、可伸缩、实时读写的分布式存储系统**</font>，同样运行在HDFS之上。与Hive不同的是，HBase能够在数据库上实时运行，而不是跑MapReduce任务，适合进行大数据的**实时查询**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;画像系统中每天在Hive里跑出的结果集数据可同步到 HBase数据库 ，用于线上实时应用的场景。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面介绍几个基本概念：

 - **row key**：用来表示唯一一行记录的**主**键，HBase的数据是按照 row key 的**字典顺序**进行全局排列的。访问HBase中的行只有3种方式：
 - **通过单个row key访问**；
 - **通过row key的正则访问**；
 - **全表扫描**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于HBase通过 rowkey 对数据进行检索，而rowkey 由于长度限制的因素不能将很多查询条件拼接在 rowkey 中，因此 HBase 无法像关系数据库那样根据多种条件对数据进行筛选。一般地，HBase需建立**二级索引**来满足根据复杂条件查询数据的需求。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Rowkey设计时需要遵循三大原则：

- **唯一性原则**：rowkey需要保证唯一性，不存在重复的情况。在画像中一般使用用户id作为rowkey
- **长度原则**：rowkey的长度一般为10-100bytes
- **散列原则**：rowkey的散列分布有利于数据均衡分布在每个RegionServer，可实现负载均衡

--  columns family：指列簇，HBase中的每个列都归属于某个列簇。列簇是表的schema的一部分，必须在使用表之前定义。划分columns family的原则如下：
- 是否具有相似的数据格式
- 是否具有相似的访问类型

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;常用的增删改查命令如下：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）创建一个表，指定表名和列簇名：

```sql
create  '<table name>','<column family>'
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）扫描表中数据，并显示其中的10条记录：

```sql
scan  '<table name>',{LIMIT=>10}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3）使用get命令读取数据：

```sql
get  '<table name>','row1'
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;4）插入数据：

```sql
put  '<table name>','row1','<colfamily:colname>','<value>'
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;5）更新数据：

```sql
put  '<table name>','row ','Column family:column name','new value'
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;6）在删除表之前先将其**禁用**，然后删除：

```sql
disable  '<table name>'
drop  '<table name>'
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过一个案例来介绍HBase在画像系统中的应用场景和工程化实现方式。


### 2. 应用场景
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;渠道运营人员为促进未注册的新安装用户注册、下单，计划通过App首页弹窗（如下图所示）发放红包或优惠券的方式进行引导。在该场景中可通过画像系统实现对应功能。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;业务逻辑上，渠道运营人员通过组合用户标签（如“未注册用户”和“安装距今天数”小于××天）筛选出对应的用户群，然后选择将对应人群推送到“广告系统”，这样每天画像系统的ETL调度完成后对应人群数据就被推送到HBase数据库进行存储。满足条件的新用户来访App时，由在线接口读取HBase数据库，在查询到该用户时为其推送该弹窗。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过某工程案例来讲解HBase在该触达用户场景中的应用方式。
![](https://img-blog.csdnimg.cn/20210222223852955.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
### 3. 工程化案例
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**运营人员在画像系统中根据业务规则定义组合用户标签筛选出用户群，并将该人群上线到广告系统中**。

![](https://img-blog.csdnimg.cn/20210222224208179.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在业务人员配置好规则后，下面我们来看在数据调度层面是如何运行的。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用户标签数据经过ETL将每个用户身上的标签聚合后插入到目标表中，如`dw.userprofile_userlabel_map_all`。聚合后数据存储为每个用户id，以及他身上对应的标签集合，数据格式如图所示：

![](https://img-blog.csdnimg.cn/20210222230712227.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来需要将 Hive 中的数据导入HBase，便于线上接口实时调用库中数据。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**HBase的服务器体系结构遵循主从服务器架构**（如图所示），同一时刻只有一个**HMaster**处于活跃状态，当活跃的Master挂掉后，Backup HMaster自动接管整个HBase集群。在同步数据前，首先需要判断HBase的当前活跃节点是哪台机器。

![](https://img-blog.csdnimg.cn/20210222230813445.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行如下脚本：

```shell
# 判断活跃节点
global activenode
for node in ("10.xxx.xx.xxx","10.xxx.xx.xxx"):   # 两台机器作为Master，判断哪台HMaster处于活跃状态
    command = "curl http://"+ str(node) + ":9870/jmx?qry=Hadoop:service=NameNode,name=NameNodeStatus"
    status = os.popen(command).read()
    print("HBase Master status: ".format(status))
    if ("active" in status):
        activenode = node
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;执行完毕后，可通过返回的“State”字段判断当前节点状态（活跃为“active”，不活跃为“standby”），如图所示。
![](https://img-blog.csdnimg.cn/20210222230930222.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为避免数据都写入一个region，造成HBase的数据倾斜问题。在当前HMaster活跃的节点上，创建预分区表：

```sql
create 'userprofile_labels', { NAME => "f", BLOCKCACHE => "true" , BLOOMFILTER => "ROWCOL" , COMPRESSION => 'snappy', IN_MEMORY => 'true' }, {NUMREGIONS => 10,SPLITALGO => 'HexStringSplit'}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将待同步的数据写入HFile，HFile中的数据以 key-value 键值对方式存储，然后将 HFile 数据使用 BulkLoad 批量写入 HBase 集群中。Scala脚本执行如下：

```shell
import org.apache.hadoop.fs.{FileSystem, Path}
import org.apache.hadoop.HBase.client.ConnectionFactory
import org.apache.hadoop.HBase.{HBaseConfiguration, KeyValue, TableName}
import org.apache.hadoop.HBase.io.ImmutableBytesWritable
import org.apache.hadoop.HBase.mapreduce.{HFileOutputFormat2, LoadIncrementalHFiles}
import org.apache.hadoop.HBase.util.Bytes
import org.apache.hadoop.mapreduce.Job
import org.apache.spark.sql.SparkSession

object Hive2HBase {
  def main(args: Array[String]): Unit = {
      // 传入日期参数 和 当前活跃的master节点
    val data_date = args(0)
    val node = args(1)   //当前活跃的节点ip

    val spark = SparkSession
      .builder()
      .appName("Hive2HBase")
      .config("spark.serializer","org.apache.spark.serializer.KryoSerializer")
      .config("spark.storage.memoryFraction", "0.1")
      .config("spark.shuffle.memoryFraction", "0.7")
      .config("spark.memory.useLegacyMode", "true")
      .enableHiveSupport()
      .getOrCreate()
        
       //创建HBase的配置
    val conf = HBaseConfiguration.create()
        conf.set("HBase.zookeeper.quorum", "10.xxx.xxx.xxx,10.xxx.xxx.xxx")
        conf.set("HBase.zookeeper.property.clientPort", "8020")

    //为了预防hfile文件数过多无法进行导入，设置参数值
    conf.setInt("HBase.hregion.max.filesize", 10737418240)
    conf.setInt("HBase.mapreduce.bulkload.max.hfiles.perRegion.perFamily", 3200)
      
    val Data = spark.sql(s"select userid,userlabels from dw.userprofile_usergroup_labels_all where data_date='${data_date}'")
    val dataRdd = Data.rdd.flatMap(row => {    
      val rowkey = row.getAs[String]("userid".toLowerCase)
      val tagsmap = row.getAs[Map[String, Object]]("userlabels".toLowerCase)
      val sbkey = new StringBuffer()  // 对MAP结构转化 a->b  'a':'b'
      val sbvalue = new StringBuffer()
      for ((key, value) <- tagsmap){
        sbkey.append(key + ":")
        val labelght = if (value == ""){
          "-999999"
        } else {
          value
        }
        sbvalue.append(labelght + ":")
      }
      val item = sbkey.substring(0,sbkey.length -1)
      val score = sbvalue.substring(0,sbvalue.length -1)
      Array(
        (rowkey,("f","i",item)),
        (rowkey,("f","s",score))
      )
    })

    // 将rdd转换成HFile需要的格式
    val rdds = dataRdd.filter(x=>x._1 != null).sortBy(x=>(x._1,x._2._1, x._2._2)).map(x => {
      //KeyValue的实例为value
      val rowKey = Bytes.toBytes(x._1)
      val family = Bytes.toBytes(x._2._1)
      val colum = Bytes.toBytes(x._2._2)
      val value = Bytes.toBytes(x._2._3.toString)
      (new ImmutableBytesWritable(rowKey), new KeyValue(rowKey, family, colum, value))
    })

    //文件保存在hdfs的位置
    val locatedir = "hdfs://" + node.toString + ":8020/user/bulkload/hfile/usergroup_HBase_" + data_date

    //在locatedir生成的Hfile文件
    rdds.saveAsNewAPIHadoopFile(locatedir,
      classOf[ImmutableBytesWritable],
      classOf[KeyValue],
      classOf[HFileOutputFormat2],
      conf)
    //HFile导入到HBase
    val load = new LoadIncrementalHFiles(conf)

    //HBase的表名
    val tableName = "userprofile_labels"
    //创建HBase的链接,利用默认的配置文件,读取HBase的master地址
    val conn = ConnectionFactory.createConnection(conf)
    //根据表名获取表
    val table = conn.getTable(TableName.valueOf(tableName))

    try {
      //获取HBase表的region分布
      val regionLocation = conn.getregionLocation(TableName.valueOf(tableName))
      //创建一个hadoop的mapreduce的job
      val job = Job.getInstance(conf)
      //设置job名称，任意命名
      job.setJobName("Hive2HBase")
      //输出文件的内容KeyValue
      job.setMapOutputValueClass(classOf[KeyValue])
      //设置文件输出key, outkey要用ImmutableBytesWritable
      job.setMapOutputKeyClass(classOf[ImmutableBytesWritable])
      //配置HFileOutputFormat2的信息
      HFileOutputFormat2.configureIncrementalLoad(job, table, regionLocation)
      //开始导入
      load.doBulkLoad(new Path(locatedir), conn.getAdmin, table, regionLocation)
    } finally {
      table.close()
      conn.close()
    }
    spark.close()
  }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;提交Spark任务，将HFile中数据bulkload到HBase中。执行完成后，可以在HBase中看到该数据已经写入“userprofile_labels”中

![](https://img-blog.csdnimg.cn/20210222231310996.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='	tomato'>在线接口在查询HBase中数据时，由于HBase无法像关系数据库那样根据多种条件对数据进行筛选</font>（类似SQL语言中的where筛选条件）。<font color='	tomato'>一般地HBase需建立**二级索引**来满足根据复杂条件查询数据的需求</font>，本案中选用 **Elasticsearch** 存储HBase索引数据

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在组合标签查询对应的用户人群场景中，首先通过组合标签的条件在 Elasticsearch 中查询对应的索引数据，然后通过索引数据去 HBase中批量获取 rowkey 对应的数据（**Elasticsearch中的documentid和HBase中的rowkey都设计为用户id**）

![](https://img-blog.csdnimg.cn/20210222231635143.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了避免从 Hive 向 HBase 灌入数据时缺失，在向HBase数据同步完成后，还需要**校验HBase和Hive中数据量是否一致**，如出现较大的波动则发送告警信息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过Python脚本来看该HBase状态表数据校验逻辑：

```powershell
# 查询Hive中数据
def check_Hive_data(data_date):
      r = os.popen("Hive -S -e\"select count(1) from dw.userprofile_usergroup_labels_all where data_date='"+data_date+"'\"")
      Hive_userid_count = r.read()
      r.close()
      Hive_count = str(int(Hive_userid_count) 
      print "Hive_result: " + str(Hive_count)
      print "Hive select finished!"

# 查询HBase中数据 
def check_HBase_data(data_date):
      r = os.popen("HBase  org.apache.hadoop.HBase.mapreduce.RowCounter 'userprofile_labels'\" 2>&1 |grep ROWS")
      HBase_count = r.read().strip()[5:]
      r.close()
      print "HBase result: " + str(HBase_count)
      print "HBase select finished!"

# 连接 DB,将查询结果插入表
db = MySQLdb.connect(host="xx.xx.xx.xx",port=3306,user="username", passwd="password", db="xxx", charset="utf8")
cursor = db.cursor()
cursor.execute("INSERT INTO service_monitor(date, service_type, Hive_count, HBase_count) VALUES('"+datestr_+"', 'advertisement', "+str(Hive_userid_count)+","+str(HBase_count)+")")
db.commit()
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本案例中将 userid 作为 rowkey 存入HBase，一方面在组合标签的场景中可以支持条件查询多用户人群，另一方面可以支持单个用户标签的查询，例如查看某 id 用户身上的标签，以便运营人员决定是否对其进行运营操作。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**HBase在离线数仓环境的服务架构如图所示**：
![](https://img-blog.csdnimg.cn/20210222232403415.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇文章主要介绍了在用户画像的业务场景下，HBase存储相关数据的真实应用场景！下一篇为大家介绍 **Elasticsearch** ，敬请期待！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210212133657263.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)




