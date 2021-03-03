&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;都说“金九银十”，马上十月份即将结束，相信还有相当多的小伙伴没找到合适的工作。在笔试过程中，总会出现那么一两道“有趣”的SQL题，来检测应聘者的一个逻辑思维，这对于初入职场的“小白”也是非常不友好。不用担心，本篇博客，博主整理了几道在面试中高频出现的“SQL”笔试题，助你在接下来的面试中一往无前，势如破竹!
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013094143159.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)



***

## 1、查询连续登陆3天以上的用户
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是一道非常经典的问题，这里提供其中一种思路。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表信息如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013100733902.png?==,size_16,color_FFFFFF,t_70#pic_center)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
### step1: 用户登录日期去重
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为一个用户同一天可能登录多次，所以我们首先需要用用户登录日期去重。

```sql
select DISTINCT date(date) as "日期",id from demo01;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询结果:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013101051631.png?==,size_16,color_FFFFFF,t_70#pic_center)

### step2: 用row_number() over()函数计数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有了第一步去重后的结果，我们可以对其进行开窗，以id分组，日期升序排序，获取到每个日期的排名。
```sql
select *,row_number() over(PARTITION by id order by `日期`) as cum from (select DISTINCT date(date) as `日期`,id from demo01)a;
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询结果：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013110047271.png?==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相信看到这里，各位小伙伴已经看出其中的“玄机”了~为什么我们需要在这一步对时间进行一个排序呢？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以发现，用row_number开窗之后的名次是连续的，那么如果日期也是连续的，它们的差值不就是一个固定的值了吗?


### step3:日期减去计数值得到结果
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为菌哥这里演示用的是hql，所以这里获取日期差值使用了`date_sub`函数。

```sql
select *,date_sub(`日期`,cum) as `结果` from (select *,row_number() over(PARTITION by id order by `日期`) as cum from (select DISTINCT date(date) as `日期`,id from demo01)a)b;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询结果：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013111712629.png?==,size_16,color_FFFFFF,t_70#pic_center)

### step4:根据id和结果分组并计算count
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最后一步，我们直接根据step3中获取到的差值，根据id和差值进行一个分组求count即可。如果是要求连续登录3天以上，我们直接判断 count 的个数大于等于3即可。


```sql
select id,count(*) from (select *,date_sub(`日期`,cum) as `结果` from (select *,row_number() over(PARTITION by id order by `日期`) as cum from (select DISTINCT date(date) as `日期`,id from demo01)a)b)c GROUP BY id,`结果` having count(*)>=3;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行结果:
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013114915744.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;答案已经出来了，id为1和3的用户至少连续登录了3天及以上，他们分别连续登录的时长为3天和4天。


## 2、统计每个用户的累计访问次数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个同样也是经常在笔试中出现的题目，大家可以根据作者的思路回顾一下:


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;表信息如下图：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013130135947.png?==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要求使用SQL统计出每个用户的累积访问次数，如下表所示：
| 用户id | 月份 |小计|累积
|--|--|--|--|
|u01  | 2017-01 |11|11
|u01|2017-02|12|23
|u02|2017-01|12|12
|u03|2017-01|8|8
|u04|2017-01|3|3


### step1: 修改数据格式

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从结果反推，需要查询实现按照 **年-月** 分组的数据，所以我们这一步先对原数据进行一个处理。

```sql
select
     userId,
     date_format(regexp_replace(visitDate,'/','-'),'yyyy-MM') mn,
     visitCount
from
     action;t1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;处理结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013131502308.png?=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

### step2: 计算每人单月访问量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了让子查询看起来更加的美观，我们这里先用t1代替上一步的结果。通过这一步，我们就可以获取到每个用户，每个月的访问量。

```sql
select
    userId,
    mn,
    sum(visitCount) mn_count
from
    t1
group by userId,mn;t2
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查询的结果：
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013131843903.png#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
### step3: 按月累计计算访问量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们将第二步的结果用变量 t2 来表示。到这一步，我们用一个sum开窗函数，对userid进行分组，mn时间进行排序即可大功告成。

```sql
select
    userId,
    mn,
    mn_count,
    sum(mn_count) over(partition by userId order by mn) mn_all
from t2;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;最终结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013132533717.png#pic_center)
### 完整SQL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<font color='red'>温馨提示:</font><font color='gray'>上述的步骤展示的都是不完整的SQL，每步使用变量代替前一步的SQL语句只是为了方便给大家展示，实际上运行的结果都是作者将完整的SQL放进去跑的哈~</font>

```sql
select
    userId,
    mn,
    mn_count,
    sum(mn_count) over(partition by userId order by mn) mn_all
from
(   select
        userId,
        mn,
        sum(visitCount) mn_count
    from
         (select
             userId,
             date_format(regexp_replace(visitDate,'/','-'),'yyyy-MM') mn,
             visitCount
         from
             action)t1
group by userId,mn)t2;
```

## 3、分组TopN
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有50W个店铺，每个顾客访客访问任何一个店铺的任何一个商品时都会产生一条访问日志，访问日志存储的表名为Visit，访客的用户id为user_id，被访问的店铺名称为shop。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013133846379.png?==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;需求：每个店铺访问次数top3的访客信息。输出店铺名称、访客id、访问次数。

### step1：查询每个店铺被每个用户访问次数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为我们最终需要获取每个店铺访问量top3的用户信息，所以在这一步，我们就先把每个店铺的每个用户的访问次数计算出来。

```sql
select shop,user_id,count(*) ct
from visit
group by shop,user_id;t1
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;计算结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/2020101313450784.png?=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
### step2:计算每个店铺被用户访问次数排名
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有了第一步每个店铺下所被访问用户的访问量，我们想获取前三，毫无疑问，我们需要使用到开窗函数 rank。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可能就有朋友问了，为什么不能用 **row_number** ？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;主要还是 **row_number** 对于相同数据的排名不是一样的，如果我们取Topic3,出现了相同访问次数的数据，那我们肯定都得保留下来的对吧~~

```sql
select shop,user_id,ct,rank() over(partition by shop order by ct) rk
from t1;t2
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;计算结果：

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013135308366.png?=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
### step3: 取每个店铺排名前3的数据
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有了 step2 的结果，我们想要取每个店铺前三的数据岂不是轻而易举~

```sql
select shop,user_id,ct
from t2
where rk<=3;
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;计算结果:

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201013135620764.png?=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
### 完整SQL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，结果已经查询出来了，这里把上面step的SQL整合到一起~

```sql
select 
shop,
user_id,
ct
from
(select 
shop,
user_id,
ct,
rank() over(partition by shop order by ct) rk
from 
(select 
shop,
user_id,
count(*) ct
from visit
group by 
shop,
user_id)t1
)t2
where rk<=3;
```
## 结语
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们不论是看书还是刷题，不在于数量多少，而一定要求“精”。这就要求我们学会去思考，学会举一反三。真正具备解题能力的人，我相信一定不是把时间花在大量刷题上，而是懂得从不同类型的习题上，及时总结复习的人。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;以上3道SQL“小菜”怕是满足不了大伙，以后有机会再为大家总结些别的题目，本篇文章到这里就结束了。对技术宇宙充满好奇，喜欢本文的朋友，可以扫码关注作者原创公众号【大数据梦想家】，我们下期见!

<img src="https://img-blog.csdnimg.cn/20210119135940253.png" width = 99% height = 50% />











