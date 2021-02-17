### 1.概述 

 - #### 介绍

  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信用过MySQL的朋友都知道，MySQL中也有开窗函数的存在。<font color='blue'>开窗函数的引入是为了既显示聚集前的数据，又显示聚集后的数据。</font>即在每一行的最后一列添加聚合函数的结果。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开窗用于为行定义一个窗口(这里的窗口是指运算将要操作的行的集合)，它对一组值进行操作，不需要使用 GROUP BY 子句对数据进行分组，能够在同一行中同时返回基础行的列和聚合列。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
 - #### 聚合函数和开窗函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>聚合函数是将多行变成一行</font>，count,avg....

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>开窗函数是将一行变成多行</font>；

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;聚合函数如果要显示其他的列必须将列加入到group by中

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开窗函数可以不使用group by，直接将所有信息显示出来


 - #### 开窗函数分类


1. <font color='red'>聚合开窗函数 </font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>聚合函数(列) OVER(选项)</font>，这里的选项可以是PARTITION BY 子句，但不可以是 ORDER BY 子句。

2. <font color='red'>排序开窗函数</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>排序函数(列) OVER(选项)</font>，这里的选项可以是ORDER BY 子句，也可以是OVER(PARTITION BY 子句 ORDER BY 子句)，但<font color='red'>不可以是 PARTITION BY 子句</font>。


### 2. 准备工作
进入到SparkShell命令行
```javascript
/export/servers/spark/bin/spark-shell --master spark://node01:7077,node02:7077
```

创建一个样例类，用于封装数据

```javascript
case class Score(name: String, clazz: Int, score: Int)
```

创建一个RDD数组，造一些数据，并调用`toDF`方法将其转换成DataFrame

```javascript
val scoreDF = spark.sparkContext.makeRDD(Array(
Score("a1", 1, 80),
Score("a2", 1, 78),
Score("a3", 1, 95),
Score("a4", 2, 74),
Score("a5", 2, 92),
Score("a6", 3, 99),
Score("a7", 3, 99),
Score("a8", 3, 45),
Score("a9", 3, 55),
Score("a10", 3, 78),
Score("a11", 3, 100))
).toDF("name", "class", "score")
```

创建一个临时表

```javascript
scoreDF.createOrReplaceTempView("scores")
```
查询所有数据

```javascript
scoreDF.show()
+----+-----+-----+
|name|class|score|
+----+-----+-----+
|  a1|    1|   80|
|  a2|    1|   78|
|  a3|    1|   95|
|  a4|    2|   74|
|  a5|    2|   92|
|  a6|    3|   99|
|  a7|    3|   99|
|  a8|    3|   45|
|  a9|    3|   55|
| a10|    3|   78|
| a11|    3|  100|
+----+-----+-----+
```


### 3. 聚合开窗函数

 - **示例1**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>OVER 关键字表示把<font color='black'>**聚合函数**</font>当成<font color='black'>**聚合开窗函数**</font>而不是聚合函数。</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SQL标准允许将所有聚合函数用做聚合开窗函数。

```objectivec
spark.sql("select  count(name)  from scores").show
```

```javascript
spark.sql("select name, class, score, count(name) over() name_count from scores").show
```

查询结果如下所示：

```javascript
查询结果如下所示：
+----+-----+-----+----------+                                                   
|name|class|score|name_count|
+----+-----+-----+----------+
|  a1|    1|   80|        11|
|  a2|    1|   78|        11|
|  a3|    1|   95|        11|
|  a4|    2|   74|        11|
|  a5|    2|   92|        11|
|  a6|    3|   99|        11|
|  a7|    3|   99|        11|
|  a8|    3|   45|        11|
|  a9|    3|   55|        11|
| a10|    3|   78|        11|
| a11|    3|  100|        11|
+----+-----+-----+----------+
```
 - **示例2**
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>OVER 关键字后的括号</font>中还可以添加选项用以改变进行聚合运算的窗口范围。
如果 **OVER** 关键字后的括号中的**选项为空**，则开窗函数**会对结果集中的所有行进行聚合运算**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开窗函数的 OVER 关键字后括号中的<font color='red'>可以使用 PARTITION BY 子句来定义行的分区来供进行聚合计算</font>。与 **GROUP BY** 子句不同，**PARTITION BY** 子句创建的<font color='red'>分区是独立于结果集的</font>，创建的分区只是供进行聚合计算的，而且<font color='red'>不同的开窗函数所创建的分区也不互相影响</font>。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面的 SQL 语句用于显示<font color='blue'>按照班级分组后每组的人数：</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>OVER(PARTITION BY class)</font>表示对结果集按照 class 进行分区，并且计算当前行所属的组的聚合计算结果。

