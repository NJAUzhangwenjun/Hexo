---
title: JVM+GC解析
date: 2020-02-13 23:44:59
tags: JVM+GC解析
category: JVM
summary: JVM+GC解析
top: false
cover: true
author: 张文军
---

<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# JVM+GC解析


## 前提复习

- JVM内存结构

	- JVM体系概述
	- Java8以后的JVM

- GC的作用域
- 常见的垃圾回收算法

	- 引用计数

	  缺点：
	  每次对对象赋值时均要维护引用计数器，且计数器本身也有一定的消耗。
	  较难处理循环引用
	  
	   JVM的实现一般不采用这种方式

	- 复制

	  Java堆从GC的角度还可以细分为：新生代（Eden区、From Survivor区和To Survivor区）和老年呆。
	  
	  优点：没有产生内存碎片（因为整体复制）
	  缺点：相对浪费空间，耗时
	  
	  MinorGC的过程（复制->清空->互换）
	  
	  1：eden、SurvivorFrom复制到SurvivorTo，年龄+1
	  首先，当Eden区满的时候会触发第一次GC，把还活着的对象拷贝到SurvivorFrom区，当Eden区再次触发GC的时候会扫描Eden区和From区域，对这两个区域进行垃圾回收，经过这次回收后还存活的对象，则直接复制到To区域（如果有对象的年龄已经到达了老年的标准，则赋值到老年代区），同时把这些对象的年龄+1
	  
	  2.清空edeb、SurvivorFrom
	  然后，清空Eden和SurvivorFrom中的对象，也即复制之后有交换，谁空谁是to
	  
	  3.SurvivorTo和SurvivorFrom互换
	  最后，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区。部分对象会在From和To区域中复制来复制去，如此交换15次（由JVM参数MaxTenuringThreshold决定，这个参数默认是15），最终如果还是存活，就存入到老年代。

	- 标记清除

	  垃圾收集算法-标记清楚法（Mark-Sweep）：算法分成标记和清除两个阶段，先标记出要回收的对象，然后统一回收这些对象。
	  
	  优点：未对对象大面积的复制，节约了内存空间
	  缺点：产生内存碎片

	- 标记整理

	  标记-压缩（Mark-Compact）
	  
	  原理：
	  标记（Mark）：
	  		与标记-清除一样。
	     2.压缩（Compact）：
	  		再次扫描，并往一端滑动存活对象。
	  
	  优点：没有内存碎片，可以利用bump
	  缺点：需要移动对象的成本

## 节外生枝补充知识

## 题目1

- 1.JVM垃圾回收的时候如何确定垃圾？是否知道什么是GC Roots

	- 什么是垃圾

		- 内存中已经不再被使用到的空间就是垃圾

	- 要进行垃圾回收，如何判断一个对象是否可以被回收？

		- 引用计数法

		  Java中，引用和对象是有关联的。如果要操作对象则必须引用进行。
		  
		  因此，简单的办法是通过引用计数来判断一个对象是否可以回收。简单的说，给对象中添加一个引用计数，每当有一个引用失效时，计数器值减1.
		  
		  任何时刻计数器值为0的对象就是不可能再被利用的，那么这个对象就是可回收对象。
		  
		  那么为什么主流的Java虚拟机里面都没有选择这种算法呢？主要的原因是它很难解决对象之间相互循环引用的问题。

		- 枚举根节点做可达性分析（根搜索路径）

		  为了解决引用计数法的循环引用问题，Java使用了可达性分析的方法。
		  
		  所谓“GC roots”或者tracing GC的“根集合”就是一组必须活跃的引用。
		  
		  基本思路就是通过一系列名为“GC Roots”的对象作为起点，从这个被称为GC Roots的对象开始向下搜索，如果一个对象到GC Roots没有任何引用链相连时，则说明此对象不可用。即给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可达性的）对象就被判定为存活，没有被遍历到的就被判断为死亡。

			- case
			- Java中可以作为GC Roots的对象

				- 虚拟机栈（栈帧中的局部变量区，也叫做局部变量表）中引用的对象。
				- 方法区中的类静态属性引用的对象。
				- 方法区中常量引用的对象
				- 本地方法栈中JNI（Native方法）引用的对象。

