## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们在上一篇 Kylin 的入门级介绍（👉[第一个“国产“Apache顶级项目——Kylin，了解一下！](https://alice.blog.csdn.net/article/details/115267160)）中，就已经谈到了有很多可以与 Kylin 结合使用的可视化工具，例如
-  ODBC：与Tableau、Excel、Power BI等工具集成。
-  JDBC：与Saiku、BIRT等Java工具集成
- REST API：与JavaScript、Web网页集成。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Kylin开发团队还贡献了 <font color='red'>Zepplin</font> 的插件，也可以使用Zepplin来访问Kylin服务

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期内容，我们就先介绍如何通过 **JDBC** 和 **Zeppelin** 的方式对 Kylin 进行集成！


## JDBC

（1）新建一个Maven项目，并在 pom 文件中导入`kylin-jdbc`的依赖
```xml
<dependencies>
     <dependency>
         <groupId>org.apache.kylin</groupId>
         <artifactId>kylin-jdbc</artifactId>
         <version>2.5.1</version>
     </dependency>
</dependencies>
```
（2）编写代码：
```java
package com.alice;

import java.sql.*;

/**
 * @Author: Alice菌
 * @Date: 2021/3/29 18:29
 * @Description:   通过 JDBC 连接 Kylin
 */
public class KylinTest {
    public static void main(String[] args) throws SQLException, ClassNotFoundException {

        // Kylin_JDBC 驱动
        String kylinDriver = "org.apache.kylin.jdbc.Driver";

        // Kylin 的 URL (jdbc:kylin://ip地址:7070/项目名称（project）)
        String KylinUrl = "jdbc:kylin://node01:7070/demoTest";

        // Kylin 的 用户名
        String KylinUser = "ADMIN";

        // Kylin 的密码
        String KylinPasswd = "KYLIN";

        // 添加驱动信息
        Class.forName(kylinDriver);

        // 获取连接
        Connection connection = DriverManager.getConnection(KylinUrl, KylinUser, KylinPasswd);

        // 预编译 SQL 语句
        PreparedStatement ps = connection.prepareStatement("select date1, sum(price) as total_money, sum(amount) as total_amount from dw_sales group by date1,channelid");

        // 执行查询
        ResultSet resultSet = ps.executeQuery();

        // 遍历打印
        while (resultSet.next()){

            // 获取时间
            String date1 = resultSet.getString("date1");
            // 获取总金额
            String total_money = resultSet.getString("total_money");
            // 获取总次数
            String total_amount = resultSet.getString("total_amount");

            // 输出结果
            System.out.println(date1 + " " + total_money + " " + total_amount);

        }

        // 关闭连接
        connection.close();

    }
}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）结果展示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329185007573.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了验证结果的准确性，我们到 Kylin 的 web 页面去查询一下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202103291906093.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以发现结果是一样的😎

## Zeppelin

### 1）Zeppelin安装与启动

（1）将`zeppelin-0.8.0-bin-all.tgz`上传至Linux

（2） 解压`zeppelin-0.8.0-bin-all.tgz`到`/export/servers`目录下

```bash
[root@node01 software]# tar -zxvf zeppelin-0.8.0-bin-all.tgz -C /export/servers/
```
（3）修改名称

```bash
[root@node01 servers]# mv zeppelin-0.8.0-bin-all/ zeppelin
```
（4）启动 Zeppelin

```bash
[root@node01 zeppelin]# bin/zeppelin-daemon.sh start
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以本机为例，我们可以通过 `http://node01:8080/#/`进行查看，Web 默认端口为 8080 ，如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329213150668.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
### 2）配置Zeppelin支持Kylin
（1）选择“anonymous” → “Interpreter”选项，配置解释器

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329213524905.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)


（2）搜索Kylin插件并修改相应的配置，如图所示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329214025951.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

（3）修改完成后，单击“Save” 按钮保存修改内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329214037659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
### 3）案例实操

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需求：查询订单商品dw_sales表的数据，并使用各种图表进行展示。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（1）选择“Notebook”→“Creat new note”选项，创建新的 note
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329214700772.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（2）在 “Note Name” 文本框中输入 “test_kylin” 并单击“Create”按钮，如图所示：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210329214829959.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（3）note 创建成功如图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004144266.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;（4）结果展示
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004625105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
（5）其他图表格式
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004714855.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004746496.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004807640.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210330004913351.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期文章为大家介绍了 2 种通过 BI 工具展示 Kylin 查询结果的方式 ，大家仅做学习了解即可。好了，本期内容就到这里，后面会为大家介绍关于 **Cube 的构建原理** 和 **构建优化**。感兴趣的小伙伴记得点个<font color='orange'>**关注**</font>，第一时间阅读！

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;你知道的越多，你不知道的也越多。我是<font color='purple'>**大数据梦想家**</font>，<font color='RoyalBlue'>**专注于研究大数据基础，架构与原型实现**</font>。点个**关注**，我们下一期见！







