## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一篇文章已经为大家介绍了 HBase 在用户画像的标签数据存储中的具体应用场景，本篇我们来谈谈 **Elasticsearch** 的使用！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224003333410.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
>**原著作者：赵宏田
来源：《用户画像方法论与工程化解决方案》**
***
## Elasticsearch存储
### Elasticsearch简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='tomato'>Elasticsearch 是一个开源的分布式全文检索引擎，可以近乎实时地存储、检索数据</font>。而且<font color='RoyalBlue'>可扩展性很好，可以扩展到上百台服务器，处理PB级别的数据</font>。对于**用户标签查询**、**用户人群计算**、**用户群多维透视分析**这类对响应时间要求较高的场景，也可以考虑选用Elasticsearch进行存储。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Elasticsearch是面向文档型数据库，**一条数据在这里就是一个文档**，用 json 作为文档格式。为了更清晰地理解 Elasticsearch 查询的一些概念，将其和关系型数据库的类型进行对照。

| Elasticsearch | MySQL ||
|--|--|--|
| index | database |数据库|
|type|table|表|
|document|row|行|
|mapping|column|列|
|GET http://...|SELECT * FROM ...|查询数据|
|PUT http://...|UPDATE table SET...|插入数据|

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在关系型数据库中查询数据时可通过选中数据库、表、行、列来定位所查找的内容，在Elasticsearch中通过**索引（index）、类型（type）、文档（document）、字段**来定位查找内容。一个Elasticsearch集群可以包括多个索引（数据库），也就是说，其中包含了很多类型（表），这些类型中包含了很多的文档（行），然后每个文档中又包含了很多的字段（列）。Elasticsearch的交互可以使用Java API，也可以使用 **HTTP** 的**RESTful API**方式。

### 应用场景
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上一节的内容中，我们谈到基于 HBase 的存储方案并没有解决数据的 **高效检索** 问题。在实际应用中，经常有**根据特定的几个字段进行组合后检索**的应用场景，而 <font color='blue'>HBase 采用 rowkey 作为一级索引，不支持多条件查询</font>，如果要对库里的非 rowkey 进行数据检索和查询，往往需要通过 MapReduce 等分布式框架进行计算，时间延迟上会比较高，**难以同时满足用户对于复杂条件查询和高效率响应这两方面的需求**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='	Tomato'>为了既能支持对数据的**高效查询**，同时也能支持通过条件筛选进行复杂查询，需要在HBase上构建**二级索引**，以满足对应的需要</font>。在本案中我们采用**Elasticsearch**存储 HBase 的索引信息，以支持复杂高效的查询功能。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要查询过程包括：

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;1）在Elasticsearch中存放用于检索条件的数据，并将rowkey 也存储进去；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;2）使用Elasticsearch的 API 根据组合标签的条件查询出rowkey的集合；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3）使用上一步得到的 rowkey 去HBase数据库查询对应的结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224004308194.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HBase存储数据的索引放在Elasticsearch中，实现了**数据和索引的分离**。在Elasticsearch中`documentid`是文档的唯一标识，在HBase中`rowkey`是记录的唯一标识。在工程实践中，两者可同时选用用户在平台上的唯一标识（如userid或deviceid）作为rowkey或documentid，进而解决 HBase 和 Elasticsearch 索引关联的问题。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过使用 Elasticsearch 解决用户人群计算和分析应用场景的案例来了解这一过程。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对汇聚后的用户标签表dw.`userprofile_userlabel_map_all`中的数据进行清洗，过滤掉一些无效字符，达到导入Elasticsearch的条件，如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224230659422.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;然后将dw.`userprofile_userlabel_map_all`数据写入**Elasticsearch** 中，Scala代码如下：

```scala
object HiveDataToEs {

  def main(args: Array[String]): Unit = {

    val spark = SparkSession.builder()
      .AppName("EsData")
      .config("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
      .config("spark.dynamicAllocation.enabled", "false")
      .config("es.index.auto.create", "true")
      .config("es.nodes", "10.xx.xx.xx")
      .config("es.batch.write.retry.count", "3")    // 默认重试3次
      .config("es.batch.write.retry.wait", "5")   // 每次重试等待时间为5秒
      .config("thread_pool.write.queue_size", "1000")
      .config("thread_pool.write.size", "50")
      .config("thread_pool.write.type", "fixed")   
      .config("es.batch.size.bytes", "20mb")  
      .config("es.batch.size.entries", "2000")  
      .config("es.http.timeout","100m")
      .enableHiveSupport()
      .getOrCreate()

    val data_date  = args(0).toString
    import spark.sql
    val hiveDF = sql(
      s"""
         | SELECT userid, tagsmap FROM dw.userprofile_userlabel_map_all where data_date = '${data_date}'
      """.stripMargin)    // dw.userprofile_userlabel_map_all 是聚合用户标签的表

    val rdd = hiveDF.rdd.map {
      row => {
        val userid = row.getAs[String]("userid")
        val userlabels = row.getAs[Map[String, Object]]("userlabels")
        Map("userid" -> userid, "userlabels" -> userlabels)
      }
    }
    EsSpark.saveToEs(rdd , "userprofile/tags", Map[String,String]("es.mApping.id"->"userid")   
    spark.stop()
  }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;工程依赖如下：

```xml
<dependency>
    <groupId>org.elasticsearch</groupId>
    <artifactId>elasticsearch-hadoop</artifactId>
    <version>6.4.2</version>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将该工程打包之后提交任务，传入日期分区参数 “20190101”执行。提交命令` “spark-submit--class com.example.HiveDataToEs--master yarn--deploy-mode client--executor-memory 2g--num-executors 50--driver-memory 3g--executor-cores 2 spark-hive-to-es.jar 20190101”`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;任务执行完毕后，当日 userid 维度的用户标签数据全部导入Elasticsearch中。使用RESTfulAPI查询包含某个标签的用户量，可实时得到返回结果。