- 2.你说你做过JVM调优和参数配置，请问如何查看JVM系统默认值

	- JVM的参数类型

		- 标配参数

			- -version
			- -help
			- java -showversion

		- x参数（了解）

			- -Xint：解释执行
			- -Xcomp：第一次使用就编译成本地代码
			- -Xmixed：混合模式

		- xx参数

			- Boolean类型

				- 公式

					- -XX：+或者-某个属性值
					- +表示开启  -表示关闭

				- Case

					- 是否打印GC收集细节

						- -XX：-PrintGCDetails
						- -XX:+PrintGCDetails

					- 是否使用串行垃圾回收器

						- -XX:-UseSerialGC
						- -XX:UseSerialGC

			- KV设值类型

				- 公式

					- -XX:属性key=属性值value

				- Case

					- -XX:MetaspaceSize=128m
					- -XX:MaxTenuringThreadhold=15

			- jinfo举例，如何查看当前运行程序的配置
			- 题外话（坑题）

				- 两个经典参数：-Xms和-Xmx
				- 这个你如何解释

					- -Xms：等价于-XX:InitialHeapSize
					- -Xmx：等价于-XX：MaxHeapSize

	- 盘点家底查看JVM默认值

		- -XX：+PrintFlagsInitial

			- 主要查看初始默认
			- 公式

				- java -XX：+PrintFlagsInitial -version
				- java -XX:+PrintFlagsInitial

			- Case

		- -XX:+PrintFlagsFinal

			- 主要查看修改更新
			- 公式

				- java -XX:+PrintFlagsFinal -version
				- java -XX:+PrintFlags -version

			- Case

		- PrintFlagsFinal举例，运行java命令的同时打印出参数
		- -XX:+PrintCommandLineFlags

- 3.你平时工作用过的JVM常用基本配置参数有哪些？
- 4.请谈谈你对OOM的认识？

	- java.lang.StackOverFlowError

		- Case：StackOverflowErrorDemo

	- java.lang.OutOfMemoryError:Java heap space

		- Case：JavaHeapSpaceDemo

	- java.lang.OutOfMemoryError:GC overhead limit exceeded

	  GC回收时间长时会抛出OutOfMemoryError。过长的定义是，超过98%的时间用来做GC并且回收了不到2%的堆内存，连续多次GC都只回收了不到2%的极端情况下才会抛出。
	  
	  假设不抛出GC overhead limit错误会发生什么情况呢？
	  那就是GC清理的这么点内存很快会再次填满，迫使GC再次执行，这样就形成恶性循环，CPU使用率一直是100%，而GC缺没有任何成果。

	- java.lang.OutOfMemoryError:Direct buffer memory

	  导致原因：
	  写NIO程序经常使用ByteBuffer来读取或者写入数据，这是一种基于通道（Channel）与缓冲区（Buffer）的I/O方式，它可以使用Native函数库直接分配堆外内存，然后通过一个存储在Java堆里面的DirectByteBuffer对象作为这块内存的引用进行操作。这样能在一些场景中显著提高性能，因为避免了在java堆和Native堆中来回复制数据。
	  
	  ByteBuffer.allocate(capability)第一种方式是分配JVM堆内存，属于GC管辖范围，由于需要拷贝所以速度相对较慢。
	  
	  ByteBuffer.allocateDirect(capability)第一种方式是分配OS本地内存，不属于GC管辖范围，由于不需要内存拷贝，所以速度相对较快。
	  
	  但如果不断分配内存，堆内存很少使用，那么JVM就不需要执行GC，DirectByteBuffer对象们就不会被回收，这时候堆内存充足，但本地内存可能已经使用光了，再次尝试分配本地内存就会出现OutOfMemoryError，那程序就直接崩溃了。

	- java.langOutOfMemoryError:Metaspace

		- 使用java -XX:+PrintFlagsInitial命令查看本机的初始化参数，-XX:Metaspacesize为218103768（大约为20.8M）

	- java.lang.OutOfMemoryError:unable to create new native thread

	  高并发请求服务器时，经常出现如下异常：java.lang.OutOfMemoryError:unbale to create new native thread
	  准确的将该native thread异常与对应的平台有关。
	  
	  导致原因：
	  应用创建了太对线程，一个应用进程创建多个线程，超过系统承载极限。
	  服务器并不允许应用程序创建那么多线程，linux系统默认允许单个进程可以创建的线程数是1024个，如果应用创建超过这个数量，就会报java.lang.OutOfMemoryError:unable to create new native thread
	  解决办法：
	  想办法降低应用程序创建线程的数量，分析应用是否真的需要创建那么多线程，如果不是，改代码将线程数降到最低。
	  对于有的应用，确实需要创建多个线程，远超过linux系统默认的1024个线程的限制，可以通过修改linux服务器配置，扩大linux默认限制。

		- 非root用户登录linux系统测试

		  查看最大线程数量命令：ulimit -u

		- 服务器级别调参调优

		  扩大服务器线程数：
		  vim/etc/security/limits.d/90-nproc.conf