```javascript
spark.sql("select name, class, score, count(name) over(partition by class) name_count from scores").show
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询结果如下所示： 

```javascript
+----+-----+-----+----------+                                                   
|name|class|score|name_count|
+----+-----+-----+----------+
|  a1|    1|   80|         3|
|  a2|    1|   78|         3|
|  a3|    1|   95|         3|
|  a6|    3|   99|         6|
|  a7|    3|   99|         6|
|  a8|    3|   45|         6|
|  a9|    3|   55|         6|
| a10|    3|   78|         6|
| a11|    3|  100|         6|
|  a4|    2|   74|         2|
|  a5|    2|   92|         2|
+----+-----+-----+----------+
```

### 4. 排序开窗函数

#### 4.1 ROW_NUMBER顺序排序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;row_number() over(order by score) as rownum 表示按score 升序的方式来排序，并得出排序结果的序号

注意：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在排序开窗函数中使用 **PARTITION  BY** 子句需要放置在**ORDER  BY** 子句之前。

 - 示例1

```javascript
spark.sql("select name, class, score, row_number() over(order by score) rank from scores").show()
+----+-----+-----+----+
|name|class|score|rank|
+----+-----+-----+----+
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
|  a4|    2|   74|   3|
|  a2|    1|   78|   4|
| a10|    3|   78|   5|
|  a1|    1|   80|   6|
|  a5|    2|   92|   7|
|  a3|    1|   95|   8|
|  a6|    3|   99|   9|
|  a7|    3|   99|  10|
| a11|    3|  100|  11|
+----+-----+-----+----+
```

```javascript
spark.sql("select name, class, score, row_number() over(partition by class order by score) rank from scores").show()
+----+-----+-----+----+                                                         
|name|class|score|rank|
+----+-----+-----+----+
|  a2|    1|   78|   1|
|  a1|    1|   80|   2|
|  a3|    1|   95|   3|
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
| a10|    3|   78|   3|
|  a6|    3|   99|   4|
|  a7|    3|   99|   5|
| a11|    3|  100|   6|
|  a4|    2|   74|   1|
|  a5|    2|   92|   2|
+----+-----+-----+----+
```

#### 4.2 RANK跳跃排序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;rank() over(order by score) as rank表示按 score升序的方式来排序，并得出排序结果的**排名号**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个函数求出来的<font color='red'>排名结果可以并列（并列第一/并列第二），并列排名之后的排名将是并列的排名加上并列数
</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='blue'>简单说每个人只有一种排名，然后出现两个并列第一名的情况，这时候排在两个第一名后面的人将是第三名，也就是没有了第二名，但是有两个第一名</font>。

 - 实例2

```javascript
spark.sql("select name, class, score, rank() over(order by score) rank from scores").show()                                                     
+----+-----+-----+----+
|name|class|score|rank|
+----+-----+-----+----+
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
|  a4|    2|   74|   3|
| a10|    3|   78|   4|
|  a2|    1|   78|   4|
|  a1|    1|   80|   6|
|  a5|    2|   92|   7|
|  a3|    1|   95|   8|
|  a6|    3|   99|   9|
|  a7|    3|   99|   9|
| a11|    3|  100|  11|
+----+-----+-----+----+
```

```javascript
spark.sql("select name, class, score, rank() over(partition by class order by score) rank from scores").show()
+----+-----+-----+----+                                                         
|name|class|score|rank|
+----+-----+-----+----+
|  a2|    1|   78|   1|
|  a1|    1|   80|   2|
|  a3|    1|   95|   3|
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
| a10|    3|   78|   3|
|  a6|    3|   99|   4|
|  a7|    3|   99|   4|
| a11|    3|  100|   6|
|  a4|    2|   74|   1|
|  a5|    2|   92|   2|
+----+-----+-----+----+
 
