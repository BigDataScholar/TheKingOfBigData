&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本篇是flink 的「电商用户行为数据分析」的第 8 篇文章，为大家带来的是**市场营销商业指标统计分析**之**订单支付实时监控**的内容！通过本期内容，我们可以实现通过使用**CEP**和**Process Function**来实现`订单支付实时监控`的功能，还能学会通过**connect** 和 **join**来实现`flink双流join`的功能，可谓干货满满！受益的朋友记得三连支持一下 ~

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213001243897.jpg?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
***

## 订单支付实时监控
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在电商网站中，**订单的支付作为直接与营销收入挂钩的一环，在业务流程中非常重要**。对于订单而言，**为了正确控制业务流程，也为了增加用户的支付意愿，网站一般会设置一个支付失效时间，超过一段时间不支付的订单就会被取消**。另外，**对于订单的支付，我们还应保证用户支付的正确性，这可以通过第三方支付平台的交易数据来做一个实时对账**。在接下来的内容中，我们将实现这两个需求。


## 模块创建和数据准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同样地，在UserBehaviorAnalysis下新建一个 maven module作为子项目，命名为`OrderTimeoutDetect`。在这个子模块中，我们同样将会用到 **flink** 的 **CEP** 库来实现事件流的模式匹配，所以需要在pom文件中引入CEP的相关依赖：

```xml
<dependency>
        <groupId>org.apache.flink</groupId>
<artifactId>flink-cep-scala_${scala.binary.version}</artifactId>
<version>${flink.version}</version>
</dependency>
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同样，在src/main/目录下，将默认源文件目录java改名为scala。


## 代码实现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在电商平台中，**最终创造收入和利润的是用户下单购买的环节**；更具体一点，是用户真正完成支付动作的时候。**用户下单的行为可以表明用户对商品的需求，但在现实中，并不是每次下单都会被用户立刻支付**。**当拖延一段时间后，用户支付的意愿会降低。所以为了让用户更有紧迫感从而提高支付转化率，同时也为了防范订单支付环节的安全风险，电商网站往往会对订单状态进行监控，设置一个失效时间（比如15分钟），如果下单后一段时间仍未支付，订单就会被取消**。


### 使用CEP实现

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们首先还是利用**CEP**库来实现这个功能。我们先将事件流按照订单号**orderId**分流，然后定义这样的一个**事件模式**：在15分钟内，事件“create”与“pay”非严格紧邻：

```scala
    // 1、 定义一个匹配事件序列的模式
    val orderPayPattern = Pattern
      .begin[OrderEvent]("create").where(_.eventType == "create")   // 首先是订单的 create 事件
      .followedBy("pay").where(_.eventType == "pay")  // 后面来的是订单的 pay 事件
      .within(Time.minutes(15))    // 间隔 15 分钟
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这样调用.select方法时，就可以同时获取**到匹配出的事件**和**超时未匹配的事件**了。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在src/main/scala下继续创建`OrderTimeout.scala`文件，新建一个单例对象。定义样例类**OrderEvent**，这是输入的订单事件流；另外还有**OrderResult**，这是输出显示的订单状态结果。订单数据也本应该从UserBehavior日志里提取，由于`UserBehavior.csv`中没有做相关埋点，我们从另一个文件`OrderLog.csv`中读取登录数据。

