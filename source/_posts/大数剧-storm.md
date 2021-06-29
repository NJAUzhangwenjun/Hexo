---
title: 大数剧-storm
date: 2020-12-09 23:11:05
tags: 大数剧-storm
category: 大数剧
summary: 
 - 大数剧
 - storm
top: false
cover: true
author: 张文军
---
<center>更多内容请关注：https://wjhub.gitee.io</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# 大数剧-storm

## 一、Storm是什么

Storm是一个免费并开源的分布式实时计算系统。利用Storm可以很容易做到可靠地处理无限的数据流，像Hadoop批量处理大数据一样，Storm可以实时处理数据。Storm简单，可以使用任何编程语言。
 　　在Storm之前，进行实时处理是非常痛苦的事情: 需要维护一堆消息队列和消费者，他们构成了非常复杂的图结构。消费者进程从队列里取消息，处理完成后，去更新数据库，或者给其他队列发新消息。
 　　这样进行实时处理是非常痛苦的。我们主要的时间都花在关注往哪里发消息，从哪里接收消息，消息如何序列化，真正的业务逻辑只占了源代码的一小部分。一个应用程序的逻辑运行在很多worker上，但这些worker需要各自单独部署，还需要部署消息队列。最大问题是系统很脆弱，而且不是容错的：需要自己保证消息队列和worker进程工作正常。
 　　Storm完整地解决了这些问题。它是为分布式场景而生的，抽象了消息传递，会自动地在集群机器上并发地处理流式计算，让你专注于实时处理的业务逻辑。

## 二、Storm的特点

Storm有如下特点：

1. 编程简单：开发人员只需要关注应用逻辑，而且跟Hadoop类似，Storm提供的编程原语也很简单
2. 高性能，低延迟：可以应用于广告搜索引擎这种要求对广告主的操作进行实时响应的场景。
3. 分布式：可以轻松应对数据量大，单机搞不定的场景
4. 可扩展： 随着业务发展，数据量和计算量越来越大，系统可水平扩展
5. 容错：单个节点挂了不影响应用
6. 消息不丢失：保证消息处理
    不过Storm不是一个完整的解决方案。使用Storm时你需要关注以下几点：
7. 如果使用的是自己的消息队列，需要加入消息队列做数据的来源和产出的代码
8. 需要考虑如何做故障处理：如何记录消息队列处理的进度，应对Storm重启，挂掉的场景
9. 需要考虑如何做消息的回退：如果某些消息处理一直失败怎么办？

## 三、Storm的应用

跟Hadoop不一样，Storm是没有包括任何存储概念的计算系统。这就让Storm可以用在多种不同的场景下：非传统场景下数据动态到达或者数据存储在数据库这样的存储系统里（或数据是被实时操控其他设备的控制器(如交易系统)所消费）
 　　Storm有很多应用：实时分析，在线机器学习(online machine learning)，连续计算(continuous computation)，分布式远程过程调用(RPC)、ETL等。Storm处理速度很快：每个节点每秒钟可以处理超过百万的数据组。它是可扩展(scalable)，容错(fault-tolerant)，保证你的数据会被处理，并且很容易搭建和操作。

## 四、Storm模型

Storm实现了一个数据流(data flow)的模型，在这个模型中数据持续不断地流经一个由很多转换实体构成的网络。一个数据流的抽象叫做流(stream)，流是无限的元组(Tuple)序列。元组就像一个可以表示标准数据类型（例如int，float和byte数组）和用户自定义类型（需要额外序列化代码的）的数据结构。每个流由一个唯一的ID来标示的，这个ID可以用来构建拓扑中各个组件的数据源。
 　　如下图所示，其中的水龙头代表了数据流的来源，一旦水龙头打开，数据就会源源不断地流经Bolt而被处理。图中有三个流，用不同的颜色来表示，每个数据流中流动的是元组(Tuple)，它承载了具体的数据。元组通过流经不同的转换实体而被处理。
 　　Storm对数据输入的来源和输出数据的去向没有做任何限制。像Hadoop，是需要把数据放到自己的文件系统HDFS里的。在Storm里，可以使用任意来源的数据输入和任意的数据输出，只要你实现对应的代码来获取/写入这些数据就可以。典型场景下，输入/输出数据来是基于类似Kafka或者ActiveMQ这样的消息队列，但是数据库，文件系统或者web服务也都是可以的。
   
   ![Storm模型](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209112528751.png)


## 五、概念

>Storm并行度
> 
> 在storm中,任务只是在集群中运行的一个Spout的bolt实例。理解并行性是如何工作的,我们必须首先解释一个Storm集群拓扑参与
> 执行的四个主要组件: 
> - Nodes(机器):这些只是配置为Storm集群参与执行拓扑的部分的机器。Storm集群包含一个或多个节点来完成工作
> - Workers(JVM):这些是在一个节点上运行独立的JVM进程。每个节点配置一个或更多运行的worker。一个拓扑可以请求一个或更多的worker分配给它。 
> - Executors(线程):这些是worker运行在JVM进程一个Java线程。多个任务可以分配给一个Executor。除非显式重写,Storm将分配一个任务给一个Executor。
> - Tasks(Spout/Bolt实例):任务是Spout和bolt的实例，在executor线程中运行nextTuple()和executre()方法。