- 5.GC回收算法和垃圾收集器的关系？分别是什么请你谈谈

	- GC算法（引用计数/复制/标清/标整）是内存回收的方法论，垃圾收集器就是算法落地实现。
	- 因为目前为止还没有完美的收集器出现，更加没有万能的收集器，知识针对具体应用最合适的收集器，进行分代收集。
	- 4种主要垃圾收集器

		- 串行垃圾回收器（Serial）

			- 它为单线程环境设计且只使用一个线程进行垃圾回收，会暂停所有的用户线程。所以不适合服务器环境。

		- 并行垃圾回收器（Parallel）

			- 多个垃圾收集线程并行工作，此时用户线程是暂停的，适用于科学计算/大数据处理平台处理等弱交互场景。

		- 并发垃圾回收器（CMS）

			- 用户线程和垃圾收集线程同时执行（不一定是并行，可能交替执行），不需要停顿用户线程，使用对响应时间有要求的场景。

		- 上述3个小总结，G1特殊后面说
		- GI垃圾回收器

			- G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收。

- 6.怎么查看服务器默认的垃圾收集器是哪个？生产上你是如何配置垃圾收集器的？谈谈你的理解？

	- 怎么查看默认的垃圾收集器是哪个？

	  JVM参数：
	  java -XX：+PrintCommandLineFlags -version

	- 默认的垃圾收集器有哪些？

	  java的gc回收的类型主要有几种：
	  UseSerialGC
	  UseParallelGC
	  UseConcMarkSweepGC
	  UseParNewGC
	  UseParallelOldGC
	  UseG1GC

	- 垃圾收集器

		- 部分参数预先说明

			- DefNew

				- Default New Generation

			- Tenured

				- Old

			- ParNew

				- Parallel New Generation

			- PSYoungGen

				- Parallel Scavenge

			- ParOldGen

				- Parallel Old Generation

		- Server/Client模式分别是什么意思

		  使用范围：只需要掌握Server模式即可，Client模式基本不会用。
		  操作系统：
		  	2.1 32位操作系统，不论硬件如何都默认使用Client的JVM模式。
		  	2.2 32位操作系统，2G内存同时有2个CPU以上用Server模式，低于该配置还是Client模式。
		  	2.3 64为only server模式

		- 新生代

			- 串行GC（Serial）/(Serial Copying)

			  串行收集器：Serial收集器
			  一个单线程的收集器，在进行垃圾收集时候，必须暂停其他所有的工作线程知道它收集结束。
			  
			  对应JVM参数是：-XX:+UseSerialGC
			  开启后会使用：Serial（Young区用）+Serial Old（Old区用）的收集器组合。
			  表示：新生代、老年代都会使用串行回收收集器，新生代使用复制算法，老年代使用标记-整理算法。

			- 并行GC（ParNew）

			  ParNew（并行）收集器
			  一句话：使用多线程进行垃圾回收，在垃圾收集时，会Stop-the-World暂停其他所有的工作线程直到它收集结束。
			  
			  ParNew收集器其实就是Serial收集器新生代的并行多线程版本，最常见的应用场景是配合老年代的CMS GC工作，其余的行为和Serial收集器完全一样，ParNew垃圾收集器在垃圾收集过程中同样也要暂停所有其他的工作线程。它是很多java虚拟机运行在Server模式下新生代的默认垃圾收集器。
			  
			  常用对应JVM参数：-XX:+UseParNewGC 启用ParNew收集器，只影响新生代的收集，不影响老年代。
			  
			  开启上述参数后，会使用：ParNew（Young区用）+Serial Old的收集器组合，新生代使用复制算法，老年代采用标记-整理算法。

			- 并行回收GC(Parallel)/(Parallel Scavenge)

			  Parallel Scavenge收集器类似ParNew也是一个新生代垃圾收集器，使用复制算法，也是一个并行的多线程的垃圾收集器，俗称吞吐量有限收集器。一句话：串行收集器在新生代和老年代的并行化。
			  
			  它重点关注的是：
			  可控制的吞吐量（Thoughput=运行用户代码时间/(运行用户代码时间+垃圾收集时间)，也即比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99%）。高吞吐量意味着高效利用CPU的时间，它多用于在后台运算而不需要太多交互的任务。
			  
			  自适应调节策略也是ParallelScavenge收集器与ParNew收集器的一个重要区别。（自使用调节策略：虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间（-XX:MaxGCPauseMillis）或最大的吞吐量。）
			  
			  常用JVM参数：-XX:+UseParallelGC或-XX:+UseParallelOldGC（可互相激活）使用Parallel Scanvenge收集器

		- 老年代

			- 串行GC(Serial Old)/(Serial MSC)
			- 并行GC（Parallel Old）/(Parallel MSC)

			  Parallel Old收集器是Parallel Scavenge的老年代版本，使用多线程的标记-整理算法，Parallel Old收集器在JDK1.6才开始提供。
			  
			  JVM常用参数：
			  -XX:+UseParallel Old 使用Parallel Old收集器，设置该参数后，新生代Parallel+老年代Parallel Old

			- 并发标记清除GC（CMS）

			  CMS收集器（Concurrent Mark Sweep：并发标记清除）是一种以获取最短回收停顿时间为目标的收集器。
			  
			  适合应用在互联网站或者B/S系统的服务器上，这类应用尤其重视服务器的响应速度，希望系统停顿时间最短。
			  
			  CMS非常适合堆内存大、CPU核数多的服务器端应用，也是G1出现之前大型应用的首选收集器。
			  
			  Concurrent Mark Sweep并发标记清除，并发收集低停顿，并发指的是与用户线程一起执行。
			  
			  开启该收集器的JVM参数：-XX:+UseConcMarkSweepGC 开启该参数后会自动将-XX:+UseParNewGC打开
			  
			  开启该参数后，使用ParNew（Young区用）+CMS（Old区用）+Serial Old的收集器组合，Serial Old将作为CMS出错的后备收集器。

				- 4步过程

					- 初始标记（CMS initial mark）

					  只是标记一下GC Roots能直接关联的对象，速度很快，仍然需要暂停所有的工作线程。

					- 并发标记（CMS concurrent mark）和用户线程一起

					  进行GC Roots跟踪的过程，和用户线程一起工作，不需要暂停工作线程。主要标记过程，标记全部对象。

					- 重新标记（CMS remark）

					  为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录，仍然需要暂停所有的工作线程。
					  
					  由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正。

					- 并发清除（CMS concurrent sweep）和用户线程一起

					  清楚GC Roots不可达对象，和用户线程一起工作，不需要暂停工作线程。基于标记结果，直接清理对象。
					  
					  由于耗时最长的并发标记和并发清除过程中，垃圾收集线程可以和用户现在一起并发工作，所以总体上来看CMS收集器的内存回收和用户线程是一起并发执行。

				- 优缺点

					- 优点

						- 并发收集低停顿

					- 缺点

						- 并发执行，对CPU资源压力大

						  由于并发进行，CMS在收集与应用线程会同时增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器会以STW的方式进行一次GC，从而造成较大停顿时间。

						- 采用的标记清除算法会导致大量碎片

						  标记清除算法无法整理空间碎片，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过担保机制对堆内存进行压缩。CMS也提供了参数-XX:CMSFullGCsBeForeCompaction（默认0，即每次都进行内存整理）来指定多少次CMS收集之后，进行一次压缩的Full GC。

		- 垃圾收集器配置代码总结

	- 如何选择垃圾收集器

	  组合的选择：
	  单CPU或小内存，单机程序
	  	-XX:+UseSerialGC
	    2.多CPU，需要最大吞吐量，如后台计算型应用
	  	-XX:+UseParallelGC 或
	  	-XX:+UseParallelOldGC
	    3.多CPU，追求低停顿时间，需要快速响应如互联网应用
	  	-XX:+UseConcMarkSweepGC
	  	-XX:+PaeNewGC

