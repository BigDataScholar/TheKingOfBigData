## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇是flink 的「电商用户行为数据分析」的第6篇文章，为大家带来的是**市场营销商业指标统计分析**之**APP市场推广统计**的内容，通过本期内容的学习，你同样能够学会处理一些特定场景领域下的方法。话不多说，我们直入正题！
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
***
## 模块创建和数据准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;继续在`UserBehaviorAnalysis`下新建一个**maven module**作为子项目，命名为`MarketAnalysis`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个模块中我们没有现成的数据，所以会用自定义的测试源来产生测试数据流，或者直接用生成测试数据文件。

## APP市场推广统计
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;随着智能手机的普及，在如今的电商网站中已经有越来越多的用户来自移动端，相比起传统浏览器的登录方式，手机APP成为了更多用户访问电商网站的首选。**对于电商企业来说，一般会通过各种不同的渠道对自己的APP进行市场推广，而这些渠道的统计数据（比如，不同网站上广告链接的点击量、APP下载量）就成了市场营销的重要商业指标**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先我们考察分渠道的市场推广统计。在src/main/scala下创建`AppMarketingByChannel.scala`文件。由于没有现成的数据，所以我们需要**自定义一个测试源**来生成用户行为的事件流。

## 自定义测试数据源
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;定义一个源数据的样例类`MarketingUserBehavior`，再定义一个`SourceFunction`，用于产生用户行为源数据，命名为`SimulatedEventSource`：

```scala
// 定义一个输入数据的样例类  保存电商用户行为的样例类
  case class MarketingUserBehavior(userId: String, behavior: String, channel: String, timestamp: Long)

  // 定义一个输出结果的样例类   保存 市场用户点击次数
  case class MarketingViewCount(windowStart: String, windowEnd: String, channel: String, behavior: String, count: Long)

  // 自定义数据源
  class SimulateEventSource extends RichParallelSourceFunction[MarketingUserBehavior] {

    // 定义是否运行的标识符
    var running: Boolean = true
    // 定义渠道的集合
    val channelSet: Seq[String] = Seq("AppStore", "XiaomiStore", "HuaweiStore", "weibo", "wechat", "tieba")
    // 定义用户行为的集合
    val behaviorTypes: Seq[String] = Seq("BROWSE", "CLICK", "PURCHASE", "UNINSTALL")
    // 定义随机数发生器
    val rand: Random.type = Random

    // 重写 run 方法
    override def run(ctx: SourceFunction.SourceContext[MarketingUserBehavior]): Unit = {

      // 获取到 Long类型的最大值
      val maxElements: Long = Long.MaxValue
      // 设置初始值
      var count: Long = 0L

      // 随机生成所有数据
      while (running && count < maxElements) {

        // 生成一个随机数
        val id: String = UUID.randomUUID().toString
        // 获取随机行为
        val behaviorType: String = behaviorTypes(rand.nextInt(behaviorTypes.size))
        // 获取随机渠道
        val channel: String = channelSet(rand.nextInt(channelSet.size))
        // 获取到当前的系统时间
        val ts: Long = System.currentTimeMillis()
        // 输出生成的用户行为的事件流
        ctx.collect(MarketingUserBehavior(id, behaviorType, channel, ts))
        // count + 1
        count += 1
        // 设置休眠的时间
        TimeUnit.MICROSECONDS.sleep(10L)

      }
    }

    override def cancel(): Unit = running = false
  }
```

## 分渠道统计
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;另外定义一个窗口处理的输出结果样例类 `MarketingViewCount`，并自定义 `ProcessWindowFunction`进行处理，完整代码如下：