### 1. 拓扑(Topologies)

![Topologies](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209113719019.png)

一个Storm拓扑打包了一个实时处理程序的逻辑。一个Storm拓扑跟一个MapReduce的任务(job)是类似的。主要区别是MapReduce任务最终会结束，而拓扑会一直运行（当然直到你杀死它)。一个拓扑是一个通过流分组(stream grouping)把Spout和Bolt连接到一起的拓扑结构。图的每条边代表一个Bolt订阅了其他Spout或者Bolt的输出流。一个拓扑就是一个复杂的多阶段的流计算。

### 2. 元组(Tuple)

![Tuple](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209113944789.png)

元组是Storm提供的一个轻量级的数据格式，可以用来包装你需要实际处理的数据。元组是一次消息传递的基本单元。一个元组是一个命名的值列表，其中的每个值都可以是任意类型的。元组是动态地进行类型转化的--字段的类型不需要事先声明。在Storm中编程时，就是在操作和转换由元组组成的流。通常，元组包含整数，字节，字符串，浮点数，布尔值和字节数组等类型。要想在元组中使用自定义类型，就需要实现自己的序列化方式。

### 3. 流(Streams)

![Streams](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209113554633.png)

流是Storm中的核心抽象。一个流由无限的元组序列组成，这些元组会被分布式并行地创建和处理。通过流中元组包含的字段名称来定义这个流。每个流声明时都被赋予了一个ID。只有一个流的Spout和Bolt非常常见，所以提供了不需要指定ID来声明一个流的函数(Spout和Bolt都需要声明输出的流)。这种情况下，流的ID是默认的“default”。

### 4. Spouts

![Spouts](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209114042295.png)

Spout(喷嘴，这个名字很形象)是Storm中流的来源。通常Spout从外部数据源，如消息队列中读取元组数据并吐到拓扑里。Spout可以是可靠的(reliable)或者不可靠(unreliable)的。可靠的Spout能够在一个元组被Storm处理失败时重新进行处理，而非可靠的Spout只是吐数据到拓扑里，不关心处理成功还是失败了。
 　　Spout可以一次给多个流吐数据。此时需要通过OutputFieldsDeclarer的declareStream函数来声明多个流并在调用SpoutOutputCollector提供的emit方法时指定元组吐给哪个流。
 　　Spout中最主要的函数是nextTuple，Storm框架会不断调用它去做元组的轮询。如果没有新的元组过来，就直接返回，否则把新元组吐到拓扑里。nextTuple必须是非阻塞的，因为Storm在同一个线程里执行Spout的函数。
 　　Spout中另外两个主要的函数是ack和fail。当Storm检测到一个从Spout吐出的元组在拓扑中成功处理完时调用ack,没有成功处理完时调用fail。只有可靠型的Spout会调用ack和fail函数。更多细节可以查看Storm Java文档和我的另外一篇文章：Storm如何保证可靠的消息处理

### 5. Bolts

![Bolts](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209114118445.png)

在拓扑中所有的计算逻辑都是在Bolt中实现的。一个Bolt可以处理任意数量的输入流，产生任意数量新的输出流。Bolt可以做函数处理，过滤，流的合并，聚合，存储到数据库等操作。Bolt就是流水线上的一个处理单元，把数据的计算处理过程合理的拆分到多个Bolt、合理设置Bolt的task数量，能够提高Bolt的处理能力，提升流水线的并发度。
 　　Bolt可以给多个流吐出元组数据。此时需要使用OutputFieldsDeclarer的declareStream方法来声明多个流并在使OutputColletor的emit方法时指定给哪个流吐数据。
 　　当你声明了一个Bolt的输入流，也就订阅了另外一个组件的某个特定的输出流。如果希望订阅另一个组件的所有流，需要单独挨个订阅。InputDeclarer有语法糖来订阅ID为默认值的流。例如declarer.shuffleGrouping("redBolt")订阅了redBolt组件上的默认流，跟declarer.shuffleGrouping("redBolt", DEFAULT_STREAM_ID)是相同的。
 　　在Bolt中最主要的函数是execute函数，它使用一个新的元组当作输入。Bolt使用OutputCollector对象来吐出新的元组。Bolts必须为处理的每个元组调用OutputCollector的ack方法以便于Storm知道元组什么时候被各个Bolt处理完了（最终就可以确认Spout吐出的某个元组处理完了）。通常处理一个输入的元组时，会基于这个元组吐出零个或者多个元组，然后确认(ack)输入的元组处理完了，Storm提供了IBasicBolt接口来自动完成确认。
 　　必须注意OutputCollector不是线程安全的，所以所有的吐数据(emit)、确认(ack)、通知失败(fail)必须发生在同一个线程里。更多信息可以参照问题定位。