- 7.G1垃圾收集器

	- 以前收集器特点

		- 年轻代和老年代是各自独立且连续的内存块
		- 年轻代收集使用单eden+S0+s进行复制算法
		- 老年代收集必须扫描整个老年代区域
		- 都是以尽可能少而快速地执行GC为设计原则。

	- G1是什么

	  G1（Garbage-First）收集器，是一款面向服务端应用的收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能的满足垃圾收集暂停时间的要求。另外还具有一下特性：
	  
	  像CMS收集器一样，能与应用程序线程并发执行。
	  整理空间空间更快
	  需要更多的时间来预测GC停顿时间
	  不希望牺牲大量的吞吐性能
	  不需要更大的Java Heap
	  
	  G1收集器的设计目标是取代CMS收集器，它同CMS相比，在以下方面表现的更出色：
	  G1是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。
	  G1的Stop The World（STW）更可控，G1在停顿时间上添加了预测机制，用户可以指定期望停顿时间。

	- 底层原理

		- Region区域化垃圾收集器

		  区域化内存划片Region，整体编为一系列不连续的内存区域，避免了全内存的GC操作。
		  
		  核心思想是将整个堆内存区域分成大小相同的子区域（Region），在JVM启动时会自动设置这些子区域的大小。
		  
		  在堆的使用上，G1并不要求对象的存储一定是物理上连续的只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换。启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小（1MB-32MB，且必须是2的幂），默认将整个堆划分为2048个分区。
		  大小范围在1MB-32MB，最多能设置2048个区域。也即能够支持的最大内存为：32MB*2048=65536MB=64G内存。
		  
		  G1将新生代、老年代的物理空间取消了。
		  
		  G1算法将堆划分为若干个区域（Region），它仍然属于分代收集器
		  
		  这些Region的一部分包含新生代，新生代的垃圾收集依然采用暂停所有应用线程的方式，将存活对象拷贝到老年代或者Survivor空间。
		  
		  这些Region的一部分包含老年代，G1收集器通过将对象从一个区域复制到另一个区域，完成了清理工作。这就意味着，在正常的处理过程中，G1完成了堆的压缩（至少是部分堆的压缩），这样也就不会有CMS内存碎片问题的存在了。
		  
		  在G1中，还有一种特殊的区域，叫Humongous（巨大的）区域。如果一个对象占用的空间超过了分区容量的50%以上，，G1收集器就认为这是一个巨型对象。这些巨型对象默认直接会被分配在老年代，但是如果它是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响。为了解决这个问题，G1划分了一个Humongous区，它用来专门存放巨型对象。如果一个H区装不下一个巨型对象，那么G1会寻找连续的H分区来存储。为了能找到连续的H区，有时候不得不启动Full GC。

			- 最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可。

		- 回收步骤

		  G1收集器下的Young GC
		  
		  针对Eden区进行收集，Eden区耗尽后会被触发，主要是小区域收集+形成连续的内存块，避免内存碎片。
		  Eden区的数据移动到新的Survivor区，部分数据晋升到Old区。
		  Survivor区的数据移动到新的Survivor区，部分数据晋升到Old区。
		  最后Eden区收集干净了，GC结束，用户的应用程序继续执行。

		- 4步过程

		  初始标记：只标记GC Roots能直接关联到的对象
		  并发标记：进行GC Roots Tracing的过程
		  最终标记：修正并发标记期间，因程序运行导致标记发生变化的那一部分对象
		  筛选回收：根据时间来进行价值最大化的回收。

	- case案例
	- 常用配置参数（了解）

	  开发人员仅仅声明一下参数即可：
	  三步归纳：开始G1+设置最大内存+设置最大停顿时间
	  
	  -XX:+UseG1GC 
	   -Xmx32g  
	  -XX:MaxGCPauseMillis=100
	  
	  -XX:MaxGCPauseMillis=n：最大GC停顿时间单位毫秒，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间。

		- -XX:+UseG1GC
		- -XX:G1HeapRegionSize=n:设置的G1区域的大小。值是2的幂，范围是1MB到32MB。目标是根据最小的Java堆大小划分出约2048个区域。
		- -XX:MaxGCPauseMillis=n：最大GC停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间。
		- -XX:InitiatingHeapOccupancyPercent=n：堆占用了多少的时候就触发GC，默认为45
		- -XX:ConcGCThreads=n：并发GC使用的线程数。
		- -XX:G1ReservePercent=n：设置作为空闲的预留内存百分比，以降低目标空间溢出的风险，默认值是10%

	- 和CMS相比的优势

	  两个优势：
	  G1不会产生内存碎片
	  是可以精确控制停顿。该收集器是把整个堆（新生代、老生代）划分成多个固定大小的区域，每次根据允许停顿时间去手机垃圾最多的区域。

	- 小总结