```scala
import java.sql.Timestamp
import java.util.UUID
import java.util.concurrent.TimeUnit

import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.source.{RichParallelSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.function.ProcessWindowFunction
import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.util.Collector

import scala.util.Random

/*
 * @Author: Alice菌
 * @Date: 2020/12/7 17:32
 * @Description: 
    电商用户行为数据分析：  市场营销商业指标统计分析
    APP市场推广统计   - - > 分渠道统计
 */
object AppMarketingByChannel {

  // 定义一个输入数据的样例类  保存电商用户行为的样例类
  case class MarketingUserBehavior(userId: String, behavior: String, channel: String, timestamp: Long)

  // 定义一个输出结果的样例类   保存 市场用户点击次数
  case class MarketingViewCount(windowStart: String, windowEnd: String, channel: String, behavior: String, count: Long)

  // 自定义数据源
  class SimulateEventSource extends RichParallelSourceFunction[MarketingUserBehavior] {

    // 定义是否运行的标识符
    var running: Boolean = true
    // 定义渠道的集合
    val channelSet: Seq[String] = Seq("AppStore", "XiaomiStore", "HuaweiStore", "weibo", "wechat", "tieba")
    // 定义用户行为的集合
    val behaviorTypes: Seq[String] = Seq("BROWSE", "CLICK", "PURCHASE", "UNINSTALL")
    // 定义随机数发生器
    val rand: Random.type = Random

    // 重写 run 方法
    override def run(ctx: SourceFunction.SourceContext[MarketingUserBehavior]): Unit = {

      // 获取到 Long类型的最大值
      val maxElements: Long = Long.MaxValue
      // 设置初始值
      var count: Long = 0L

      // 随机生成所有数据
      while (running && count < maxElements) {

        // 生成一个随机数
        val id: String = UUID.randomUUID().toString
        // 获取随机行为
        val behaviorType: String = behaviorTypes(rand.nextInt(behaviorTypes.size))
        // 获取随机渠道
        val channel: String = channelSet(rand.nextInt(channelSet.size))
        // 获取到当前的系统时间
        val ts: Long = System.currentTimeMillis()
        // 输出生成的用户行为的事件流
        ctx.collect(MarketingUserBehavior(id, behaviorType, channel, ts))
        // count + 1
        count += 1
        // 设置休眠的时间
        TimeUnit.MICROSECONDS.sleep(10L)

      }
    }

    override def cancel(): Unit = running = false
  }

  def main(args: Array[String]): Unit = {

    // 创建流处理的环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    env.addSource(new SimulateEventSource()) //  添加数据源
      .assignAscendingTimestamps(_.timestamp) // 设置水印
      .filter(_.behavior != "UNINSTALL") // 过滤掉 卸载 的数据
      .map(data => {
        ((data.channel, data.behavior), 1L)
      })
      .keyBy(_._1) //以渠道和行为作为key分组
      .timeWindow(Time.hours(1), Time.seconds(1)) // 设置滑动窗口,窗口大小为1h,滑动距离为1s
      .process(new MarketingCountByChannel) // 调用自定义处理方法
      .print() // 输出结果

    // 执行程序
    env.execute("app marketing by channel job")

  }

  // 自定义处理函数
  class MarketingCountByChannel() extends ProcessWindowFunction[((String, String), Long), MarketingViewCount, (String, String), TimeWindow] {

    override def process(key: (String, String), context: Context, elements: Iterable[((String, String), Long)], out: Collector[MarketingViewCount]): Unit = {

      // 根据 context 对象分别获取到 Long 类型的 窗口的开始和结束时间
      //context.window.getStart是长整形   所以new 一个 变成String类型
      val startTs: String = new Timestamp(context.window.getStart).toString
      val endTs: String = new Timestamp(context.window.getEnd).toString

      // 获取到 渠道
      val channel: String = key._1
      // 获取到 行为
      val behaviorType: String = key._2
      // 获取到 次数
      val count: Int = elements.size

      // 输出结果
      out.collect(MarketingViewCount(startTs, endTs, channel, behaviorType, count))
    }
  }
}
```

### 运行效果