### 6. 任务(Tasks)


每个Spout和Bolt会以多个任务(Task)的形式在集群上运行。每个任务对应一个执行线程，流分组定义了如何从一组任务(同一个Bolt)发送元组到另外一组任务(另外一个Bolt)上。可以在调用TopologyBuilder的setSpout和setBolt函数时设置每个Spout和Bolt的并发数。

### 7. 组件(Component)

组件(component)是对Bolt和Spout的统称

### 8. 流分组(Stream Grouping)

![Stream Grouping](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209114159139.png)

Storm中的Stream Groupings用于告知Topology如何在两个组件间（如Spout和Bolt之间，或者不同的Bolt之间）进行Tuple的传送。每一个Spout和Bolt都可以有多个分布式任务，一个任务在什么时候、以什么方式发送Tuple就是由Stream Groupings来决定的

　　在Storm中有七个内置的流分组策略，你也可以通过实现CustomStreamGrouping接口来自定义一个流分组策略:

1. 洗牌分组(Shuffle grouping): 随机分配元组到Bolt的某个任务上，这样保证同一个Bolt的每个任务都能够得到相同数量的元组。

2. 字段分组(Fields grouping): 按照指定的分组字段来进行流的分组。例如，流是用字段“user-id"来分组的，那有着相同“user-id"的元组就会分到同一个任务里，但是有不同“user-id"的元组就会分到不同的任务里。这是一种非常重要的分组方式，通过这种流分组方式，我们就可以做到让Storm产出的消息在这个"user-id"级别是严格有序的，这对一些对时序敏感的应用(例如，计费系统)是非常重要的。

3. Partial Key grouping: 跟字段分组一样，流也是用指定的分组字段进行分组的，但是在多个下游Bolt之间是有负载均衡的，这样当输入数据有倾斜时可以更好的利用资源。这篇论文很好的解释了这是如何工作的，有哪些优势。

4. All grouping: 流会复制给Bolt的所有任务。小心使用这种分组方式。在拓扑中，如果希望某类元祖发送到所有的下游消费者，就可以使用这种All grouping的流分组策略。

5. Global grouping: 整个流会分配给Bolt的一个任务。具体一点，会分配给有最小ID的任务。

6. 不分组(None grouping): 说明不关心流是如何分组的。目前，None grouping等价于洗牌分组。

7. Direct grouping：一种特殊的分组。对于这样分组的流，元组的生产者决定消费者的哪个任务会接收处理这个元组。只能在声明做直连的流(direct streams)上声明Direct groupings分组方式。只能通过使用emitDirect系列函数来吐元组给直连流。一个Bolt可以通过提供的TopologyContext来获得消费者的任务ID，也可以通过OutputCollector对象的emit函数(会返回元组被发送到的任务的ID)来跟踪消费者的任务ID。在ack的实现中，Spout有两个直连输入流，ack和ackFail，使用了这种直连分组的方式。

8. Local or shuffle grouping：如果目标Bolt在同一个worker进程里有一个或多个任务，元组就会通过洗牌的方式分配到这些同一个进程内的任务里。否则，就跟普通的洗牌分组一样。这种方式的好处是可以提高拓扑的处理效率，因为worker内部通信就是进程内部通信了，相比拓扑间的进程间通信要高效的多。worker进程间通信是通过使用Netty来进行网络通信的。

### 9. 可靠性(Reliability)

Storm保证了拓扑中Spout产生的每个元组都会被处理。Storm是通过跟踪每个Spout所产生的所有元组构成的树形结构并得知这棵树何时被完整地处
理来达到可靠性。每个拓扑对这些树形结构都有一个关联的“消息超时”。如果在这个超时时间里Storm检测到Spout产生的一个元组没有被成功处理完，
那Sput的这个元组就处理失败了，后续会重新处理一遍。
为了发挥Storm的可靠性，需要你在创建一个元组树中的一条边时告诉Storm，也需要在处理完每个元组之后告诉Storm。这些都是通过Bolt吐
元组数据用的OutputCollector对象来完成的。标记是在emit函数里完成，完成一个元组后需要使用ack函数来告诉Storm。
这些都在“保证消息处理”一文中会有更详细的介绍。

### 10. Workers(工作进程)

