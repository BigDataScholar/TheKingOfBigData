>**本文已收录github：https://github.com/BigDataScholar/TheKingOfBigData，里面有大数据高频考点，Java一线大厂面试题资源，上百本免费电子书籍，作者亲绘大数据生态圈思维导图…持续更新，欢迎star！**

## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一篇文章已经为大家介绍了 Hive 在用户画像的标签数据存储中的具体应用场景，本篇我们来谈谈MySQL的使用！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210221224201410.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
> **原著作者：赵宏田**
> **来源：《用户画像方法论与工程化解决方案》**
***
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MySQL作为关系型数据库，在用户画像中可用于元数据管理、监控预警数据、结果集存储等应用中。下面详细介绍这3个应用场景。

### 元数据管理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='tomato'>Hive适合于大数据量的批处理作业，对于量级较小的数据，MySQL具有更快的读写速度</font>。Web端产品读写MySQL数据库会有更快的速度，方便标签的定义、管理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在介绍**用户画像产品化**的时候，我们会介绍元数据录入和查询功能，将相应的数据存储在MySQL中。用户标签的元数据表结构设计也会在之后进行详细的介绍。这里给出了平台标签视图和元数据管理页面。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210221192719476.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)![在这里插入图片描述](https://img-blog.csdnimg.cn/20210221192731558.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;平台标签视图中的标签元数据可以维护在MySQL关系数据库中，便于标签的编辑、查询和管理。

### 监控预警数据
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;MySQL还可用于**存储每天对ETL结果的监控信息**。从整个画像调度流的关键节点来看，需要监控的环节主要包括对每天标签的产出量、服务层数据同步情况的监控等主要场景。下图展示的是用户画像调度流主要模块。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210221193123119.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



#### 1.标签计算数据监控
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要用于监控每天标签ETL的数据量是否出现异常，如果有异常情况则发出告警邮件，同时暂停后面的ETL任务。

#### 2. 服务层同步数据监控
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;服务层一般采用`HBase`、`Elasticsearch`等作为数据库**存储标签数据**供线上调用，将标签相关数据从Hive数仓向服务层同步的过程中，有出现差错的可能，**因此需要记录相关数据在Hive中的数量及同步到对应服务层后的数量**，如果数量不一致则触发告警。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在对画像的数据监控中，调度流每跑完相应的模块，就将该模块的监控数据插入MySQL中，当校验任务判断达到触发告警阈值时，发送告警邮件，同时中断后续的调度任务。待开发人员解决问题后，可重启后续调度。


#### 3. 结果集存储
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果集可以用来存储多维透视分析用的标签、圈人服务用的用户标签、当日记录各标签数量，用于校验标签数据是否出现异常。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有的线上业务系统使用MySQL、Oracle等关系型数据库存储数据，如短信系统、消息推送系统等。在打通画像数据与线上业务系统时，需要考虑将存储在Hive中的用户标签相关数据同步到各业务系统，此时**MySQL可用于存储结果集**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='tomato'>Sqoop是一个用来将Hadoop和关系型数据库中的数据相互迁移的工具。它可以将一个关系型数据库（如MySQL、Oracle、PostgreSQL等）中的数据导入Hadoop的HDFS中，也可以将HDFS中的数据导入关系型数据库中</font>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面通过一个案例来讲解如何使用Sqoop将Hive中的标签数据迁移到MySQL中。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='RoyalBlue'>电商、保险、金融等公司的客服部门的日常工作内容之一是对目标用户群（如已流失用户、高价值用户等）进行主动外呼，以此召回用户来平台进行购买或复购</font>。这里可以借助用户画像系统实现该功能。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;将Hive中存储的与用户身份相关的数据同步到客服系统中，首先在Hive中建立一张记录用户身份相关信息的表（例如`dw.userprofile_userservice_all`）。设置日期分区以满足按日期选取当前人群的需要。

```sql
CREATE TABLE `dw.userprofile_userservice_all `(
`user_id` string COMMENT 'userid', 
`user_sex` string COMMENT 'user_sex', 
`city` string COMMENT 'city',
`payid_money` string COMMENT 'payid_money', 
`payid_num` string COMMENT 'payid_num', 
`latest_product` string COMMENT 'latest_product', 
`date` string COMMENT 'date', 
`data_status` string COMMENT 'data_status')
COMMENT 'userid 用户客服数据'
PARTITIONED BY ( `data_date` string COMMENT '数据日期')
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在MySQL中建立一张用于接收同步数据的表（`userservice_data`）。

```sql
CREATE TABLE `userservice_data` (
  `user_id` varchar(128) DEFAULT NULL COMMENT '用户id',
  `user_sex` varchar(128) NOT NULL COMMENT '用户性别',
  `city` varchar(128) DEFAULT NULL COMMENT '城市',
  `payid_money` varchar(128) DEFAULT NULL COMMENT '消费金额',
  `payid_num` varchar(128) DEFAULT NULL COMMENT '消费次数',
  `latest_product` varchar(128) DEFAULT NULL COMMENT '最近购买产品',
  `date` varchar(64) NOT NULL COMMENT '传输日期',
  `data_status` varchar(64) DEFAULT '0' COMMENT '0:未传输,1:传输中,2:成功,3:失败',
  PRIMARY KEY (`user_id`),
) ENGINE=InnoDB AUTO_INCREMENT=2261628 DEFAULT CHARSET=utf8 COMMENT='用户客服数据表';
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过Python脚本调用shell命令，将Hive中的数据同步到MySQL中。执行如下脚本：

```sql
# -*- coding: utf-8 -*-
import os
import MySQLdb
import sys
def export_data(hive_tab, data_date):
    sqoop_command = "sqoop export --connect jdbc:mysql://10.xxx.xxx.xxx:3306/mysql_database --username username --password password  --table mysql_table --export-dir hdfs://nameservice1/user/hive/warehouse
/dw.db/" + hive_tab + "/data_date=" + data_date + " --input-fields-terminated-by '\001'"
    os.system(sqoop_command)
    print(sqoop_command)

if __name__ == '__main__':
    export_data("dw.userprofile_userservice_all", '20181201')
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;其中用到了 sqoop 从 Hive 导出数据到 MySQL 的命令：

```sql
sqoop export
--connect 指定JDBC连接字符串,包括IP 端口 数据库名称 \
--username  JDBC连接的用户名\
--passowrd  JDBC连接的密码\
--table  表名\
--export-dir  导出的Hive表, 对应的是HDFS地址 \
--input fileds-terminated-by ‘,’ 分隔符号
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;不熟悉Sqoop使用的小伙伴可以去看我的这篇文章[《硬核 | Sqoop入门指南》](https://alice.blog.csdn.net/article/details/113065391)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同步后 MySQL中的数据如图所示

![](https://img-blog.csdnimg.cn/20210221195149779.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇文章主要介绍了在用户画像的业务场景下，MySQL存储相关数据的真实应用场景！后续会陆续为大家介绍 HBase和Elasticsearch，敬请期待！

![](https://img-blog.csdnimg.cn/20210119222335538.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)