![部分运行结果](https://img-blog.csdnimg.cn/20201212004419847.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 不分渠道（总量）统计
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同样我们还可以考察不分渠道的市场推广统计，这样得到的就是所有渠道推广的**总量**。在src/main/scala下创建`AppMarketingStatistics.scala`文件，代码如下：

```scala
import java.sql.Timestamp
import java.util.UUID
import java.util.concurrent.TimeUnit

import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.source.{RichParallelSourceFunction, SourceFunction}
import org.apache.flink.streaming.api.scala.function.ProcessWindowFunction
import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.streaming.api.windowing.windows.TimeWindow
import org.apache.flink.util.Collector

import scala.util.Random
/*
 * @Author: Alice菌
 * @Date: 2020/12/10 22:45
 * @Description: 
    电商用户行为数据分析：  市场营销商业指标统计分析
    APP市场推广统计   - - > 不分渠道（总量）统计
 */
object AppMarketingStatistics {

  // 定义一个输入数据的样例类  保存电商用户行为的样例类
  case class MarketingUserBehavior(userId: String, behavior: String, channel: String, timestamp: Long)

  // 定义一个输出结果的样例类   保存 市场用户点击次数
  case class MarketingViewCount(windowStart: String, windowEnd: String, count: Long)

  def main(args: Array[String]): Unit = {

    // 定义流处理环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    env.addSource(new SimulateEventSource)  // 添加数据源
      .assignAscendingTimestamps(_.timestamp)
      .filter(_.behavior != "UNINSTALL")
      .map(data => {
        ("key",1L)   // 因为这里我们不分渠道,所以我们就将key值固定,将所有数据放入到同一个组
      })
      .keyBy(_._1)
      .timeWindow(Time.hours(1),Time.seconds(1))   // 设置滑动窗口,窗口大小为1h,滑动距离为1s
      .process(new MarketingCountByChannel) // 调用自定义处理方法
      .print() // 输出结果

    // 执行程序
    env.execute("app marketing by channel job")

  }


  // 自定义数据源
  class SimulateEventSource extends RichParallelSourceFunction[MarketingUserBehavior] {

    // 定义是否运行的标识符
    var running: Boolean = true
    // 定义渠道的集合
    val channelSet: Seq[String] = Seq("AppStore", "XiaomiStore", "HuaweiStore", "weibo", "wechat", "tieba")
    // 定义用户行为的集合
    val behaviorTypes: Seq[String] = Seq("BROWSE", "CLICK", "PURCHASE", "UNINSTALL")
    // 定义随机数发生器
    val rand: Random.type = Random

    // 重写 run 方法
    override def run(ctx: SourceFunction.SourceContext[MarketingUserBehavior]): Unit = {

      // 获取到 Long类型的最大值
      val maxElements: Long = Long.MaxValue
      // 设置初始值
      var count: Long = 0L

      // 随机生成所有数据
      while (running && count < maxElements) {
        // 生成一个随机数
        val id: String = UUID.randomUUID().toString
        // 获取随机行为
        val behaviorType: String = behaviorTypes(rand.nextInt(behaviorTypes.size))
        // 获取随机渠道
        val channel: String = channelSet(rand.nextInt(channelSet.size))
        // 获取到当前的系统时间
        val ts: Long = System.currentTimeMillis()
        // 输出生成的用户行为的事件流
        ctx.collect(MarketingUserBehavior(id, behaviorType, channel, ts))
        // count + 1
        count += 1
        // 设置休眠的时间
        TimeUnit.MICROSECONDS.sleep(10L)

      }
    }

    override def cancel(): Unit = running = false
  }


  // 自定义处理函数
  class MarketingCountByChannel() extends ProcessWindowFunction[(String, Long), MarketingViewCount, String, TimeWindow] {

    override def process(key: String, context: Context, elements: Iterable[(String, Long)], out: Collector[MarketingViewCount]): Unit = {

      // 根据 context 对象分别获取到 Long 类型的 窗口的开始和结束时间
      //context.window.getStart是长整形   所以new 一个 变成String类型
      val startTs: String = new Timestamp(context.window.getStart).toString
      val endTs: String = new Timestamp(context.window.getEnd).toString

      // 获取到 次数
      val count: Int = elements.size

      // 输出结果
      out.collect(MarketingViewCount(startTs, endTs,count))

    }
  }
}
```

### 运行效果

![部分运行结果](https://img-blog.csdnimg.cn/20201212004940384.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
***
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期关于介绍flink 电商用户行为数据分析之**APP市场推广统计**的文章就到这里，主要为大家介绍了在自定义数据源的基础上，如何分渠道和不分渠道计算APP市场推广的数据 。考虑到部分小伙伴对于中间的部分代码有疑问，所以我每行都写上了注释，因此详细的过程笔者就不在这里详细赘述了。看了注释仍有疑惑的小伙伴们欢迎添加我的个人微信询问，互相学习，共同进步！**你知道的越多，你不知道的也越多**，我是Alice，我们下一期见！

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**受益的朋友记得三连支持小菌！**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
>**文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…期待您的关注!**

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201116102452301.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)