拓扑以一个或多个Worker进程的方式运行。每个Worker进程是一个物理的Java虚拟机，执行拓扑的一部分任务。例如，如果拓扑的并发设置成了300，
分配了50个Worker，那么每个Worker执行6个任务(作为Worker内部的线程）。Storm会尽量把所有的任务均分到所有的Worker上。


## 六、Storm中用到的技术

**ZeroMQ** 提供了可扩展环境下的传输层高效消息通信，一开始Storm的内部通信使用的是ZeroMQ，后来作者想把Storm移交给Apache开源基金
会来管理，而ZeroMQ的许可证书跟Apache基金会的政策有冲突。在Storm中，Netty比ZeroMQ更加高效，而且提供了worker间通信时的验证机制，
所以在Storm0.9中，就改用了Netty。
 　　**Clojure** Storm系统的实现语言。Clojure是由Rich Hicky作为一种通用语言发明的，它衍生自Lisp语言，简化了多线程编程。
 　　**Apache ZooKeeper **Zookeeper是一个实现高可靠的分布式协作的开源项目。Storm使用Zookeeper来协调集群中的多个节点。
   
## 七、Storm架构思想

### 1. Storm和Hadoop架构组件功能对应关系

Storm运行任务的方式与Hadoop类似：Hadoop运行的是MapReduce作业，而Storm运行的是“Topology”
但两者的任务大不相同，主要的不同是：MapReduce作业最终会完成计算并结束运行，而Topology将持续处理消息（直到人为终止）

|   | **Hadoop** | **Storm**  |
| ----------- | ---------- | ---------- |
| 应用名称    | Job        | Topology   |
| 系统角色    | JobTracker | Nimbus     |
|    | TaskTracker | Supervisor |
| 组件接口    | Map/Reduce | Spout/Bolt |


### 2. Storm集群模式

Storm集群采用“Master—Worker”的节点方式：

- Master节点运行名为“Nimbus”的后台程序（类似Hadoop中的“JobTracker”），负责在集群范围内分发代码、为Worker分配任务和监测故障

- Worker节点运行名为“Supervisor”的后台程序，负责监听分配给它所在机器的工作，即根据Nimbus分配的任务来决定启动或停止Worker进程，
  一个Worker节点上同时运行若干个Worker进程

- Storm使用Zookeeper来作为分布式协调组件，负责Nimbus和多个Supervisor之间的所有协调工作。借助于Zookeeper，若Nimbus进程或Supervisor进程
  意外终止，重启时也能读取、恢复之前的状态并继续工作，使得Storm极其稳定

![Storm集群架构示意图](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201209115647656.png)

(1)worker（JVM进程）:每个worker进程都属于一个特定的Topology，每个Supervisor节点的worker可以有多个，每个worker对Topology中的每个组件（Spout或 Bolt
）运行一个或者多个executor线程来提供task的运行服务

(2)executor（线程）：executor是产生于worker进程内部的线程，会执行同一个组件的一个或者多个task。
(3)task:实际的数据处理由task完成，在Topology的生命周期中，每个组件的task数目是不会发生变化的，而executor的数目却不一定。executor数目小于等
于task的数目，默认情况下，二者是相等的

![Worker、Executor和Task的关系](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210120057547.png)

### 3. Storm的工作流程

1. 所有Topology任务的提交必须在Storm客户端节点上进行，提交后，由Nimbus节点分配给其他Supervisor节点进行处理
2. Nimbus节点首先将提交的Topology进行分片，分成一个个Task，分配给相应的Supervisor，并将Task和Supervisor相关的信息提交到Zookeeper集群上
3. Supervisor会去Zookeeper集群上认领自己的Task，通知自己的Worker进程进行Task的处理

> 说明：在提交了一个Topology之后，Storm就会创建Spout/Bolt实例并进行序列化。之后，将序列化的组件发送给所有的任务所在的机器(即Supervisor节点)
> ，在每一个任务上反序列化组件

#### Storm工作流程示意图

![Storm工作流程示意图](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210120537898.png)

## 八、代码实战:单词计数(WordCount)

### 1. 代码：

pom.xml

```xml
    <dependencies>
        <dependency>
            <groupId>org.apache.storm</groupId>
            <artifactId>storm-core</artifactId>
            <version>0.9.1-incubating</version>
        </dependency>
    </dependencies>

```

1). SentenceSpout： 数据生成并以元组（tuple）形式发送

