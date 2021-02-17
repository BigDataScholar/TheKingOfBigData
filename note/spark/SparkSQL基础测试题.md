&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;之前推出过一期关于Spark的练习，反响还不错。而最近博主又写了关于**SparkSQL**,**SparkStreaming**，**Structured Streaming**的内容，为了巩固大家的基础，提升实战的能力，故备下了一道综合性比较全面的题，希望大家能够受用。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419103210183.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

***

## 准备数据

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419110823219.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70 =300x300)

### student.txt

字段从左到右依次代表：

		学号   学生姓名	  学生性别	学生出生年月  学生所在班级
```javascript
108	丘东	男	1977-09-01	95033
105	匡明	男	1975-10-02	95031
107	王丽	女	1976-01-23	95033
101	李军	男	1976-02-20	95033
109	王芳	女	1975-02-10	95031
103	陆君	男	1974-06-03	95031
```

### course.txt
		课程号   课程名称	 教工编号
```javascript
3-105	计算机导论	825
3-245	操作系统	804
6-166	数字电路	856
9-888	高等数学	831
```

### score.txt
		学号   课程号	 成绩
```javascript
103	3-245	86
105	3-245	75
109	3-245	68
103	3-105	92
105	3-105	88
109	3-105	76
101	3-105	64
107	3-105	91
108	3-105	78
101	6-166	85
107	6-166	79
108	6-166	81
```

### teacher.txt
		教工编号   教工姓名	 教工性别		教工出生年月	职称		教工所在部门 
```javascript
804	李诚	男	1958-12-02	副教授	计算机系
856	张旭	男	1969-03-12	讲师	电子工程系
825	王萍	女	1972-05-05	助教	计算机系
831	刘冰	女	1977-08-14	助教	电子工程系
```

### 上题

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419105349318.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70 =300x300)

#### 1.查询Student表中“95031”班或性别为“女”的同学记录。

#### 2.以Class降序,升序查询Student表的所有记录。

#### 3.以Cno升序、Degree降序查询Score表的所有记录。

#### 4.查询“95031”班的学生。

#### 5.查询Score表中的最高分的学生学号和课程号。（子查询或者排序）


#### 6.查询每门课的平均成绩。
#### 7.查询Score表中至少有5名学生选修的并以3开头的课程的平均分数。
#### 8.查询分数大于70，小于90的Sno列。
#### 9.查询所有学生的Sname、Cno和Degree列。
#### 10.查询所有学生的Sno、Cname和Degree列。
#### 11.查询所有学生的Sname、Cname和Degree列。
#### 12.查询“95033”班学生的平均分。


#### 13.查询所有选修“计算机导论”课程的“女”同学的成绩表。

#### 14.查询选修“3-105”课程的成绩高于“109”号同学成绩的所有同学的记录。

#### 15.查询score中选学多门课程的同学中分数为非最高分成绩的记录。


#### 16.查询成绩高于学号为“109”、课程号为“3-105”的成绩的所有记录。

#### 17.查询和学号为105的同学同年出生的所有学生的Sno、Sname和Sbirthday列。

#### 18.查询“张旭“教师任课的学生成绩

#### 19.查询选修某课程的同学人数多于4人的教师姓名
#### 20.查询95033班和95031班全体学生的记录

#### 21.查询存在有85分以上成绩的课程Cno

#### 22.查询出“计算机系“教师所教课程的成绩表

#### 23.查询“计算机系”与“电子工程系“不同职称的教师的Tname和Prof

#### 24.查询选修编号为“3-105“课程且成绩至少高于选修编号为“3-245”的同学的Cno、Sno和Degree,并按Degree从高到低次序排序

#### 25.查询选修编号为“3-105”且成绩高于选修编号为“3-245”课程的同学的Cno、Sno和Degree

#### 26.查询所有教师和同学的name、sex和birthday

#### 27.查询所有“女”教师和“女”同学的name、sex和birthday. 

#### 28.查询成绩比该课程平均成绩低的同学的成绩表

#### 29.查询所有任课教师的Tname和Depart

#### 30.查询所有未讲课的教师的Tname和Depart

#### 31.查询至少有2名男生的班号

#### 32.查询Student表中不姓“王”的同学记录

#### 33.查询Student表中每个学生的姓名和年龄。

#### 34.查询Student表中最大和最小的Sbirthday日期值。（时间格式最大值,最小值）