- 8.强引用、软引用、弱引用、虚引用分别是什么？

	- 整体架构
	- 强引用（默认支持模式）

	  当内存不足，JVM开始垃圾回收，对于强引用的对象，就算是出现了OOM也不会对该对象进行回收，死都不收。
	  
	  强引用是最常见的普通对象引用，只要还有前饮用指向一个对象，就能表明对象还“活着”，垃圾收集器不会碰到这种对象。在Java中最常见的就是强引用，把一个对象赋给一个引用变量，这个引用变量就是一个强引用。当一个对象被强引用变量引用时，它处于可达状态，它是不可能被垃圾机制回收的，即使该对象以后永远都不能被用到，JVM也不会回收。因此强引用是造成Java内存泄露的主要原因之一。
	  
	  对弈一个普通的对象，如果没有其他的引用关系，只要超过了引用的作用域或者显示地将相应（强）引用赋值为null，一般认为就是可以被垃圾收集的了（当然具体回收时还要看垃圾收集策略）。

		- Case：StrongReferenceDemo

	- 软引用

	  软引用是一种相对强化了一些的引用，需要用java.lang.ref.SoftReference类来实现，可以让对象豁免一些垃圾收集。
	  
	  对于只有软引用的对象来说，
	  当系统内存充足时，它不会被回收
	  当系统内存不足时，它会被回收
	  
	  软引用通常用在对内存敏感的程序中，比如高速缓存就有用到软引用，内存够用的时候就保留，不够用就回收。

		- Case：SoftReferenceDemo

	- 弱引用

	  弱引用需要用java.lang.ref.WeakReference类来实现，它比软引用的生存期更短。
	  
	  对于软引用对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，都会回收该对象占用的内存。

		- Case：WeakReferenceDemo
		- 软引用和所引用的使用场景

		  假如有一个应用需要读取大量的本地图片：
		  如果每次读取图片都从硬盘读取则会严重影响性能。
		  如果一次性全部加载到内存中有可能造成内存泄露。
		  此时使用软引用可以解决这个问题。
		  
		  设计思路：用一个HashMap来保存图片的路径和相应图片对象关联的软引用之间的映射关系，在内存不足时，JVM会自动回收这些缓存图片对象所占用的空间，从而有效地避免OOM的问题。
		  
		  Map<String,SoftReference<Bitmap>> imageCache = new HashMap<String,SoftReference<Bitmap>>();

		- 你知道所引用的话，能谈谈weakHashMap吗

			- Case:WeakHashMapDemo

	- 虚引用

	  虚引用需要java.lang.refPhantonReference类来实现。
	  
	  顾名思义，就是形同虚设，与其他几种引用都不同，虚引用并不会决定对象的生命周期。
	  如果一个对象仅持有虚引用，那么它就和没有任何引用一样，在任何时候都可能被垃圾回收器回收，它不能单独使用也不能通过它访问对象，虚引用必须和队列（ReferenceQueue）联合使用。
	  
	  虚引用的主要作用是跟踪对象被垃圾回收的状态。仅仅是提供了一种确保对象被finalize以后，做某些事情的机制。
	  
	  PhantomReference的get方法总是返回null，因此无法访问对应的引用对象。其意义在于说明一个对象那个已经进入finalization阶段，可以被gc回收，用来实现比finalization机制更灵活的回收操作。
	  
	  换句话说，设置虚引用关联的唯一目的，就是在这个对象被收集器回收的时候收到一个系统通知或者后续添加进一步的处理。
	  Java技术允许使用finalize（）方法在垃圾收集器将对象从内存中清楚之前做必要的清理工作。

		- 引用队列

		  我被回收前需要被引用队列保存下。

			- case：ReferenceQueueDemo

		- case：PhantomReferenceDemo

	- GCRoots和四大引用小总结

