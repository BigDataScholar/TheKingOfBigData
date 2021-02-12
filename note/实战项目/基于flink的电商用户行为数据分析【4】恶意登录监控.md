## 前言
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在上一期内容中，菌哥已经为大家介绍了实时热门商品统计模块的功能开发的过程(👉[基于flink的电商用户行为数据分析【3】| 实时流量统计](https://alice.blog.csdn.net/article/details/110212749)）。本期文章，我们需要学习的是**恶意登录监控**模块功能的开发过程。

![在这里插入图片描述](https://img-blog.csdnimg.cn/202011281936211.png?,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

## 模块创建和数据准备
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;继续在`UserBehaviorAnalysis`下新建一个 maven module作为子项目，命名为`LoginFailDetect`。在这个子模块中，我们将会用到flink的`CEP`库来实现**事件流的模式匹配**，所以需要在pom文件中引入CEP的相关依赖：

```xml
<dependency>
        <groupId>org.apache.flink</groupId>
<artifactId>flink-cep-scala_${scala.binary.version}</artifactId>
<version>${flink.version}</version>
</dependency>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;同样，在src/main/目录下，将默认源文件目录java改名为scala。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
## 代码实现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;对于网站而言，**用户登录并不是频繁的业务操作**。如果一个用户短时间内频繁登录失败，就有可能是出现了程序的恶意攻击，比如密码暴力破解。因此我们考虑，应该对用户的登录失败动作进行统计，具体来说，**如果同一用户（可以是不同IP）在2秒之内连续两次登录失败，就认为存在恶意登录的风险，输出相关的信息进行报警提示**。这是电商网站、也是几乎所有网站**风控**的基本一环。


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以我们可以思考一下解决方案：
>- 基本需求
>-- 用户在短时间内频繁登录失败，有程序恶意攻击的可能
>-- 同一用户（可以是不同IP）在2秒内连续两次登录失败，需要报警
>
>- 解决思路
>-- 将用户的登录失败行为存入 ListState，设定定时器2秒后触发，查看 ListState 中有几次失败登录
>-- 更加准确的检测，可以使用 **CEP** 库实现事件流的**模式匹配**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既然现在思路清楚了，那我们就尝试将方案落地。

## 状态编程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于同样引入了时间，我们可以想到，最简单的方法其实与之前的热门统计类似，只需要**按照用户ID分流**，然后遇到登录失败的事件时将其保存在**ListState**中，然后**设置一个定时器**，2秒后触发。定时器触发时**检查状态中的登录失败事件个数**，如果大于等于2，那么就**输出报警信息**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在src/main/scala下创建`LoginFail.scala`文件，新建一个单例对象。定义样例类`LoginEvent`，这是输入的登录事件流。登录数据本应该从UserBehavior日志里提取，由于UserBehavior.csv中没有做相关埋点，我们从另一个文件`LoginLog.csv`中读取登录数据。

![LoginLog.csv](https://img-blog.csdnimg.cn/20201128154915810.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;代码如下：

```scala
object LoginFailOne {

  // 输入的登录事件样例类
  case class LoginEvent( userId:Long,ip:String,eventType:String,eventTime:Long)

  // 输出的报警信息样例类
  case class Warning( userId:Long,firstFailTime:Long,lastFailTime:Long,warningMsg:String)

  def main(args: Array[String]): Unit = {

    // 创建流环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    // 读取csv文件
    env.readTextFile("G:\\LoginLog.csv")
       .map(data => {
          // 将文件中的数据封装成样例类
          val dataArray: Array[String] = data.split(",")
          LoginEvent(dataArray(0).toLong, dataArray(1), dataArray(2), dataArray(3).toLong)
        })
        // 设置 WaterMark 水印
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[LoginEvent](Time.seconds(5)) {
        override def extractTimestamp(element: LoginEvent): Long = element.eventTime * 1000
      })
      // 以用户id为key，进行分组
      .keyBy(_.userId)
      // 计算出同一个用户2秒内连续登录失败超过2次的报警信息
      .process(new LoginWarning(2))
      .print()

    //  执行程序
    env.execute("login fail job")


  }

  // 自定义处理函数，保留上一次登录失败的事件，并可以注册定时器    [键的类型，输入元素的类型，输出元素的类型]
  class LoginWarning(maxFailTimes:Int) extends KeyedProcessFunction[Long,LoginEvent,Warning]{

    // 定义  保存登录失败事件的状态
    lazy val loginFailState: ListState[LoginEvent] = getRuntimeContext.getListState( new ListStateDescriptor[LoginEvent]("loginfail-state", classOf[LoginEvent]) )

    override def processElement(value: LoginEvent, ctx: KeyedProcessFunction[Long, LoginEvent, Warning]#Context, out: Collector[Warning]): Unit = {

      // 判断当前登录状态是否为 fail
      if (value.eventType == "fail"){
        // 判断存放失败事件的state是否有值，没有值则创建一个2秒后的定时器
        if (!loginFailState.get().iterator().hasNext){
          // 注册一个定时器，设置在 2秒 之后
          ctx.timerService().registerEventTimeTimer((value.eventTime + 2) * 1000L)
        }
        // 把新的失败事件添加到  state
        loginFailState.add(value)
      }else{
        // 如果登录成功，清空状态重新开始
        loginFailState.clear()
      }
    }

    override def onTimer(timestamp: Long, ctx: KeyedProcessFunction[Long, LoginEvent, Warning]#OnTimerContext, out: Collector[Warning]): Unit = {
      // 触发定时器的时候，根据状态的失败个数决定是否输出报警
      val allLoginFailEvents: ListBuffer[LoginEvent] = new ListBuffer[LoginEvent]()

      val iter: util.Iterator[LoginEvent] = loginFailState.get().iterator()

      // 遍历状态中的数据，将数据存放至 ListBuffer
      while ( iter.hasNext ){
        allLoginFailEvents += iter.next()
        }

      //判断登录失败事件个数，如果大于等于 maxFailTimes ，输出报警信息
      if (allLoginFailEvents.length >= maxFailTimes){
        out.collect(Warning(allLoginFailEvents.head.userId,
          allLoginFailEvents.head.eventTime,
          allLoginFailEvents.last.eventTime,
          "在2秒之内连续登录失败" + allLoginFailEvents.length + "次"))
      }

      // 清空状态
      loginFailState.clear()
    }
  }
}
```

程序运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128155457574.png)
我们可以到`LoginLog.csv`来验证结果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128155618696.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
貌似看到这里感觉我们的程序写的没有错，事实真的是这样的吗？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128155940179.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
那好，现在我改一个数据，把`1558430844`秒的登录状态改成`success`
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128160109499.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
然后重新运行一下程序，看看会发生什么？
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112816023894.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128160309863.png#pic_center)
我了个乖乖，什么情况，现在连结果都没了？

仔细看代码，才发现我们的思路是没错的，但是还是有 逻辑Bug !
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128160437197.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

不管一个用户之前连续登录失败多少次，只要中间成功一次，之前的记录就被清空了！


![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112816064128.png#pic_center)

## 状态编程的改进
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一节的代码实现中我们可以看到，**直接把每次登录失败的数据存起来、设置定时器一段时间后再读取，这种做法尽管简单，但和我们开始的需求还是略有差异的**。这种做法<font color='Tomato'>**只能隔2秒之后去判断一下这期间是否有多次失败登录，而不是在一次登录失败之后、再一次登录失败时就立刻报警**</font>。这个需求如果严格实现起来，相当于要判断任意紧邻的事件，是否符合某种模式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;于是我们可以想到，这个需求其实可以不用定时器触发，直接在状态中存取上一次登录失败的事件，每次都做判断和比对，就可以实现最初的需求。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上节的代码MatchFunction中删掉onTimer，processElement改为：

```scala
 // 自定义处理函数，保留上一次登录失败的事件    [键的类型，输入元素的类型，输出元素的类型]
  class LoginWarning(maxFailTimes:Int) extends KeyedProcessFunction[Long, LoginEvent, Warning] {

    // 定义  保存登录失败事件的状态
    lazy val loginFailState: ListState[LoginEvent] = getRuntimeContext.getListState(new ListStateDescriptor[LoginEvent]("loginfail-state", classOf[LoginEvent]))

    override def processElement(value: LoginEvent, ctx: KeyedProcessFunction[Long, LoginEvent, Warning]#Context, out: Collector[Warning]): Unit = {
      // 首先按照type做筛选，如果success直接清空，如果fail再做处理
      if(value.eventType == "fail"){
        // 先获取之前失败的事件
        val iter: util.Iterator[LoginEvent] = loginFailState.get().iterator()
        if (iter.hasNext){
          // 如果之前已经有失败的事件，就做判断，如果没有就把当前失败事件保存进state
          val firstFailEvent: LoginEvent = iter.next()
          // 判断两次失败事件间隔小于2秒，输出报警信息
          if (value.eventTime < firstFailEvent.eventTime + 2){
            out.collect(Warning( value.userId,firstFailEvent.eventTime,value.eventTime,"在2秒内连续两次登录失败。"))
          }

          // 更新最近一次的登录失败事件，保存在状态里
          loginFailState.clear()
          loginFailState.add(value)

        }else{
          // 如果是第一次登录失败，之前把当前记录 保存至 state
          loginFailState.add(value)
        }
      }else{
        // 当前登录状态 不为 fail，则直接清除状态
        loginFailState.clear()
      }
    }
  }
  }
```
这次我们基于上述已经修改过的`LoginLog.csv`文件，重新运行程序，发现此时是有结果的。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128162727206.png)
那现在的程序还会有Bug吗？
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128163330923.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当然还有会，例如我们去掉了定时器，如果运行过程中**数据处理乱序**，同一个用户每次登录失败的时间相差距离过大，可能很长一段时间都不会有该用户的报警信息。当然，还有其他的问题，我们放在下面一小节来说！


## CEP编程
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;上一节我们通过对状态编程的改进，去掉了定时器，在process function中做了更多的逻辑处理，实现了最初的需求。不过这种方法里有很多的条件判断，而我们目前仅仅实现的是检测“**连续2次登录失败**”，这是最简单的情形。如果需要检测更多次，**内部逻辑显然会变得非常复杂**。那有什么方式可以方便地实现呢？


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;很幸运，flink为我们提供了**CEP**`（Complex Event Processing，复杂事件处理）`库，用于**在流中筛选符合某种复杂模式的事件**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了担心小伙伴们对于 **CEP**  这个 “新事物”感到陌生，我们先来补一补`CEP`的内容！

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128165340132.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)
### 什么是复杂事件处理CEP

>- 复杂事件处理（Complex Event Processing，CEP）
>- Flink CEP是在 Flink 中实现的复杂事件处理（CEP）库
>- CEP 允许**在无休止的事件流中检测事件模式，让我们有机会掌握数据中重要的部分**
>- 一个或多个由简单事件构成的事件流通过一定的规则匹配，然后输出用户想得到的数据 —— 满足规则的复杂事件

### CEP特点

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果我们想从一堆图形中找到符合预期的结果，就可以根据某个规则去进行匹配，如下图所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128170043245.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

> - 目标：从有序的简单事件流中发现一些高阶特征
> - 输入：一个或多个由简单事件构成的事件流
> - 处理：识别简单事件之间的内在联系，多个符合一定规则的简单事件构成复杂事件
> - 输出：满足规则的复杂事件


### Pattern API

> - **处理事件的规则，被叫做“模式”（Pattern）**
> - **Flink CEP 提供了 Pattern API，用于对输入流数据进行复杂事件规则定义，用来提取符合规则的事件序列**
>![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128171319466.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
> - **个体模式（Individual Patterns）**
> -- 组成复杂规则的每一个单独的模式定义，就是“个体模式”
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128171015161.png)
>- **组合模式（Combining Patterns，也叫模式序列）**
-- 很多个体模式组合起来，就形成了整个的模式序列
-- 模式序列必须以一个“初始模式”开始：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128171230259.png)
>- **模式组（Groups of patterns）**
-- 将一个模式序列作为条件嵌套在个体模式里，成为一组模式
> 

### 个体模式（Individual Patterns）
- 个体模式可以包括“单例（singleton）模式”和“循环（looping）模式”
- 单例模式只接收一个事件，而循环模式可以接收多个

>  ★ 量词（Quantifier）
>  - 可以在一个个体模式后追加量词，也就是指定循环次数
>  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128171908448.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
### 个体模式的条件
> ★ 条件（Condition）
> -- 每个模式都需要指定触发条件，作为模式是否接受事件进入的判断依据
> -- CEP 中的个体模式主要通过调用 `.where() ` .`or()` 和 `.until() `来指定条件
> -- 按不同的调用方式，可以分成以下几类
> <br>
> ★简单条件（Simple Condition）
> -- 通过 `.where() `方法对事件中的字段进行判断筛选，决定是否接受该事件
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128172239106.png)
> ★组合条件（Combining Condition）
> -- 将简单条件进行合并；`.or()` 方法表示或逻辑相连，`where `的直接组合就是 AND
> ![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128172458733.png)
> ★ 终止条件（Stop Condition）
> -- 如果使用了 `oneOrMore` 或者 `oneOrMore.optional`，建议使用 `.until() `作为终止条件，以便清理状态
> <br>
> ★ 迭代条件（Iterative Condition）
> -- 能够对模式之前所有接收的事件进行处理
> -- 调用` .where( (value, ctx) => {...} )`，可以调用 `ctx.getEventsForPattern(“name”) ` 
>  提示： name可以是当前个体模式的名称，这个方法可以将之前匹配好的事件从状态中都拿出来，再做具体的判断，使用。一般在比较复杂的场景才会用到。


### 模式序列
> 
>- **不同的“近邻”模式**
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128173500269.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
>- **严格近邻**（Strict Contiguity）
-- 所有事件按照严格的顺序出现，中间没有任何不匹配的事件，由 **.next()** 指定
-- 例如对于模式`a  next b`，事件序列 `[a, c, b1, b2] `没有匹配
>- **宽松近邻**（ Relaxed Contiguity ）
>-- 允许中间出现不匹配的事件，由 **.followedBy()** 指定
-- 例如对于模式`a followedBy b`，事件序列` [a, c, b1, b2] 匹配为 {a, b1}`
>- **非确定性宽松近邻**（ Non-Deterministic Relaxed Contiguity ）
>-- 进一步放宽条件，之前已经匹配过的事件也可以再次使用，由  **.followedByAny()**  指定
>-- 例如对于模式`a followedByAny b`，事件序列 `[a, c, b1, b2]` 匹配为` {a, b1}，{a, b2}`
><br>
>- 除以上模式序列外，还可以定义“**不希望出现某种近邻关系**”：
>-- **.notNext()**  —— 不想让某个事件严格紧邻前一个事件发生
>-- **.notFollowedBy()** —— 不想让某个事件在两个事件之间发生
>- 需要注意：
>-- 所有模式序列必须以 **.begin()** 开始
>-- 模式序列不能以 **.notFollowedBy()** 结束
>-- **“not” 类型的模式不能被 optional 所修饰**
>-- 此外，还可以为模式指定**时间约束**，用来要求在多长时间内匹配有效
>![在这里插入图片描述](https://img-blog.csdnimg.cn/2020112817540698.png)
### 模式的检测
> - 指定要查找的模式序列后，就可以将其应用于输入流以检测潜在匹配
> - 调用 CEP.pattern()，给定输入流和模式，就能得到一个 PatternStream
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128175533723.png)

### 匹配事件的提取

>- 创建 `PatternStream` 之后，就可以应用` select `或者 `flatselect `方法，从检测到的事件序列中提取事件了
>- select() 方法需要输入一个 select function 作为参数，每个成功匹配的事件序列都会调用它
>- select() 以一个 `Map[String，Iterable [IN]] `来接收匹配到的事件序列，其中 key 就是每个模式的名称，而 value 就是所有接收到的事件的 **Iterable** 类型
>![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128180142848.png?,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)
### 超时事件的提取
> - 当一个模式通过 `within` 关键字定义了检测窗口时间时，部分事件序列可能因为超过窗口长度而被丢弃；为了能够处理这些超时的部分匹配，`select `和` flatSelect API` 调用允许指定超时处理程序。
>- 超时处理程序会接收到目前为止由模式匹配到的所有事件，由一个 `OutputTag `定义接收到的超时事件序列。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128180447522.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;接下来我们就需要基于**CEP**来完成这个模块的实现。

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128181336976.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;相关的pom文件我们已经在最开始的时候到导入了，现在在src/main/scala下继续创建`LoginFailWithCep.scala`文件，新建一个单例对象。样例类`LoginEvent`由于在LoginFail.scala已经定义，我们在同一个模块中就不需要再定义。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;具体代码如下：

```scala
object LoginFailWithCep {
  // 输入的登录事件样例类
  case class LoginEvent(userId: Long, ip: String, eventType: String, eventTime: Long)

  // 输出的报警信息样例类
  case class Warning(userId: Long, firstFailTime: Long, lastFailTime: Long, warningMsg: String)

  def main(args: Array[String]): Unit = {
    
    // 1、创建流环境
    val env: StreamExecutionEnvironment = StreamExecutionEnvironment.getExecutionEnvironment
    // 设置并行度
    env.setParallelism(1)
    // 设置时间特征为事件时间
    env.setStreamTimeCharacteristic(TimeCharacteristic.EventTime)

    // 构建数据
    val loginEventStream: KeyedStream[LoginEvent, Long] = env.readTextFile("G:\\idea arc\\BIGDATA\\project\\src\\main\\resources\\LoginLog.csv")
      .map(data => {
        // 将文件中的数据封装成样例类
        val dataArray: Array[String] = data.split(",")
        LoginEvent(dataArray(0).toLong, dataArray(1), dataArray(2), dataArray(3).toLong)
      })
      // 设置水印，防止数据乱序
      .assignTimestampsAndWatermarks(new BoundedOutOfOrdernessTimestampExtractor[LoginEvent](Time.seconds(3)) {
        override def extractTimestamp(element: LoginEvent): Long = element.eventTime * 1000
      })
      // 以用户id为key，进行分组
      .keyBy(_.userId)

    // 定义匹配的模式
    val loginFailPattern: Pattern[LoginEvent, LoginEvent] = Pattern.begin[LoginEvent]("begin")
      .where(_.eventType == "fail")
      .next("next")
      .where(_.eventType == "fail")
      .within(Time.seconds(2))    // 通过 within 关键字定义了检测窗口时间时间

    // 将 pattern 应用到 输入流 上，得到一个 pattern stream
    val patternStream: PatternStream[LoginEvent] = CEP.pattern(loginEventStream,loginFailPattern)

    // 用 select 方法检出 符合模式的事件序列
    val loginFailDataStream: DataStream[Warning] = patternStream.select(new LoginFailMatch())

    // 将匹配到的符合条件的事件打印出来
    loginFailDataStream.print("warning")
    
    // 执行程序
    env.execute("login fail with cep job")

  }

  // 自定义 pattern select function
  // 当检测到定义好的模式序列时就会调用，输出报警信息
  class LoginFailMatch() extends PatternSelectFunction[LoginEvent,Warning]{

    override def select(map: util.Map[String, util.List[LoginEvent]]): Warning = {
      // 从 map 中可以按照模式的名称提取对应的登录失败事件
      val firstFail: LoginEvent = map.get("begin").iterator().next()
      val secondFail: LoginEvent = map.get("next").iterator().next()
         
      Warning( firstFail.userId,firstFail.eventTime,secondFail.eventTime,"在2秒内连续2次登录失败。")
    }
  }
}
```

运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201128190916185.png)
可以发现也是符合我们预期的效果~

## 小结
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本期关于介绍**恶意登录监控**功能开发的文章肝了笔者近五个小时的时间，期望受益的朋友们能来发一键三连，多多支持一下作者。在上一期，我们介绍**实时流量统计**模块中，只介绍了基于**服务器log的热门页面浏览量统计**，下一期我们将介绍基于**埋点日志数据的网络流量统计**，分别介绍`网站总浏览量（PV）的统计`，`网站独立访客数（UV）的统计`还有使用到`使用布隆过滤器的UV统计`，感兴趣的朋友们可以关注加星标，第一时间获取每日的大数据干货哦~你知道的越多，你不知道的也越多，我是Alice，我们下一期见！

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **受益的朋友记得三连支持小菌！**

>**文章持续更新，可以微信搜一搜「 猿人菌 」第一时间阅读，思维导图，大数据书籍，大数据高频面试题，海量一线大厂面经…期待您的关注!**



![在这里插入图片描述](https://img-blog.csdnimg.cn/20201116102452301.png?,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDMxODgzMA==,size_16,color_FFFFFF,t_70#pic_center)