![OrderLog.csv部分数据](https://img-blog.csdnimg.cn/20201213010959481.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
**完整代码如下：**

```scala
import java.util

import org.apache.flink.cep.scala.pattern.Pattern
import org.apache.flink.cep.scala.{CEP, PatternStream}
import org.apache.flink.cep.{PatternSelectFunction, PatternTimeoutFunction}
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.scala.{StreamExecutionEnvironment, _}
import org.apache.flink.streaming.api.windowing.time.Time
/*
 * @Author: Alice菌
 * @Date: 2020/12/13 15:46
 * @Description:

 */
object OrderTimeoutWithOutCep {
   
  // 定义输入的订单事件样例类
  case class OrderEvent(orderId:Long,eventType:String,eventTime:Long)
  // 定义输出的订单检测结果样例类
  case class OrderResult(orderId:Long,resultMsg:String)

  def main(args: Array[String]): Unit = {

    // 定义流处理环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置程序并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    // 从文件中读取数据，并转换成样例类
    val orderEventStream: DataStream[OrderEvent] = env.readTextFile("YOUR_PATH\\OrderLog.csv")
      .map(data => {
        // 样例数据： 34729,pay,sd76f87d6,1558430844
        val dataArray: Array[String] = data.split(",")
        OrderEvent(dataArray(0).toLong, dataArray(1), dataArray(3).toLong)
      })  //  处理数据
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[OrderEvent](Time.seconds(3)) {
        override def extractTimestamp(element: OrderEvent): Long = element.eventTime * 1000L
      })  //  设置时间戳

    // 1、 定义一个匹配事件序列的模式
    val orderPayPattern = Pattern
      .begin[OrderEvent]("create").where(_.eventType == "create")   // 首先是订单的 create 事件
      .followedBy("pay").where(_.eventType == "pay")  // 后面来的是订单的 pay 事件
      .within(Time.minutes(15))    // 间隔 15 分钟

    // 2、 将 pattern 应用在按照 orderId分组的数据流上
    val patterStream: PatternStream[OrderEvent] = CEP.pattern(orderEventStream.keyBy(_.orderId), orderPayPattern)

    // 3、定义一个侧输出流标签，用来标明超时事件的侧输出流
    val orderTimeOutputTag: OutputTag[OrderResult] = new OutputTag[OrderResult]("order time out")

    // 4、调用select方法，提取匹配事件和超时事件，分别进行处理转换输出
    val result: DataStream[OrderResult] = patterStream
      .select(orderTimeOutputTag, new OrderTimeOutSelect(), new OrderPaySelect())

    // 5、打印输出
    result.print("payed")
    result.getSideOutput(orderTimeOutputTag).print("timeout")

    // 执行程序
    env.execute("order timeout detect job")

  }

  // 自定义超时处理函数
  class OrderTimeOutSelect() extends PatternTimeoutFunction[OrderEvent,OrderResult]{
    override def timeout(pattern: util.Map[String, util.List[OrderEvent]], timeoutTimestamp: Long): OrderResult = {
      val timeOutOrderId: Long = pattern.get("create").iterator().next().orderId
      OrderResult(timeOutOrderId,"timeout at" + timeoutTimestamp)
    }
  }

  // 自定义匹配处理函数
  class OrderPaySelect() extends PatternSelectFunction[OrderEvent,OrderResult]{
    override def select(pattern: util.Map[String, util.List[OrderEvent]]): OrderResult = {

      val payedOrderId: Long = pattern.get("pay").get(0).orderId
      OrderResult(payedOrderId,"pay successfully")
    }
  }
}

```

**运行结果：**

![部分结果展示](https://img-blog.csdnimg.cn/20201213011256706.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
###  使用Process Function实现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们同样可以利用Process Function，自定义实现检测订单超时的功能。为了简化问题，我们只考虑超时报警的情形，在pay事件超时未发生的情况下，输出超时报警信息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个简单的思路是，可以在订单的 create 事件到来后注册定时器，15分钟后触发；然后再用一个布尔类型的Value状态来作为标识位，表明pay事件是否发生过。如果pay事件已经发生，状态被置为true，那么就不再需要做什么操作；而如果pay事件一直没来，状态一直为false，到定时器触发时，就应该输出超时报警信息。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;具体代码实现如下：

```scala
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.KeyedProcessFunction
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.util.Collector


/*
 * @Author: Alice菌
 * @Date: 2020/12/23 19:35
 * @Description: 
    
 */
object OrderTimeout {

  // 定义输入的订单事件样例类
  case class OrderEvent(orderId: Long, eventType: String, eventTime: Long)

  // 定义输出的订单检测结果样例类
  case class OrderResult(orderId: Long, resultMsg: String)

  def main(args: Array[String]): Unit = {
    // 定义流处理环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置程序并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)
    // 读取输入的订单数据流
    val orderEventStream: DataStream[OrderEvent] = env.readTextFile("YOUR_PATH\\OrderLog.csv")
      .map(data => {
        // 示例数据： 34729,pay,sd76f87d6,1558430844
        val dataArray: Array[String] = data.split(",")
        OrderEvent(dataArray(0).toLong, dataArray(1), dataArray(3).toLong)
      })
      // 设置水印
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[OrderEvent](Time.seconds(3)) {
        override def extractTimestamp(element: OrderEvent): Long = element.eventTime * 1000L
      })

    // 自定义 Process Function,做精细化的流程控制
    val orderResultStream: DataStream[OrderResult] = orderEventStream
      .keyBy(_.orderId)
      .process(new OrderPayMatchDetect())

    // 打印输出
    orderResultStream.print("payed")
    orderResultStream.getSideOutput(new OutputTag[OrderResult]("timeout")).print("timeout")

    // 执行程序
    env.execute("order timeout without cep job")
  }

  class OrderPayMatchDetect() extends KeyedProcessFunction[Long,OrderEvent,OrderResult]{
    // 定义状态，用来保存是否来过 create 和 pay 事件的标识位，以及定时器的时间戳
    lazy val isPayState:ValueState[Boolean] = getRuntimeContext.getState(new ValueStateDescriptor[Boolean]("is-payed", classOf[Boolean]))
    lazy val isCreateState:ValueState[Boolean] = getRuntimeContext.getState(new ValueStateDescriptor[Boolean]("is-created", classOf[Boolean]))

    // 定义一个状态，保存每次定时器的时间戳
    lazy val timerTsState:ValueState[Long] = getRuntimeContext.getState(new ValueStateDescriptor[Long]("timer-ts", classOf[Long]))

    // 定义一个侧输出流
    val orderTimeOutputTag = new OutputTag[OrderResult]("timeout")

    override def processElement(value: OrderEvent, ctx: KeyedProcessFunction[Long, OrderEvent, OrderResult]#Context, out: Collector[OrderResult]): Unit = {

      // 取出当前的状态
      val isPayed: Boolean = isPayState.value()
      val isCreated: Boolean = isCreateState.value()
      val timeTs: Long = timerTsState.value()

      // 判断当前事件的类型，分成不同的情况讨论：
      // 情况1： 来的是 create，要继续判断之前是否有 pay 来过
      if (value.eventType == "create"){
        // 情况 1.1 ： 如果已经pay过，匹配成功，输出到主流，清空状态
        if (isPayed){
          out.collect(OrderResult(value.orderId,"payed successfully"))
          // 清除状态
          isPayState.clear()
          timerTsState.clear()
          // 删除定时器
          ctx.timerService().deleteEventTimeTimer(timeTs)
        }
          // 情况 1.2：如果没有pay过，那么就注册一个15分钟后的定时器，开始等待
        else{
          val ts: Long = value.eventTime * 1000L + 15 * 60 *1000L
          // 设置一个15分钟的定时器
          ctx.timerService().registerEventTimeTimer(ts)

          timerTsState.update(ts)
          isCreateState.update(true)
        }
      }

      // 情况2：来的是pay，要继续判断是否来过 create
      else if (value.eventType == "pay"){
        // 情况2.1 ： 如果 create 已经来过，匹配成功，要继续判断间隔时间是否超过了15分钟
        if (isCreated){
          // 情况 2.1.1：如果没有超时，正常输出结果到主流
          if (value.eventTime * 1000L < timeTs){
            out.collect(OrderResult(value.orderId,"payed successfully"))
          }else{
            // 情况2.1.2： 如果已经超时，那么输出 timeout 报警到侧输出流
            ctx.output(orderTimeOutputTag,OrderResult(value.orderId,"payed but already timeout"))
          }
          // 无论哪种情况，都已经有了输出，清空状态
          isCreateState.clear()
          timerTsState.clear()
          ctx.timerService().deleteEventTimeTimer(timeTs)
        }
           // 情况2.2 ：如果 create 没来，需要等待乱序 create，注册一个当前pay时间戳的定时器
        else{
          val ts: Long = value.eventTime * 1000L
          // 设置定时器
          ctx.timerService().registerEventTimeTimer(ts)
          // 更新状态
          timerTsState.update(ts)
          isPayState.update(true)
        }
      }
    }

    override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, OrderEvent, OrderResult]#OnTimerContext, out: Collector[OrderResult]): Unit = {
      // 定时器触发，需要判断是哪种情况
      if (isPayState.value()){
        // 如果 pay 过,那么说明 create没来，可能出现了数据丢失异常的情况
        ctx.output(orderTimeOutputTag,OrderResult(ctx.getCurrentKey,"already payed but not found created log"))
      }else{
        // 如果 没有 pay过，那么说明真正 15 分钟 超时 [提交了订单，但是超过了15分钟仍未支付]
        ctx.output(orderTimeOutputTag,OrderResult(ctx.getCurrentKey,"order timeout"))
      }

      // 清空状态
      isPayState.clear()
      isCreateState.clear()
      timerTsState.clear()

    }
  }
}

```

**运行结果：**


![部分结果展示](https://img-blog.csdnimg.cn/20201213011955730.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 来自两条流的订单交易匹配
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于订单支付事件，用户支付完成其实并不算完，我们还得确认平台账户上是否到账了。而往往这会来自不同的日志信息，所以我们要同时读入两条流的数据来做合并处理。这里我们利用`connect`将两条流进行连接，然后用自定义的**CoProcessFunction**进行处理。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;具体代码如下：

```scala
import org.apache.flink.api.common.state.{ValueState, ValueStateDescriptor}
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.co.CoProcessFunction
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.util.Collector

/*
 * @Author: Alice菌
 * @Date: 2020/12/13 15:57
 * @Description: 
    来自两条流的订单交易匹配  （ connect 实现 ）
 */
object OrderPayTxMatch {

  // 输入输出的样例类
  case class ReceiptEvent(txId:String, payChannel:String, timestamp:Long)
  case class OrderEvent(orderId:Long, eventType:String, txId:String, eventTime:Long)

  def main(args: Array[String]): Unit = {
    // 创建流处理的环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置程序并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    // 从 OrderLog.csv 文件中读取数据 ，并转换成样例类
    val orderEventStream: KeyedStream[OrderEvent, String] = env.readTextFile("G:\\idea arc\\BIGDATA\\project\\src\\main\\resources\\OrderLog.csv")
      .map(data => {
        // 样例数据 ：  34731,pay,35jue34we,1558430849
        val dataArray: Array[String] = data.split(",")
        OrderEvent(dataArray(0).toLong,dataArray(1),dataArray(2),dataArray(3).toLong)
      })
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[OrderEvent](Time.seconds(3)) {
        override def extractTimestamp(element: OrderEvent): Long = element.eventTime * 1000L
      })       //  为数据流中的元素分配时间戳
      .filter(_.eventType != "") // 只过滤出pay事件
      .keyBy(_.txId)    // 根据 订单id 分组

    // 从 ReceiptLog.csv 文件中读取数据 ，并转换成样例类
    val receiptStream: KeyedStream[ReceiptEvent, String] = env.readTextFile("YOUR_PATH\\ReceiptLog.csv")
      .map(data => {
        // 样例数据： 3hu3k2432,alipay,1558430848
        val dataArray: Array[String] = data.split(",")
        ReceiptEvent(dataArray(0), dataArray(1), dataArray(2).toLong)
      })
      .assignAscendingTimestamps(_.timestamp * 1000L) // 设置水印
      .keyBy(_.txId)   // 根据 txId 进行分组

    // connect 连接两条流，匹配事件进行处理
    val resultStream: DataStream[(OrderEvent, ReceiptEvent)] = orderEventStream.connect(receiptStream)
      .process(new OrderPayTxDetect())

    // 定义侧输出流
    val unmatchedPays: OutputTag[OrderEvent] = new OutputTag[OrderEvent]("unmatched-pays")
    val unmatchedReceipts: OutputTag[ReceiptEvent] = new OutputTag[ReceiptEvent]("unmatched-receipts")

    // 打印输出
    resultStream.print("matched")
    resultStream.getSideOutput(unmatchedPays).print("unmatched-pays")
    resultStream.getSideOutput(unmatchedReceipts).print("unmatched-receipts")
    env.execute("order pay tx match job")

  }

  // 定义 CoProcessFunction，实现两条流数据的匹配检测
  class OrderPayTxDetect() extends CoProcessFunction[OrderEvent,ReceiptEvent,(OrderEvent,ReceiptEvent)]{

    // 定义两个 ValueState，保存当前交易对应的支付事件和到账事件
    lazy val payState: ValueState[OrderEvent] = getRuntimeContext.getState(new ValueStateDescriptor[OrderEvent]("pay", classOf[OrderEvent]))
    lazy val receiptState: ValueState[ReceiptEvent] = getRuntimeContext.getState(new ValueStateDescriptor[ReceiptEvent]("receipt", classOf[ReceiptEvent]))

    //定义侧输出流
    val unmatchedPays: OutputTag[OrderEvent] = new OutputTag[OrderEvent]("unmatched-pays")
    val unmatchedReceipts: OutputTag[ReceiptEvent] = new OutputTag[ReceiptEvent]("unmatched-receipts")

    override def processElement1(pay: OrderEvent, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#Context, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {
      // pay 来了，考察有没有对应的 receipt 来过
      val receipt: ReceiptEvent = receiptState.value()
      if (receipt != null){
        // 如果已经有 receipt，正常输出到主流
        out.collect((pay,receipt))
        receiptState.clear()
      }else{
        // 如果 receipt 还没来，那么把 pay 存入状态，注册一个定时器等待 5 秒
        payState.update(pay)
        ctx.timerService().registerEventTimeTimer(pay.eventTime * 1000L + 5000L)
      }
    }

    override def processElement2(receipt: ReceiptEvent, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#Context, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {
      //receipt来了，考察有没有对应的pay来过
      val pay: OrderEvent = payState.value()
      if (pay != null) {
        //如果已经有pay，那么正常匹配，输出到主流
        out.collect((pay, receipt))
        payState.clear()
      }else{
      // 如果 pay 还没来，那么把 receipt 存入状态，注册一个定时器等待 3 秒
        receiptState.update(receipt)
        ctx.timerService().registerEventTimeTimer(receipt.timestamp * 1000L + 3000L)
    }
  }

    // 定时触发， 有两种情况，所以要判断当前有没有pay和receipt
    override def onTimer(timestamp: Long, ctx: CoProcessFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]
      #OnTimerContext, out: Collector[(OrderEvent, ReceiptEvent)]): Unit = {

      //如果 pay 不为空，说明receipt没来，输出unmatchedPays
      if (payState.value() != null){
        ctx.output(unmatchedPays,payState.value())
      }

      if (receiptState.value() != null){
      ctx.output(unmatchedReceipts,receiptState.value())
      }

      // 清除状态
      payState.clear()
      receiptState.clear()
    }
    }
}
```

**运行结果：**
![部分结果展示](https://img-blog.csdnimg.cn/20201213012406942.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于flink的双流join通过`connect`的做法，肯定会有小伙伴觉得过程比较冗复杂，那还有没有其他的方法也能实现类似的效果呢？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201213013010483.png)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然是有的，下面就为大家展示另一种通过`intervalJoin`方法实现的方式：

```scala
import org.apache.flink.streaming.api.TimeCharacteristic
import org.apache.flink.streaming.api.functions.co.ProcessJoinFunction
import org.apache.flink.streaming.api.functions.timestamps.BoundedOutOfOrdernessTimestampExtractor
import org.apache.flink.streaming.api.scala._
import org.apache.flink.streaming.api.windowing.time.Time
import org.apache.flink.util.Collector

/*
 * @Author: Alice菌
 * @Date: 2020/12/12 20:23
 * @Description: 
     来自两条流的订单交易匹配  （  JOIN 实现 ）
 */
object OrderPayTxMatchWithJoin {

  // 输入输出的样例类
  case class ReceiptEvent(txId:String, payChannel:String, timestamp:Long)
  case class OrderEvent(orderId:Long, eventType:String, txId:String, eventTime:Long)

  def main(args: Array[String]): Unit = {
    // 创建流处理的环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置程序并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    // 从 OrderLog.csv 文件中读取数据 ，并转换成样例类
    val orderEventStream: KeyedStream[OrderEvent, String] = env.readTextFile("YOUR_PATH\\OrderLog.csv")
      .map(data => {
        // 样例数据 ：  34731,pay,35jue34we,1558430849
        val dataArray: Array[String] = data.split(",")
        OrderEvent(dataArray(0).toLong,dataArray(1),dataArray(2),dataArray(3).toLong)
      })
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[OrderEvent](Time.seconds(3)) {
        override def extractTimestamp(element: OrderEvent): Long = element.eventTime * 1000L
      })       //  为数据流中的元素分配时间戳
      .filter(_.eventType != "") // 只过滤出pay事件
      .keyBy(_.txId)    // 根据 订单id 分组

    // 从 ReceiptLog.csv 文件中读取数据 ，并转换成样例类
    val receiptStream: KeyedStream[ReceiptEvent, String] = env.readTextFile("YOUR_PATH\\ReceiptLog.csv")
      .map(data => {
        // 样例数据： 3hu3k2432,alipay,1558430848
        val dataArray: Array[String] = data.split(",")
        ReceiptEvent(dataArray(0), dataArray(1), dataArray(2).toLong)
      })
      .assignAscendingTimestamps(_.timestamp * 1000L) // 设置水印
      .keyBy(_.txId)   // 根据 txId 进行分组

    // 使用 join 连接两条流
    val resultStream: DataStream[(OrderEvent, ReceiptEvent)] = orderEventStream
      .intervalJoin(receiptStream)
      .between(Time.seconds(-5), Time.seconds(3))    
      .process(new OrderPayTxDetectWithJoin())

    resultStream.print()
    env.execute("order pay tx match with join job")

  }

  // 自定义 ProcessJoinFunction
  class OrderPayTxDetectWithJoin() extends ProcessJoinFunction[OrderEvent,ReceiptEvent,(OrderEvent,ReceiptEvent)]{
    override def processElement(left: OrderEvent, right: ReceiptEvent, ctx: ProcessJoinFunction[OrderEvent, ReceiptEvent, (OrderEvent, ReceiptEvent)]#Context, collector: Collector[(OrderEvent, ReceiptEvent)]): Unit = {

      collector.collect((left,right))
    }
  }
}
```

**虽然这种方法看似代码简单了不少，但是也存在局限性。只能匹配对应上的，不能输出没有匹配上的。**
![结果展示](https://img-blog.csdnimg.cn/20201213013602255.png)
***
## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;好了，当你看到这里的时候，意味着**电商用户行为数据分析**暂时完结了，不对，下一篇文章会为大家再总结一些**电商常见指标**的干货，敬请期待！！！**考虑到部分小伙伴对于中间的部分代码有疑问，所以我每行都写上了注释，因此详细的过程笔者就不在这里详细赘述了**。看了注释仍有疑惑的小伙伴们欢迎添加我的个人微信询问，**互相学习，共同进步**！你知道的越多，你不知道的也越多，我是Alice，我们下一期见！

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
> **文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…期待您的关注!**



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201116102452301.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)