```json
# 查询命令
GET userprofile/tags/_search
{
  "size":0,
  "aggs": {
    "tagcounts": {
      "terms": {
        "field": "tags.ACTION_U_01_003"
      }
    }
  }
}
```
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224231410177.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从返回结果中可以看到，用户总量（total）为100000000人，包含标签“`ACTION_U_01_003`”的用户有2500000人（doc_count）。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询人群 index 查看标签总量：

```json
# 查询命令
GET userprofile/_search 
{
  "query":{
    "match_all": {}
  }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询结果如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2021022423172936.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在人群的计算和分析场景中，经过产品的迭代，前期采用 **Impala** 进行计算，一般耗费几十秒到几分钟的时间，在使用 Elasticsearch 后，实现了对人群计算的秒级响应。

### 工程化案例
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过一个工程案例来讲解实现画像产品中“用户人群”和“人群分析”功能对用户群计算秒级响应的一种解决方案。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在每天的 ETL 调度中，需要将 Hive 计算的标签数据导入Elasticsearch中。如图所示，在标签调度完成且通过校验后（图中的“标签监控预警”任务执行完成后），将标签数据同步到Elasticsearch中。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224232257644.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在与 Elasticsearch 数据同步完成并通过校验后，向在  MySQL 中维护的状态表中插入一条状态记录，表示当前日期的 Elasticsearch 数据可用，线上计算用户人群的接口则读取最近日期对应的数据。如果某天因为调度延迟等方面的原因，没有及时将当日数据导入Elasticsearch中，接口也能读取最近一天对应的数据，是一种可行的灾备方案。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;例如，数据同步完成后向MySQL状态表“elasticsearch_state”中插入记录（如图所示），当日数据产出正常时，state字段为“0”，产出异常时为“1”。图3-29中1月20日导入的数据出现异常，则“state”状态字段置1，线上接口扫描该状态记录位后不读取1月20日数据，而是取用最近的1月19日数据。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224232536590.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了避免从 Hive 向 Elasticsearch 中灌入数据时发生数据缺失，在向状态表更新状态位前需要校验 Elasticsearch 和 Hive 中的数据量是否一致。下面通过Python脚本来看**数据校验逻辑**：

```python
# 查询Hive中的数据
def monitor_hive_data(data_date):
    hive_user = " select count(1) from dw.userprofile_userlabel_map_all where data_date='{}' ".format(data_date)
    user_count = os.popen("hive -S -e \"" + hive_user + "\"").read().strip()
    return user_count

# 查询es中的数据
def monitor_es_data(data_date):
    userid_search = "curl http://10.xxx.xxx.xxx:9200/_cat/count/" + data_date + "_userid/"
    userid_num = str(os.popen(userid_search).read()).split(' ')[-1].strip()
    return userid_num

# 比较Hive和es中的数据，如通过校验，更新MySQL状态位
def update_es_data(data_date):
    '''
    data_date: 查询数据日期
    '''
    esdata = monitor_es_data(data_date)    # 查询es中的数据
    hivedata = monitor_hive_data(data_date)  # 查询Hive中的数据
    print("esdata ======>{}".format(esdata))
    print("hivedata ======>{}".format(hivedata))

    # 更新MySQL状态位 
    if (esdata[0] == hivedata[0] ):
        db = MySQLdb.connect(host="10.xx.xx.xx", port=3306, user="username", passwd="password",
                             db="userprofile", charset="utf8")
        cursor = db.cursor()
        try:
            select_command = "INSERT INTO `elasticsearch_state` VALUES ('"+ str(data_date) +"', 'elasticsearch', '0', '2');"
            cursor.execute(select_command)
            db.commit()
        except Exception as e:
            db.rollback()
           exit(1)
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上面介绍了在工程化调度流中何时将Hive中的用户标签数据灌入Elasticsearch中，之后业务人员在画像产品端计算人群或透视分析人群时（如图所示），
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224232746353.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

通过RESTful API访问 Elasticsearch 进行计算

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210224232806313.png?type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结合前面几期文章，分别为大家讲解了使用 Hive、MySQL、HBase 和 Elasticsearch 存储标签数据的解决方案，包括：<font color='RoyalBlue'>Hive存储数据相关标签表、人群计算表的表结构设计以及ID-Mapping的一种实现方式；MySQL存储标签元数据、监控数据及结果集数据；HBase存储线上接口实时调用的数据；Elasticsearch存储标签用于人群计算和人群多维透视分析</font>。存储过程中涉及如下相关表。

 - `dw.userprofile_attritube_all`：存储人口属性维度的标签表；
 - `dw.userprofile_action_all`：存储行为属性维度的标签表；
 - `dw.userprofile_consume_all`：存储用户消费维度的标签表；
 - `dw.userprofile_riskmanage_all`：存储风险控制维度的标签表；
 - `dw.userprofile_social_all`：存储社交属性维度的标签表；
 - `dw.userprofile_userlabel_map_all`：汇聚用户各维度标签的表；
 - `dw.userprofile_usergroup_labels_all`：存储计算后人群数据的表。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，本篇文章就到这里，更多干货文章请关注我的公众号。你知道的越多，你不知道的也越多。我是Alice，我们下一期见！



![在这里插入图片描述](https://img-blog.csdnimg.cn/20210303133100452.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
