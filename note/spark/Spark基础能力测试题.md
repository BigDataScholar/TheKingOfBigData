&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为这段时间在学习Spark，所以本篇博客为大家带来关于Spark的综合性练习一道。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='#00CACA'>**码字不易，先赞后看，养成习惯!**</font>

![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405091407104.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)



&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先让我们准备好该题所需的数据 test.txt

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;数据结构如下依次是：班级 姓名 年龄 性别 科目 成绩
```javascript
12 宋江 25 男 chinese 50
12 宋江 25 男 math 60
12 宋江 25 男 english 70
12 吴用 20 男 chinese 50
12 吴用 20 男 math 50
12 吴用 20 男 english 50
12 杨春 19 女 chinese 70
12 杨春 19 女 math 70
12 杨春 19 女 english 70
13 李逵 25 男 chinese 60
13 李逵 25 男 math 60
13 李逵 25 男 english 70
13 林冲 20 男 chinese 50
13 林冲 20 男 math 60
13 林冲 20 男 english 50
13 王英 19 女 chinese 70
13 王英 19 女 math 80
13 王英 19 女 english 70
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <font color='#84C1FF'>**题目先为大家展示出来，答案见文末~</font>**


#### 1.   读取文件的数据test.txt

#### 2.   一共有多少个小于20岁的人参加考试？


#### 3.   一共有多少个等于20岁的人参加考试？

#### 4.   一共有多少个大于20岁的人参加考试？

#### 5.   一共有多个男生参加考试？

#### 6.   一共有多少个女生参加考试？
#### 7.   12班有多少人参加考试？
#### 8.  13班有多少人参加考试？
#### 9.   语文科目的平均成绩是多少？

#### 10. 数学科目的平均成绩是多少？

#### 11. 英语科目的平均成绩是多少？

#### 12. 每个人平均成绩是多少？

#### 13. 12班平均成绩是多少？

#### 14. 12班男生平均总成绩是多少？

#### 15. 12班女生平均总成绩是多少？

#### 16. 13班平均成绩是多少？

#### 17. 13班男生平均总成绩是多少？

#### 18. 13班女生平均总成绩是多少？

#### 19. 全校语文成绩最高分是多少？

#### 20. 12班语文成绩最低分是多少？

#### 21. 13班数学最高成绩是多少？

#### 22. 总成绩大于150分的12班的女生有几个？

#### 23. 总成绩大于150分，且数学大于等于70，且年龄大于等于19岁的学生的平均成绩是多少？

***

<font color='#CA8EFF' size='4'>答案马上来咯，Ready？Go</font>
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200405091623146.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

```python

/*
 * @Auther: Alice菌
 * @Date: 2020/4/5 09:18
 * @Description: 
    流年笑掷 未来可期。以梦为马,不负韶华!
 */
object test {

  def main(args: Array[String]): Unit = {

    val config = new SparkConf().setMaster("local[*]").setAppName("test")
    val sc = new SparkContext(config)

    // 1.读取文件的数据test.txt
    // 返回包含所有行数据的列表
    val data: RDD[String]  = sc.textFile("E:\\2020大数据新学年\\BigData\\05-Spark\\0403\\test.txt")

    //val value: RDD[Array[String]] = sc.makeRDD(List("12 宋江 25 男 chinese 50")).map(x=>x.split(" "))

    // 2. 一共有多少个小于20岁的人参加考试？   2
    val count1: Long = data.map(x=>x.split(" ")).filter(x=>x(2).toInt<20).groupBy(_(1)).count()


    // 3. 一共有多少个等于20岁的人参加考试？   2
    val count2: Long = data.map(x=>x.split(" ")).filter(x=>x(2).toInt==20).groupBy(_(1)).count()

    // 4. 一共有多少个大于20岁的人参加考试？   2
    val count3: Long = data.map(x=>x.split(" ")).filter(x=>x(2).toInt>20).groupBy(_(1)).count()

    // 5. 一共有多个男生参加考试？  4
    val count4: Long = data.map(x=>x.split(" ")).filter(x=>x(3).equals("男")).groupBy(_(1)).count()

    // 6.  一共有多少个女生参加考试？   2
    val count5: Long = data.map(x=>x.split(" ")).filter(x=>x(3).equals("女")).groupBy(_(1)).count()

    // 7.  12班有多少人参加考试？  3
    val count6: Long = data.map(x=>x.split(" ")).filter(x=>x(0).equals("12")).groupBy(_(1)).count()

    // 8.  13班有多少人参加考试？  3
    val count7: Long = data.map(x=>x.split(" ")).filter(x=>x(0).equals("13")).groupBy(_(1)).count()

    // 9.  语文科目的平均成绩是多少？   58.333333333333336
    val mean1 = data.map(x=>x.split(" ")).filter(x=>x(4).equals("chinese")).map(x=>x(5).toInt).mean()

    // 10.  数学科目的平均成绩是多少？  63.333333333333336
    val mean2 = data.map(x=>x.split(" ")).filter(x=>x(4).equals("math")).map(x=>x(5).toInt).mean()

    // 11. 英语科目的平均成绩是多少？  63.333333333333336
    val mean3 = data.map(x=>x.split(" ")).filter(x=>x(4).equals("english")).map(x=>x(5).toInt).mean()

    // 12. 每个人平均成绩是多少？
    //(王英,73)
    //(杨春,70)
    //(宋江,60)
    //(李逵,63)
    //(吴用,50)
    //(林冲,53)
    val every_socre: RDD[(String, Any)] = data.map(x=>x.split(" ")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum /t._2.size))