```java

package cn.zhanghub.bigdata.storm;

import backtype.storm.spout.SpoutOutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichSpout;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Values;
import backtype.storm.utils.Utils;

import java.util.Map;

/**
 * SentenceSpout 组件，将语句作为数据源以元组(tuple)发出
 * BaseRichSpout类是一个方便的类，它实现了ISpout和IComponent接口并提供默认的不需要的方法。
 * 使用这个类，我们需只专注于我们所需要的方法。
 * @author user
 */
public class SentenceSpout extends BaseRichSpout {
	private SpoutOutputCollector collector;
	private String[] sentences = {"my dog has fleas", "i like cold beverages", "the dog ate my homework",
			"don't have a cow man", "i don't think i like fleas"};
	private int index = 0;

	/**
	 * declareOutputFields()方法是Storm IComponent接口中定义的接口，所有的Storm组件(包括Spout和bolt)必须实现该方法,
	 * 它用于告诉Storm流组件将会发出的每个流的元组将包含的字段。在这种情况下,我们定义的spout将发射一个包含一个字段(“sentence”)
	 * 的单一(默认)的元组流。
	 * @param declarer
	 */
	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		/*定义spout将发射一个包含一个字段(“sentence”)的单一(默认)的元组流*/
		declarer.declare(new Fields("sentence"));
	}

	/**
	 *open()方法中是ISpout中定义的接口，在Spout组件初始化时被调用。
	 * open()方法接受三个参数:
	 * 	一个包含Storm配置的Map,
	 * 	一个TopologyContext对象,它提供了关于组件在一个拓扑中的上下文信息,
	 * 	一个SpoutOutputCollector对象提供发射元组的方法。
	 * 	在这个例子中,我们不需要执行初始化,因此,open()实现简单将SpoutOutputCollector对象的引用存储在一个实例变量中 。
	 * @param config 配置
	 * @param context topology 上下文
	 * @param collector 提供发射元组的方法。
	 */
	@Override
	public void open(Map config, TopologyContext context, SpoutOutputCollector collector) {
		this.collector = collector;
	}

	/**
	 * nextTuple方法是所有 spout实现的核心所在，Storm通过主动调用这个方法向输出的
	 * collctor 发射tuple。这个例子中，我们发射当前索引对应的语句，并且递增索引指向下一
	 * 个语句。
	 */
	@Override
	public void nextTuple() {
		/**
		 * 将元组发射出去
		 */
		this.collector.emit(new Values(sentences[index]));
		index++;
		if (index >= sentences.length) {
			index = 0;
		}
		Utils.sleep(1);
	}
}


```

2). SplitSentenceBolt ：实现单词分割 blot

```java
package cn.zhanghub.bigdata.storm;

import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Tuple;
import backtype.storm.tuple.Values;

import java.util.Map;

/**
 *
 * 实现单词分割 blot
 *
 * BaseRichBolt类是IComponent和IBolt接口的一个简便实现。继承这个类，就不用去
 * 实现本例不关心的方法，将注意力放在实现我们需要的功能上。
 * @author user
 */
public class SplitSentenceBolt extends BaseRichBolt {
	private OutputCollector collector;

	/**
	 * prepare(方法在IBolt中定义，类同与ISpout接口中定义的open()方法。这个方法
	 * 在bolt初始化时调用，可以用来准备bolt 用到的资源，如数据库连接。和SentenceSpout
	 * 类一样，SplitSentenceBolt 类在初始化时没有额外操作，因此prepare(方法仅仅保存
	 * OutputCollector对象的引用。
	 * @param config 配置信息
	 * @param context 上下文
	 * @param collector 提供发射元组的方法
	 */
	@Override
	public void prepare(Map config, TopologyContext context, OutputCollector collector) {
		this.collector = collector;
	}

	/**
	 * SplitSentenceBolt 类的核心功能在execute()方法中实现，这个方法是IBolt接口定义
	 * 的。每当从订阅的数据流中接收一个tuple,都会调用这个方法。本例中，execute( 方法按
	 * 照字符串读取“sentence"字段的值，然后将其拆分为单词，每个单词向后面的输出流发
	 * 射一个tuple。
	 * @param tuple
	 */
	@Override
	public void execute(Tuple tuple) {
		String sentence = tuple.getStringByField("sentence");
		String[] words = sentence.split(" ");
		for (String word : words) {
			this.collector.emit(new Values(word));
		}
	}

	/**
	 * 	在declareOutputFields()方法中，SplitSentenceBolt 声明了一个输出流，每个tuple包
	 * 	含一个字段“word”。
	 * @param declarer 元组申明
	 */
	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		declarer.declare(new Fields("word"));
	}
}


```
3). WordCountBolt : 实现单词计数 blot

```java
package cn.zhanghub.bigdata.storm;

import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Tuple;
import backtype.storm.tuple.Values;

import java.util.HashMap;
import java.util.Map;

/**
 * 实现单词计数 blot
 * @author user
 */
public class WordCountBolt extends BaseRichBolt {
	/**
	 * 必须要在prepare()方法中对不可序列化的对象进行实例化。
	 * （虽然HashMap是可序列化的）
	 */
    private OutputCollector collector;
    private HashMap<String, Long> counts = null;
    @Override
	public void prepare(Map config, TopologyContext
            context, OutputCollector collector) {
        this.collector = collector;
        this.counts = new HashMap<String, Long>();
    }

    @Override
	public void execute(Tuple tuple) {
        String word = tuple.getStringByField("word");
        Long count = this.counts.get(word);
        if(count == null){
            count = 0L;
        }
        count++;
        this.counts.put(word, count);
        this.collector.emit(new Values(word, count));
    }

    @Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word", "count"));
    }
}


```

4). WordCountTopology : 实现单词计数 topology 拓扑