```
#### 4.3 DENSE_RANK连续排序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;dense_rank() over(order by  score) as  dense_rank 表示按score 升序的方式来排序，并得出排序结果的排名号。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个函数<font color='red'>并列排名之后的排名是并列排名加１</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单说每个人只有一种排名，然后<font color='blue'>出现两个并列第一名的情况，这时候排在两个第一名后面的人将是第二名，也就是两个第一名，一个第二名
</font>

 - 实例3

```javascript
spark.sql("select name, class, score, dense_rank() over(order by score) rank from scores").show()
+----+-----+-----+----+
|name|class|score|rank|
+----+-----+-----+----+
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
|  a4|    2|   74|   3|
|  a2|    1|   78|   4|
| a10|    3|   78|   4|
|  a1|    1|   80|   5|
|  a5|    2|   92|   6|
|  a3|    1|   95|   7|
|  a6|    3|   99|   8|
|  a7|    3|   99|   8|
| a11|    3|  100|   9|
+----+-----+-----+----+
```

```javascript
spark.sql("select name, class, score, dense_rank() over(partition by class order by score) rank from scores").show()
+----+-----+-----+----+                                                         
|name|class|score|rank|
+----+-----+-----+----+
|  a2|    1|   78|   1|
|  a1|    1|   80|   2|
|  a3|    1|   95|   3|
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
| a10|    3|   78|   3|
|  a6|    3|   99|   4|
|  a7|    3|   99|   4|
| a11|    3|  100|   5|
|  a4|    2|   74|   1|
|  a5|    2|   92|   2|
+----+-----+-----+----+

```

#### 4.4  NTILE分组排名[了解]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**ntile(6) over(order by score)as ntile**表示按 score 升序的方式来排序，然后 6 等<font color='red'>分成 6 个组，并显示所在组的序号。</font>

 - 实例4

```javascript
spark.sql("select name, class, score, ntile(6) over(order by score) rank from scores").show()
+----+-----+-----+----+
|name|class|score|rank|
+----+-----+-----+----+
|  a8|    3|   45|   1|
|  a9|    3|   55|   1|
|  a4|    2|   74|   2|
|  a2|    1|   78|   2|
| a10|    3|   78|   3|
|  a1|    1|   80|   3|
|  a5|    2|   92|   4|
|  a3|    1|   95|   4|
|  a6|    3|   99|   5|
|  a7|    3|   99|   5|
| a11|    3|  100|   6|
+----+-----+-----+----+
```

```javascript
spark.sql("select name, class, score, ntile(6) over(partition by class order by score) rank from scores").show()
+----+-----+-----+----+                                                         
|name|class|score|rank|
+----+-----+-----+----+
|  a2|    1|   78|   1|
|  a1|    1|   80|   2|
|  a3|    1|   95|   3|
|  a8|    3|   45|   1|
|  a9|    3|   55|   2|
| a10|    3|   78|   3|
|  a6|    3|   99|   4|
|  a7|    3|   99|   5|
| a11|    3|  100|   6|
|  a4|    2|   74|   1|
|  a5|    2|   92|   2|
+----+-----+-----+----+
```

***

### 结语
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本次的分享就到这里，<font color='#BE77FF' size='4'>**受益的朋友或对大数据技术感兴趣的伙伴**</font>可以点个赞关注一下博主，后续会持续更新大数据的相关内容，敬请期待(✪ω✪)

![](https://img-blog.csdnimg.cn/20210119222335538.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)