    // 13. 12班平均成绩是多少？   60.0
    var mean5 = data.map(x => x.split(" ")).filter(x => x(0).equals("12")).map(x => x(5).toInt).mean()

    // 14. 12班男生平均总成绩是多少？   165.0
    // (宋江,180)
    // (吴用,150)
    val boy12_avgsocre: Double = data.map(x=>x.split(" ")).filter(x=>x(0).equals("12") && x(3).equals("男")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).map(x=>x._2).mean()

    // 15. 12班女生平均总成绩是多少？   210.0
    // (杨春,210)
    val girl12_avgsocre: Double = data.map(x=>x.split(" ")).filter(x=>x(0).equals("12") && x(3).equals("女")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).map(x=>x._2).mean()

    // 16. 13班平均成绩是多少？     63.333333333333336
    var mean8 = data.map(x => x.split(" ")).filter(x => x(0).equals("13")).map(x => x(5).toInt).mean()

    // 17. 13班男生平均总成绩是多少？    175.0
    //(李逵,190)
    //(林冲,160)
    val boy13_avgsocre: Double = data.map(x=>x.split(" ")).filter(x=>x(0).equals("13") && x(3).equals("男")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).map(x=>x._2).mean()

    // 18. 13班女生平均总成绩是多少？
    //(王英,220)
    val girl13_avgsocre: Double = data.map(x=>x.split(" ")).filter(x=>x(0).equals("13") && x(3).equals("女")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).map(x=>x._2).mean()

    // 19. 全校语文成绩最高分是多少？    70
    var max1 = data.map(x => x.split(" ")).filter(x => x(4).equals("chinese")).map(x => x(5).toInt).max()

    // 20. 12班语文成绩最低分是多少？   50
    var max2 = data.map(x => x.split(" ")).filter(x => x(4).equals("chinese") && x(0).equals("12")).map(x => x(5).toInt).min()

    // 21. 13班数学最高成绩是多少？   80
    var max3 = data.map(x => x.split(" ")).filter(x => x(4).equals("math") && x(0).equals("13")).map(x => x(5).toInt).max()

    // 22. 总成绩大于150分的12班的女生有几个？   1
    //(杨春,210)
    val count12_gt150girl: Long = data.map(x=>x.split(" ")).filter(x=>x(0).equals("12") && x(3).equals("女")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).filter(x=>x._2>150).count()

    // 23. 总成绩大于150分，且数学大于等于70，且年龄大于等于19岁的学生的平均成绩是多少？
    //val countall: Long = data.map(x=>x.split(" ")).filter(x=>x(2).toInt>=19 && x(3).equals("女")).map(x=>(x(1),x(5).toInt)).groupByKey().map(t=>(t._1,t._2.sum)).filter(x=>x._2>150).count()
    val complex1 = data.map(x => {val line = x.split(" "); (line(0)+","+line(1)+","+line(3),line(5).toInt)})
    //(13,李逵,男 , 60)
    val complex2 = data.map(x => {val line = x.split(" "); (line(0)+","+line(1)+","+line(2)+","+line(3)+","+line(4),line(5).toInt)})
    //(12,宋江,男,chinese , 50)

    // 过滤出总分大于150的,并求出平均成绩    (13,李逵,男,(60,1))               (13,李逵,男,(190,3))             总成绩大于150                (13,李逵,男,63)
    val com1: RDD[(String, Int)] = complex1.map(x=>(x._1,(x._2,1))).reduceByKey((a, b)=>(a._1+b._1,a._2+b._2)).filter(a=>(a._2._1>150)).map(t=>(t._1,t._2._1/t._2._2))
    // 注意：reduceByKey 自定义的函数 是对同一个key值的value做聚合操作
    //(12,杨春,女 , 70)
    //(13,王英,女 , 73)
    //(12,宋江,男 , 60)
    //(13,林冲,男 , 53)
    //(13,李逵,男 , 63)

    //过滤出 数学大于等于70，且年龄大于等于19岁的学生                filter方法返回一个boolean值 【数学成绩大于70并且年龄>=19】                                       为了将最后的数据集与com1做一个join,这里需要对返回值构造成com1格式的数据
    val com2: RDD[(String, Int)] = complex2.filter(a=>{val line = a._1.split(",");line(4).equals("math") && a._2>=70 && line(2).toInt>=19}).map(a=>{val line2 = a._1.split(",");(line2(0)+","+line2(1)+","+line2(3),a._2.toInt)})
    //(12,杨春,女 , 70)
    //(13,王英,女 , 80)

    // val common: RDD[(String, (Int, Int))] = com1.join(com2)
    // common.foreach(println)
    // (12,杨春,女 , (70,70))
    // (13,王英,女 , (73,80))

    // 使用join函数聚合相同key组成的value元组
    // 再使用map函数格式化元素
    val result = com1.join(com2).map(a =>(a._1,a._2._1))
    //(12,杨春,女,70)
    //(13,王英,女,73)
    //到这里就大功告成了!!!!!!!!!!
    
  }
}
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为在博主写的代码中，对于每一题都有较为详细的注释，并且将每题的答案都标注在了题目的周围，细致的小伙伴们可以参考一下。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，本次的分享就到这里了，对大数据技术感兴趣的朋友可以点赞关注博主|ू･ω･` )

![](https://img-blog.csdnimg.cn/20210119222335538.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)