```java
package cn.zhanghub.bigdata.storm;

import backtype.storm.Config;
import backtype.storm.LocalCluster;
import backtype.storm.topology.TopologyBuilder;
import backtype.storm.tuple.Fields;
import backtype.storm.utils.Utils;

/**
 *实现单词计数 topology 拓扑
 * @author user
 */
public class WordCountTopology {

	/**
	 * 首先定义字符串常量,这将作ID为我们唯一标识Storm组件
	 */
	private static final String SENTENCE_SPOUT_ID = "sentence-spout";
	private static final String SPLIT_BOLT_ID = "split-bolt";
	private static final String COUNT_BOLT_ID = "count-bolt";
	private static final String REPORT_BOLT_ID = "report-bolt";
	private static final String TOPOLOGY_NAME = "word-count-topology";

	public static void main(String[] args) throws Exception {
		SentenceSpout spout = new SentenceSpout();
		SplitSentenceBolt splitBolt = new SplitSentenceBolt();
		WordCountBolt countBolt = new WordCountBolt();
		ReportBolt reportBolt = new ReportBolt();

		/* TopologyBuilder类提供了流式接口风格的API来定义topology组件之间的数据流。 */
		TopologyBuilder builder = new TopologyBuilder();

		/* 注册一个sentence spout并且赋值给其唯- - -的ID: */
		builder.setSpout(SENTENCE_SPOUT_ID, spout);

		/**
		 * 类TopologyBuilder的setBolt()方法会注册一个bolt,并且返回BoltDeclarer的实
		 * 例，可以定义bolt的数据源。这个例子中，我们将SentenceSpout的唯一ID赋值给
		 * shuffleGrouping() 方法确立了这种订阅关系。shuffleGrouping()方法告诉Storm,要将
		 * 类SentenceSpout发射的tuple随机均匀的分发给SplitSentenceBolt的实例。
		 */
		/* 注册一个SplitSentenceBolt, 这个bolt订阅SentenceSpout发射出来的数据流: */
		// SentenceSpout --> SplitSentenceBolt
		builder.setBolt(SPLIT_BOLT_ID, splitBolt).shuffleGrouping(SENTENCE_SPOUT_ID);

		/**
		 * fieldsGrouping:需要将含有特定数据的tuple路由到特殊的bolt实例中。
		 * 在此我们使用类BoltDeclarer的fieldsGrouping()方法来保证所有“ word"字段值相同的tuple会
		 * 被路由到同一个WordCountBolt实例中。
		 */
		/* 确立类SplitSentenceBolt和类theWordCountBolt之间的连接关系: */
		// SplitSentenceBolt --> WordCountBolt
		builder.setBolt(COUNT_BOLT_ID, countBolt).fieldsGrouping(SPLIT_BOLT_ID, new Fields("word"));

		/**
		 *我们希望WordCountBolt发射的所有tuple 路由到唯一 的ReportBolt任务中。
		 * globalGrouping()方法提供了这种用法:
		 */
		// WordCountBolt --> ReportBolt
		builder.setBolt(REPORT_BOLT_ID, reportBolt).globalGrouping(COUNT_BOLT_ID);
		/**
		 * Storm的Config类仅仅是HashMap的之列,它定义了一系列配置Storm拓扑的运行时行为具体常量和方便的方法。
		 * 当提交一个拓扑时,Storm将合并其预定义的默认配置值和Congif实例的内容传递给submitTopology()方法,
		 * 并将结果分别传递给拓扑的spout的open()和bolt的prepare()方法。在这个意义上,配置参数的配置对象表示一组全局拓扑中的所有组件。
		 */
		Config config = new Config();
		LocalCluster cluster = new LocalCluster();
		cluster.submitTopology(TOPOLOGY_NAME, config, builder.createTopology());
		Utils.sleep(10000);
		cluster.killTopology(TOPOLOGY_NAME);
		cluster.shutdown();
	}
}


```

### 2. WordCountTopology并行性

到目前为止,单词计数的例子中,没有显式地使用任何Storm的并行api;相反,是允许Storm使用其默认设置。
在大多数情况下,除非覆盖,Storm将默认使用最大并行性设置。
在改变拓扑结构的并行设置之前,让拓扑在默认设置下，是如何将执行的：假设我们有一台机器(节点),指定
一个worker的拓扑,并允许Storm每一个任务一个executor执行,执行的拓扑，
将会如下:


![WordCountTopology默认设置拓扑](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210122551355.png)

正如看到的,并行性只有线程级别(多个Executor)。每个任务运行在一个JVM的一个单独的线程内。那怎样才能利用硬件更有
效地提高并行性?通过增加worker和executors的数量来运行拓扑。

### 3. 给topology增加worker


增加额外的worker是增加topology计算能力的简单方法。为此Storm提供了API和修改配置项两种修改方法。无论采取哪种方法，
spout 和bolt组件都不需要做变更，可以直接复用。

在单词计数topology前面的版本中，引入了Config对象在发布时传递参数给submitTopology()方法，但是没有做更多配置操作。
为了增加分配给一个topology的worker数量，
只需要简单的调用一下Config对象的setNumWorkers()方法:


```java

Config config = new Config();
config.setNumWorkers(2);

```
这样就给topology分配了两个worker而不是默认的一个。从而增加了topology 的计算资源，也更有效的利用了计算资源。还可以调整topology中的executor
个数以及每个executor分配的task数量。