#### 35.以班号和年龄从大到小的顺序查询Student表中的全部记录。 查询结果排序

#### 36.查询“男”教师及其所上的课程

#### 37.查询最高分同学的Sno、Cno和Degree列

#### 38.查询和“李军”同性别的所有同学的Sname

#### 39.查询和“李军”同性别并同班的同学Sname

#### 40.查询所有选修“计算机导论”课程的“男”同学的成绩表

#### 41.查询Student表中的所有记录的Sname、Ssex和Class列

#### 42.查询教师所有的单位即不重复的Depart列

#### 43.查询Student表的所有记录

#### 44.查询Score表中成绩在60到80之间的所有记录

#### 45.查询Score表中成绩为85，86或88的记录

***

## 答案
<font color='red'>声明：</font><font color='#BE77FF'>下面的答案均为博主自己的解法，结果均经得起测试，如有纰漏，烦请大佬指点。</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200419105657680.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
***
```java
import org.apache.spark.SparkContext
import org.apache.spark.rdd.RDD
import org.apache.spark.sql.{Dataset, SparkSession}

/*
 * @Auther: Alice菌
 * @Date: 2020/4/18 08:14
 * @Description: 
    流年笑掷 未来可期。以梦为马,不负韶华!
 */
object demo06 {

  // 定义样例类封装数据
  case class Student(Sno:String,Sname:String,Ssex:String,Sbirthday:String,Class:String)
  case class Course(Cno:String,Cname:String,Tno:String)
  case class Score(Sno:String,Cno:String,Degree:String)
  case class Teacher(Tno:String,Tname:String,Tsex:String,Tbirthday:String,Prof:String,Depart:String)


  def main(args: Array[String]): Unit = {


    // 1. 创建SparkSession
    val spark: SparkSession = SparkSession.builder().master("local[*]").appName("demo06").getOrCreate()

    val sc: SparkContext = spark.sparkContext

    // 设置日志级别
    sc.setLogLevel("WARN")

    // 读取test文件
   val rdd1: RDD[String] = sc.textFile("E:\\BigData\\05-Spark\\0417\\4.16号练习题50道2.0\\data\\student.txt")
   val rdd2: RDD[String] = sc.textFile("E:\\BigData\\05-Spark\\0417\\4.16号练习题50道2.0\\data\\course.txt")
   val rdd3: RDD[String] = sc.textFile("E:\\BigData\\05-Spark\\0417\\4.16号练习题50道2.0\\data\\score.txt")
   val rdd4: RDD[String] = sc.textFile("E:\\BigData\\05-Spark\\0417\\4.16号练习题50道2.0\\data\\teacher.txt")

    val student: RDD[Student] = rdd1.map(x=>{val str = x.split("\t");Student(str(0),str(1),str(2),str(3),str(4))})
    val course: RDD[Course] = rdd2.map(x=>{val str = x.split("\t");Course(str(0),str(1),str(2))})
    val score: RDD[Score] = rdd3.map(x=>{val str = x.split("\t");Score(str(0),str(1),str(2))})
    val teacher: RDD[Teacher] = rdd4.map(x=>{val str = x.split("\t");Teacher(str(0),str(1),str(2),str(3),str(4),str(5))})

    // 导入隐式转换
    import spark.implicits._

    // 将RDD转换成DF
    val stuframe = student.toDF()
    val couframe = course.toDF()
    val scoframe = score.toDF()
    val teaframe = teacher.toDF()

    // 创建临时表
    stuframe.createOrReplaceTempView("student")
    couframe.createOrReplaceTempView("course")
    scoframe.createOrReplaceTempView("score")
    teaframe.createOrReplaceTempView("teacher")


    /* 6. 查询Student表中“95031”班或性别为“女”的同学记录*/
    spark.sql("Select * from student where sno = '95031' or ssex = '女'").show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|107|   王丽|   女|1976-01-23|95033|
    //|109|   王芳|   女|1975-02-10|95031|
    //+---+-----+----+----------+-----+

    /* 7. 以Class降序,升序查询Student表的所有记录 */
    spark.sql("Select * from student order by class desc").show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|107|   王丽|   女|1976-01-23|95033|
    //|101|   李军|   男|1976-02-20|95033|
    //|108|   丘东|   男|1977-09-01|95033|
    //|105|   匡明|   男|1975-10-02|95031|
    //|109|   王芳|   女|1975-02-10|95031|
    //|103|   陆君|   男|1974-06-03|95031|
    //+---+-----+----+----------+-----+

    /* 8. 以Cno升序、Degree降序查询Score表的所有记录。 */
    spark.sql("Select * from score order by cno,degree desc").show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-105|    92|
    //|107|3-105|    91|
    //|105|3-105|    88|
    //|108|3-105|    78|
    //|109|3-105|    76|
    //|101|3-105|    64|
    //|103|3-245|    86|
    //|105|3-245|    75|
    //|109|3-245|    68|
    //|101|6-166|    85|
    //|108|6-166|    81|
    //|107|6-166|    79|
    //+---+-----+------+

    /* 9. 查询“95031”班的学生。 */
    spark.sql("Select sname from student where class = '95031'").show()
    //+-----+
    //|sname|
    //+-----+
    //|   匡明|
    //|   王芳|
    //|   陆君|
    //+-----+

    /* 10. 查询Score表中的最高分的学生学号和课程号。（子查询或者排序） */
    spark.sql("Select sno,cno from score order by degree desc limit 1").show()
    //+---+-----+
    //|sno|  cno|
    //+---+-----+
    //|103|3-105|
    //+---+-----+

    /* 11. 查询每门课的平均成绩。  */
    spark.sql("Select first(course.Cname),avg(degree) from score,course where score.Cno = course.Cno  group by score.Cno").show()
    //+-------------------+---------------------------+
    //|first(Cname, false)|avg(CAST(degree AS DOUBLE))|
    //+-------------------+---------------------------+
    //|               数字电路|          81.66666666666667|
    //|               操作系统|          76.33333333333333|
    //|              计算机导论|                       81.5|
    //+-------------------+---------------------------+

    /* 12. 查询Score表中至少有5名学生选修的并以3开头的课程的平均分数。  */
    spark.sql("Select first(cno),avg(Degree) from score where cno like '3%' group by cno having count(cno) >= 5").show()
    //+-----------------+---------------------------+
    //|first(cno, false)|avg(CAST(Degree AS DOUBLE))|
    //+-----------------+---------------------------+
    //|            3-105|                       81.5|
    //+-----------------+---------------------------+

    /* 13.查询分数大于70，小于90的Sno列  */
    spark.sql("Select sno from score where Degree > 70 and Degree < 90 ").show()
    //+---+
    //|sno|
    //+---+
    //|103|
    //|105|
    //|105|
    //|109|
    //|108|
    //|101|
    //|107|
    //|108|
    //+---+

    /* 14.查询所有学生的Sname、Cno和Degree列。 */
    spark.sql("select sname,cno,degree from score,student where score.sno = student.sno").show()
    //+-----+-----+------+
    //|sname|  cno|degree|
    //+-----+-----+------+
    //|   李军|3-105|    64|
    //|   李军|6-166|    85|
    //|   王丽|3-105|    91|
    //|   王丽|6-166|    79|
    //|   陆君|3-245|    86|
    //|   陆君|3-105|    92|
    //|   丘东|3-105|    78|
    //|   丘东|6-166|    81|
    //|   匡明|3-245|    75|
    //|   匡明|3-105|    88|
    //|   王芳|3-245|    68|
    //|   王芳|3-105|    76|
    //+-----+-----+------+

    /* 15.查询所有学生的Sno、Cname和Degree列。 */
    spark.sql("select sno,cname,degree from score,course where score.cno=course.cno").show()
    //+---+-----+------+
    //|sno|cname|degree|
    //+---+-----+------+
    //|101| 数字电路|    85|
    //|107| 数字电路|    79|
    //|108| 数字电路|    81|
    //|103| 操作系统|    86|
    //|105| 操作系统|    75|
    //|109| 操作系统|    68|
    //|103|计算机导论|    92|
    //|105|计算机导论|    88|
    //|109|计算机导论|    76|
    //|101|计算机导论|    64|
    //|107|计算机导论|    91|
    //|108|计算机导论|    78|
    //+---+-----+------+

    /* 16.查询所有学生的Sname、Cname和Degree列。 */
    spark.sql(
      """
        |select sname,cname,degree from score,course,student
        |where score.cno = course.cno and student.sno = score.sno
      """.stripMargin).show()
     //+-----+-----+------+
    //|sname|cname|degree|
    //+-----+-----+------+
    //|   李军| 数字电路|    85|
    //|   李军|计算机导论|    64|
    //|   王丽| 数字电路|    79|
    //|   王丽|计算机导论|    91|
    //|   陆君| 操作系统|    86|
    //|   陆君|计算机导论|    92|
    //|   丘东| 数字电路|    81|
    //|   丘东|计算机导论|    78|
    //|   匡明| 操作系统|    75|
    //|   匡明|计算机导论|    88|
    //|   王芳| 操作系统|    68|
    //|   王芳|计算机导论|    76|
    //+-----+-----+------+

    /* 17.查询“95033”班学生的平均分。*/
    spark.sql("select first(class),avg(Degree) from student,score where student.sno = score.sno and class = '95033'").show()
    //+-------------------+---------------------------+
    //|first(class, false)|avg(CAST(Degree AS DOUBLE))|
    //+-------------------+---------------------------+
    //|              95033|          79.66666666666667|
    //+-------------------+---------------------------+

    /* 18.查询所有选修“计算机导论”课程的“女”同学的成绩表。 */
    spark.sql(
      """
        |select sname,cname,ssex,Degree from student,course,score
        |where student.sno = score.sno and course.cno=score.cno
        |and ssex = "女" and cname = "计算机导论"
      """.stripMargin).show()
    //|sname|cname|ssex|Degree|
    //+-----+-----+----+------+
    //|   王丽|计算机导论|   女|    91|
    //|   王芳|计算机导论|   女|    76|
    //+-----+-----+----+------+

    /* 19.查询选修“3-105”课程的成绩高于“109”号同学成绩的所有同学的记录。*/
    spark.sql(
      """
        |select * from score where cno = "3-105" and degree > (select degree from score where sno = "109" and cno = "3-105" )
        |
      """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-105|    92|
    //|105|3-105|    88|
    //|107|3-105|    91|
    //|108|3-105|    78|
    //+---+-----+------+

    /* 20.查询score中选学多门课程的同学中分数为非最高分成绩的记录。  */
//    spark.sql(
//      """
//        |select first(sno),first(cno),first(degree) from score a where sno in
//        |(select sno from score group by sno having count(*)>1)　　　　
//        |and degree<(select max(degree) from score b where a.cno=b.cno)　　　　　　
//      """.stripMargin).show()
    spark.sql("select * from score a where sno in (select sno from score group by sno having count(*)>1) and degree<(select max(degree) from score b where a.cno=b.cno )").show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|107|6-166|    79|
    //|108|6-166|    81|
    //|105|3-245|    75|
    //|109|3-245|    68|
    //|101|3-105|    64|
    //|107|3-105|    91|
    //|108|3-105|    78|
    //|105|3-105|    88|
    //|109|3-105|    76|
    //+---+-----+------+

    /* 21.查询成绩高于学号为“109”、课程号为“3-105”的成绩的所有记录。 */
     spark.sql("select * from score where degree > (select degree from score where sno = '109' and cno = '3-105')").show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-245|    86|
    //|103|3-105|    92|
    //|105|3-105|    88|
    //|107|3-105|    91|
    //|108|3-105|    78|
    //|101|6-166|    85|
    //|107|6-166|    79|
    //|108|6-166|    81|
    //+---+-----+------+

    /* 22. 查询和学号为105的同学同年出生的所有学生的Sno、Sname和Sbirthday列。*/

    spark.udf.register("sub",(str:String,num1:Int,num2:Int)=>str.substring(num1,num2))

    spark.sql("select sno,sname,sbirthday from student where sub(Sbirthday,0,5) = (select sub(Sbirthday,0,5) from student where sno = '105') ").show()
    //+---+-----+----------+
    //|sno|sname| sbirthday|
    //+---+-----+----------+
    //|105|   匡明|1975-10-02|
    //|109|   王芳|1975-02-10|
    //+---+-----+----------+

    /* 23. 查询“张旭“教师任课的学生成绩 */
    spark.sql(
      """
        |select  sno,degree from score,course,teacher
        |where tname = "张旭" and teacher.tno = course.tno
        |and course.Cno = score.Cno
      """.stripMargin).show()
    //+---+------+
    //|sno|degree|
    //+---+------+
    //|101|    85|
    //|107|    79|
    //|108|    81|
    //+---+------+

    /* 24. 查询选修某课程的同学人数多于4人的教师姓名。 */
     spark.sql(
       """
         |select first(tname) from teacher,score,course
         |where teacher.tno = course.tno
         |and course.Cno = score.Cno
         |group by score.Cno having count(score.Cno) > 4
       """.stripMargin).show()

    //+-------------------+
    //|first(tname, false)|
    //+-------------------+
    //|                 王萍|
    //+-------------------+

    /* 25. 查询95033班和95031班全体学生的记录。 */
     spark.sql("select * from student where class= '95033' or class = '95031'").show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|108|   丘东|   男|1977-09-01|95033|
    //|105|   匡明|   男|1975-10-02|95031|
    //|107|   王丽|   女|1976-01-23|95033|
    //|101|   李军|   男|1976-02-20|95033|
    //|109|   王芳|   女|1975-02-10|95031|
    //|103|   陆君|   男|1974-06-03|95031|
    //+---+-----+----+----------+-----+

    /* 26. 查询存在有85分以上成绩的课程Cno.  */
     spark.sql("select distinct cno from score where degree >= 85").show()
    //+-----+
    //|  cno|
    //+-----+
    //|6-166|
    //|3-245|
    //|3-105|
    //+-----+

    /* 27. 查询出“计算机系“教师所教课程的成绩表。 */
     spark.sql(
       """
         |select  score.*  from score,Course,teacher
         |where  teacher.tno = course.tno
         |and course.Cno = score.Cno
         |and teacher.Depart = "计算机系"
       """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-105|    92|
    //|105|3-105|    88|
    //|109|3-105|    76|
    //|101|3-105|    64|
    //|107|3-105|    91|
    //|108|3-105|    78|
    //|103|3-245|    86|
    //|105|3-245|    75|
    //|109|3-245|    68|
    //+---+-----+------+

    /* 28. 查询“计算机系”与“电子工程系“不同职称的教师的Tname和Prof */
    spark.sql(
      """
        |select tname,prof from teacher where depart = '计算机系' and prof not in
        |(select prof from teacher where depart = '电子工程系')
      """.stripMargin).show()
    //+-----+----+
    //|tname|prof|
    //+-----+----+
    //|   李诚| 副教授|
    //+-----+----+

    /* 29. 查询选修编号为“3-105“课程且成绩至少高于选修编号为“3-245”的同学的Cno、Sno和Degree,并按Degree从高到低次序排序。 */
     spark.sql(
       """
         |select s1.cno,s1.sno,s1.degree from score s1,score s2
         |where s1.cno = "3-105" and s2.cno = "3-245"
         |and s1.Sno = s2.Sno
         |and s1.degree > s2. degree
         |order by s1.degree desc
       """.stripMargin).show()

    //+-----+---+------+
    //|  cno|sno|degree|
    //+-----+---+------+
    //|3-105|103|    92|
    //|3-105|105|    88|
    //|3-105|109|    76|
    //+-----+---+------+

    /* 30. 查询选修编号为“3-105”且成绩高于选修编号为“3-245”课程的同学的Cno、Sno和Degree. */
    spark.sql(
      """
        |select cno,sno,degree from score
        |where cno = "3-105" and degree > (select max(degree) from score where cno = "3-245")
      """.stripMargin).show()
    //+-----+---+------+
    //|  cno|sno|degree|
    //+-----+---+------+
    //|3-105|103|    92|
    //|3-105|105|    88|
    //|3-105|107|    91|
    //+-----+---+------+

    /* 31. 查询所有教师和同学的name、sex和birthday. */
     spark.sql(
       """
         |select distinct Sname as name,Ssex as sex,Sbirthday as birthday from student
         |union
         |select distinct Tname as name,Tsex as sex,Tbirthday as birthday from Teacher
         |
       """.stripMargin).show()
    //+----+---+----------+
    //|name|sex|  birthday|
    //+----+---+----------+
    //|  李诚|  男|1958-12-02|
    //|  张旭|  男|1969-03-12|
    //|  王萍|  女|1972-05-05|
    //|  李军|  男|1976-02-20|
    //|  匡明|  男|1975-10-02|
    //|  刘冰|  女|1977-08-14|
    //|  陆君|  男|1974-06-03|
    //|  王芳|  女|1975-02-10|
    //|  王丽|  女|1976-01-23|
    //|  丘东|  男|1977-09-01|
    //+----+---+----------+

    /* 32. 查询所有“女”教师和“女”同学的name、sex和birthday. */
    spark.sql(
      """
        |select distinct Sname as name,Ssex as sex,Sbirthday as birthday from student where Ssex = "女"
        |union
        |select distinct Tname as name,Tsex as sex,Tbirthday as birthday from Teacher where Tsex = "女"
        |
      """.stripMargin).show()
    //+----+---+----------+
    //|name|sex|  birthday|
    //+----+---+----------+
    //|  王萍|  女|1972-05-05|
    //|  刘冰|  女|1977-08-14|
    //|  王芳|  女|1975-02-10|
    //|  王丽|  女|1976-01-23|
    //+----+---+----------+

    /* 33. 查询成绩比该课程平均成绩低的同学的成绩表。 */
     spark.sql(
       """
         |select * from score s1
         |where degree < (select avg(degree) from score s2 where s1.Cno = s2.Cno)
       """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|107|6-166|    79|
    //|108|6-166|    81|
    //|105|3-245|    75|
    //|109|3-245|    68|
    //|109|3-105|    76|
    //|101|3-105|    64|
    //|108|3-105|    78|
    //+---+-----+------+

    /* 34. 查询所有任课教师的Tname和Depart. */
     spark.sql(
       """
         |select tname,depart from teacher
         |where tno in (select tno from course where cno in (select distinct cno from score) )
       """.stripMargin).show()
    //+-----+------+
    //|tname|depart|
    //+-----+------+
    //|   张旭| 电子工程系|
    //|   王萍|  计算机系|
    //|   李诚|  计算机系|
    //+-----+------+

    /* 35. 查询所有未讲课的教师的Tname和Depart.  */
   spark.sql(
     """
       |select tname,depart from teacher
       |where tno not in (select tno from course where cno in (select distinct cno from score) )
     """.stripMargin).show()
    //+-----+------+
    //|tname|depart|
    //+-----+------+
    //|   刘冰| 电子工程系|
    //+-----+------+

    /* 36. 查询至少有2名男生的班号。 */
    spark.sql(
      """
        |select class from student
        |where ssex = "男"
        |group by class having count(class) >= 2
      """.stripMargin).show()
    //+-----+
    //|class|
    //+-----+
    //|95033|
    //|95031|
    //+-----+

    /* 37. 查询Student表中不姓“王”的同学记录。 */
     spark.sql(
       """
         |select * from student
         |where sname not like "王%"
       """.stripMargin).show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|108|   丘东|   男|1977-09-01|95033|
    //|105|   匡明|   男|1975-10-02|95031|
    //|101|   李军|   男|1976-02-20|95033|
    //|103|   陆君|   男|1974-06-03|95031|
    //+---+-----+----+----------+-----+

    /* 38. 查询Student表中每个学生的姓名和年龄。将函数运用到spark sql中去计算，可以直接拿String的类型计算不需要再转换成数值型 默认是会转换成Double类型计算浮点型转整型 */

    spark.sql(
      """
        |select Sname,year(current_date)-year(Sbirthday) from student
      """.stripMargin).show()
    //+-----+------------------------------------------------------+
    //|Sname|(year(current_date()) - year(CAST(Sbirthday AS DATE)))|
    //+-----+------------------------------------------------------+
    //|   丘东|                                                    43|
    //|   匡明|                                                    45|
    //|   王丽|                                                    44|
    //|   李军|                                                    44|
    //|   王芳|                                                    45|
    //|   陆君|                                                    46|
    //+-----+------------------------------------------------------+

    /* 39. 查询Student表中最大和最小的Sbirthday日期值。 时间格式最大值,最小值*/
    spark.sql(
      """
        |select max(Sbirthday),min(Sbirthday) from student
      """.stripMargin).show()
    //+--------------+--------------+
    //|max(Sbirthday)|min(Sbirthday)|
    //+--------------+--------------+
    //|    1977-09-01|    1974-06-03|
    //+--------------+--------------+

    /* 40. 以班号和年龄从大到小的顺序查询Student表中的全部记录。 查询结果排序*/
    spark.sql(
      """
        |select * from student order by class desc,year(current_date)-year(Sbirthday) desc
      """.stripMargin).show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|107|   王丽|   女|1976-01-23|95033|
    //|101|   李军|   男|1976-02-20|95033|
    //|108|   丘东|   男|1977-09-01|95033|
    //|103|   陆君|   男|1974-06-03|95031|
    //|105|   匡明|   男|1975-10-02|95031|
    //|109|   王芳|   女|1975-02-10|95031|
    //+---+-----+----+----------+-----+

    /* 41. 查询“男”教师及其所上的课程。 */
    spark.sql(
      """
        |select cname from course,teacher
        |where teacher.Tno = course.Tno and Tsex = "男"
      """.stripMargin).show()
    //+-----+
    //|cname|
    //+-----+
    //| 数字电路|
    //| 操作系统|
    //+-----+

    /* 42. 查询最高分同学的Sno、Cno和Degree列。  */
    spark.sql(
      """
        |select * from score
        |where degree = (select max(degree) from score)
      """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-105|    92|
    //+---+-----+------+

    /* 43. 查询和“李军”同性别的所有同学的Sname.*/
    spark.sql(
      """
        |select sname from student
        |where ssex = (select ssex from student where sname = "李军") and sname != "李军"
      """.stripMargin).show()
    //+-----+
    //|sname|
    //+-----+
    //|   丘东|
    //|   匡明|
    //|   李军|
    //|   陆君|
    //+-----+

    /* 44. 查询和“李军”同性别并同班的同学Sname. */
    spark.sql(
      """
        |select distinct s1.sname from student s1,student s2
        |where s1.ssex = s2.ssex and s1.class = s2.class
        |and s1.sname != "李军"
      """.stripMargin).show()
    //+-----+
    //|sname|
    //+-----+
    //|   陆君|
    //|   匡明|
    //|   王芳|
    //|   王丽|
    //|   丘东|
    //+-----+

    /* 45. 查询所有选修“计算机导论”课程的“男”同学的成绩表。 */
    spark.sql(
      """
        |select degree from score,student,course
        |where score.cno = course.cno
        |and score.sno = student.sno
        |and student.ssex = "男"
        |and course.cname = "计算机导论"
      """.stripMargin).show()
    //+------+
    //|degree|
    //+------+
    //|    64|
    //|    92|
    //|    78|
    //|    88|
    //+------+

    /* 46. 查询Student表中的所有记录的Sname、Ssex和Class列。 */
     spark.sql(
       """
         |select sname,ssex,class from student
       """.stripMargin).show()
    //+-----+----+-----+
    //|   丘东|   男|95033|
    //|   匡明|   男|95031|
    //|   王丽|   女|95033|
    //|   李军|   男|95033|
    //|   王芳|   女|95031|
    //|   陆君|   男|95031|
    //+-----+----+-----+

    /* 47. 查询教师所有的单位即不重复的Depart列。*/
     spark.sql(
       """
         |select distinct depart from teacher
       """.stripMargin).show()
    //+------+
    //|depart|
    //+------+
    //| 电子工程系|
    //|  计算机系|
    //+------+

    /* 48. 查询Student表的所有记录 */
    spark.sql(
      """
        |select * from student
      """.stripMargin).show()
    //+---+-----+----+----------+-----+
    //|Sno|Sname|Ssex| Sbirthday|Class|
    //+---+-----+----+----------+-----+
    //|108|   丘东|   男|1977-09-01|95033|
    //|105|   匡明|   男|1975-10-02|95031|
    //|107|   王丽|   女|1976-01-23|95033|
    //|101|   李军|   男|1976-02-20|95033|
    //|109|   王芳|   女|1975-02-10|95031|
    //|103|   陆君|   男|1974-06-03|95031|
    //+---+-----+----+----------+-----+

    /* 49. 查询Score表中成绩在60到80之间的所有记录。 */
    spark.sql(
      """
        |select * from score where degree > 60 and degree < 80
      """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|105|3-245|    75|
    //|109|3-245|    68|
    //|109|3-105|    76|
    //|101|3-105|    64|
    //|108|3-105|    78|
    //|107|6-166|    79|
    //+---+-----+------+

    /* 50. 查询Score表中成绩为85，86或88的记录。 */
     spark.sql(
       """
         |select * from score where Degree in (85,86,88)
       """.stripMargin).show()
    //+---+-----+------+
    //|Sno|  Cno|Degree|
    //+---+-----+------+
    //|103|3-245|    86|
    //|105|3-105|    88|
    //|101|6-166|    85|
    //+---+-----+------+


  }
}
```

***
## 结语
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，本篇关于SparkSQL的练习题就分享到这里，后续还会继续分享其他类型的习题，<font color='#B15BFF'>**受益的朋友或对大数据技术感兴趣的伙伴记得点赞关注支持一波(＾Ｕ＾)ノ~ＹＯ**</font>

![](https://img-blog.csdnimg.cn/20210119222335538.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)