- 9.生产环境服务器变慢，诊断思路和性能评估谈谈？

	- 整机：top

		- uptime，系统性能命令的精简版

	- CPU:vmstat

		- vmstat

			- 查看CPU（包含不限于）

			  vmstat -n 2 3
			  一般vmstat工具的使用是通过两个数字参数来完成的，第一个参数是采样的时间间隔数单位是秒，第二个参数是采样的次数。
			  
			  -procs
			  r：运行等待CPU时间片的进程数，原则上1核的CPU运行队列不要超过2，整个系统的运行队列不能超过总核数的2倍，否则代表系统压力过大。
			  -cpu
			  us：用户进程消耗CPU时间百分比，us值高，用户进程小号CPU时间多，如果长期大于50%，优化程序；
			  sy：内核进程消耗的CPU百分比；

			- 查看额外

				- 查看所有cpu核信息

					- mpstat -P ALL 2

				- 每个进程使用cpu的用量分解信息

					- pidstat -u1 -p进程编号

	- 内存：free

		- 应用程序可用内存数

		  free 
		  free -g
		  free -m
		  
		  -经验值
		  应用程序可用内存/系统物理内存>70%内存充足。
		  应用程序可用内存/系统物理内存<20%内存不足，需要增加内存。
		  20%<应用程序可用内存/系统物理内存<70%内训基本够用

		- 查看额外

			- pidstat -p 进程号 -r 采样间隔秒数

	- 硬盘：df

		- 查看磁盘剩余空间数

		  df -h

	- 磁盘IO：iostat

		- 磁盘I/O性能评估

		  iostat -xdk 2 3

		- 查看额外

			- pidstat -d采样间隔秒数 -p 进程号

	- 网络IO:ifstat

		- 默认本地没有，下载ifstat
		- 查看网络IO

- 10.假设生产环境出现CPU占用过高，请谈谈你的分析思路和定位

	- 结合Linux和JDK命令一块分析
	- 案例步骤

		- 1.先用top命令找出CPU占比最高的
		- 2.ps -ef或者jps进一步定位，得知是一个怎么样的一个后台程序惹事
		- 3.定位到具体线程或者代码

			- ps -mp进程 -o THREAD,tid，time
			- 参数解释

				- -m显示所有的线程
				- -p pid进程使用cpu的时间
				- -o该参数后是用户自定义格式

		- 4.将需要的线程ID转换为16进制格式（英文小写格式）

			- printf "%x\n"有问题的线程ID

		- 5.jstack进程ID | grep（16进制线程ID小写英文） -A60

- 11.对于JDK自带的JVM监控和性能分析工具用过哪些？一般你是怎么用的？

	- 概览
	- 性能监控工具

		- jps

			- 官网
			- 解释
			- Case

		- jinfo
		- jmap
		- jstat
		- jstack