![WordCountTopology增加2个worker的拓扑](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210124139468.png)

### 4. 配置executor和task

Storm给topology中定义的每个组件建立一个task,默认的情况下，每个task分配一个executor。Storm的并发机制API对此提供
了控制方法，允许设定每个task对应的executor个数和每
个executor可执行的task的个数。
在定义数据流分组时，可以设置给一个组件指派的executor的数量。
为了说明这个功能，修改topology的定义代码，设置SentenceSpout 并发为两个task,每个task指派各自的executor线程。

```java

builder.setSpout(SENTENCE_SPOUT_ID, spout, 2);

```
下一步，给语句分割bolt SplitSentenceBolt设置4个task和2个executor。每个executor线程指派2个task来执行(4/2=2)。
还将配置单词计数bolt 运行四个task,每个task由一一个
executor线程执行:

```java
builder.setBolt(SPLIT_BOLT_ID, splitBolt, 2).setNumTasks(4)
    .shuffleGrouping(SENTENCE_SPOUT_ID);
builder.setBolt(COUNT_BOLT_ID, countBolt, 4)
    .fieldsGrouping(SPLIT_BOLT_ID, newFields("word"));

```
经过上述设置后的topology如下：

![topology](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210125355522.png)

#### 5. 理解数据流分组

数据流分组定义了一个数据流中的tuple如何分发给topology中不同bolt的task。举
例说明，在并发版本的单词计数topology中，SplitSentenceBolt 类指派了四个task数据
流分组决定了指定的一个tuple 会分发到哪个task上。

数据流分组除了前面提到的七个分组方式之外，还可以通过实现CustomStreamGrouping (自定义分组)
接口来自定义分组方式:

```java

package cn.zhanghub.bigdata.storm;

import backtype.storm.generated.GlobalStreamId;
import backtype.storm.task.WorkerTopologyContext;

import java.io.Serializable;
import java.util.List;

/**
 *
 * 自定义分组 CustomStreamGrouping
 * @author user
 */
public interface CustomStreamGrouping extends Serializable {
	/**
	 * 在运行时调用，用来初始化分组信息，分组的具体实现会使用这些信息决定如何向接收task分发tuple。
	 * @param context WorkerTopologyContext对象提供了topology 的上下文信息
	 * @param stream GlobalStreamId提供了待分组数据流的属性
	 * @param targetTasks 分组所有待选task的标识符列表 ,通常，会将targetTasks的引用存在变量里作为chooseTasks（）的参数。
	 */
    void prepare(WorkerTopologyContext context, GlobalStreamId stream, List<Integer> targetTasks);

	/**
	 * chooseTasks
	 * @param taskId tuple的组件的id
	 * @param values tuple的组件的值
	 * @return tuple发送目标task的标识符列表
	 */
    List<Integer> chooseTasks(int taskId, List<Object> values);
}


```

## 九、Storm 数据处理保障机制

Storm提供了一种API能够保证spout发送出来的每个tuple都能够执行完整的处理过程。在上面的例子中，不担心执行失败的情况。可以看
到在一个topology中一个spout的数据流会被分割生成任意多的数据流，取决于下游bolt的行为。

如果发生了执行失败会怎样?
举个例子，考虑一个负责将数据持久化到数据库的bolt。 怎样处理数据库更新失败的情况?

### 1. spout的可靠性

在Storm中，可靠的消息处理机制是从spout开始的。一个提供了可靠的处理机制的spout需要记录它发射出去的tuple,当下游bolt处理tuple
或者子tuple失败时spout能够重新发射。
子tuple可以理解为bolt处理spout发射的原始tuple后，作为结果发射出去的tuple。
另外一个视角来看，可以将spout发射的数据流看作一个tuple树的主干：

![tuple树](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/%E5%A4%A7%E6%95%B0%E5%89%A7-storm/20201210012741503.png)

在图中，实线部分表示从spout发射的原始主干tuple,虚线部分表示的子tuple都是源自于原始tuple。这样产生的图形叫做tuple树。在有保障数据的处理过
程中，bolt每收到一个tuple,都需要向上游确认应答(ack)者报错。对主干tuple中的一个tuple,如果tuple树上的每个bolt进行了确认应答，spout会调用
ack方法来标明这条消息已经完全处理了。如果树中任何一个bolt 处理tuple报错，或者处理超时，spout 会调用fail方法。

Storm的ISpout接口定义了三个可靠性相关的API: nextTuple, ack 和fail。

```java

public interface ISpout extends Serializable {
    void open(Map conf, TopologyContext context, SpoutOutputCollector collector);

    void close();
    void nextTuple();
    void ack(Object msgId);
    void fail(Object msgId);
}

```
前面说过，Storm 通过调用Spout的nextTuple（)发送一个tuple。为实现可靠的消息处理，首先要给每个发出的tuple带上唯一的ID,并且将ID作为参数传递给SpoutOutputCollector
的emit)方法:

```java

collector.emit(new Values("value1", "value2") ,msgId);

```

给tuple指定ID告诉Storm系统，无论执行成功还是失败，spout 都要接收tuple树上所有节点返回的通知。如果处理成功，spout的ack()方法将会对编号是ID的消息应答确
认，如果执行失败或者超时，会调用fail() 方法。

### 2. bolt的可靠性

bolt要实现可靠的消息处理机制包含两个步骤:

1. 当发射衍生的tuple时，需要锚定读入的tuple
2. 当处理消息成功或者失败时分别确认应答或者报错

锚定一个tuple的意思是，建立读入tuple和衍生出的tuple之间的对应关系，这样下游的bolt就可以通过应答确认、报错或超时来加人到tuple树结构中。
可以通过调用OutputCollector中emit()的一个重载函数锚定一个或者一组tuple:

```java

collector.emit(tuple, new Values(word));

```

这里，我们将读人的tuple和发射的新tuple锚定起来，下游的bolt就需要对输出的tuple进行确认应答或者报错。

另外一个emit() 方法会发射非锚定的tuple:

```java

collector.emit(new Values(word));

```

未锚定的元组不参与一个流的可靠性保证。如果一个非锚点元组下游失败,它不会导致原始根元组的重发。

当处理完成或者发送了新tuple之后，可靠数据流中的bolt需要应答读入的tuple:

```java

this.collector.ack(tuple);

```

如果处理失败，这样的话spout必须发射tuple,bolt就要明确地对处理失败的tuple报错:

this.collector.fail(tuple)

如果因为超时的原因，或者显式调用OutputColector.fail（）方法，spout 都会重新发送
原始tuple。

### 3. 可靠的单词计数

为了进一步说明可控性，需要增强SentenceSpout类，支持可靠的tuple发射方式。
需要记录所有发送的tuple,并且分配一个唯一的ID。我们使用 HashMap<UUID, Values>来存储已发送待确认的tuple。每当发送一个新的tuple,分配一个唯一的标识符并且存储
在我们的hashmap中。当收到一个确认消息，从待确认列表中删除该tuple。如果收到报错，从新发送tuple:

```java

package cn.zhanghub.bigdata.storm;

import backtype.storm.spout.SpoutOutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichSpout;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Values;
import backtype.storm.utils.Utils;

import java.util.Map;
import java.util.UUID;
import java.util.concurrent.ConcurrentHashMap;

/**
 * 可靠的单词计数 SentenceSpout
 * @author user
 */
public class SentenceSpout2 extends BaseRichSpout {

	private ConcurrentHashMap<UUID, Values> pending;

	private SpoutOutputCollector collector;

	private String[] sentences = {"my dog has fleas", "i like cold beverages", "the dog ate my homework",
			"don't have a cow man", "i don't think i like fleas"};
	private int index = 0;

	@Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
		declarer.declare(new Fields("sentence"));
	}

	@Override
	public void open(Map config, TopologyContext context, SpoutOutputCollector collector) {
		this.collector = collector;
		this.pending = new ConcurrentHashMap<UUID, Values>();
	}

	@Override
	public void nextTuple() {
		Values values = new Values(sentences[index]);
		UUID msgId = UUID.randomUUID();
		this.pending.put(msgId, values);
		this.collector.emit(values, msgId);
		index++;
		if (index >= sentences.length) {
			index = 0;
		}
		Utils.sleep(1);
	}

	@Override
	public void ack(Object msgId) {
		this.pending.remove(msgId);
	}

	@Override
	public void fail(Object msgId) {
		this.collector.emit(this.pending.get(msgId), msgId);
	}
}


```

为支持有保障的处理，需要修改bolt,将输出的tuple和输人的tuple锚定，并且应答确认输人的tuple:


```java

package cn.zhanghub.bigdata.storm;

import backtype.storm.task.OutputCollector;
import backtype.storm.task.TopologyContext;
import backtype.storm.topology.OutputFieldsDeclarer;
import backtype.storm.topology.base.BaseRichBolt;
import backtype.storm.tuple.Fields;
import backtype.storm.tuple.Tuple;
import backtype.storm.tuple.Values;

import java.util.Map;

/**
 * 可靠的单词计数 ReliableSplitSentenceBolt
 * @author user
 */
public class ReliableSplitSentenceBolt extends BaseRichBolt {
    private OutputCollector collector;
    @Override
	public void prepare(Map config, TopologyContext
            context, OutputCollector collector) {
        this.collector = collector;
    }

    @Override
	public void execute(Tuple tuple) {
        String sentence = tuple.getStringByField("sentence");
        String[] words = sentence.split(" ");
        for(String word : words){
            this.collector.emit(tuple, new Values(word));
        }
        this.collector.ack(tuple);
    }
    @Override
	public void declareOutputFields(OutputFieldsDeclarer declarer) {
        declarer.declare(new Fields("word"));
    }
}


```









