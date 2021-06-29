---
title: JVM-垃圾回收器
top: false
cover: true
author: 张文军
date: 2020-07-20 20:08:07
tags: 
    - JVM
    - JVM垃圾回收器
    - JVM常用参数
category: JVM
summary: JVM-垃圾回收器-JVM常用参数
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

## JVM 常用参数

> Java 虚拟机栈

```java
//-Xss 这个虚拟机参数来指定每个线程的 Java 虚拟机栈内存大小，
//在 JDK 1.4 中默认为 256K，而在 JDK 1.5+ 默认为 1M

java -Xss2M HeakTheJava


// 当线程请求的栈深度超过最大值，会抛出 StackOverflowError 异常；
// 栈进行动态扩展时如果无法申请到足够内存，会抛出 OutOfMemoryError 异常。
```

> Java 堆

```java
// 堆不需要连续内存，并且可以动态增加其内存，增加失败会抛出 OutOfMemoryError 异常。

java -XmsGM -Xmx2G HeakTheJava

```

## JVM 垃圾收集器

![垃圾收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595247718.png)

>以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

- 单线程与多线程：单线程指的是垃圾收集器只使用一个线程，而多线程使用多个线程；
- 串行与并行：串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并行指的是垃圾收集器和用户程序同时执行。``除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行``。

### Serial 收集器 ( client场景下年轻代默认收集器)

![Serial 收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595247902.png)

> Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，在单个 CPU 环境下，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

### ParNew 收集器

![ParNew 收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595248230.png)

>它是 Serial 收集器的多线程版本。

它是 Server 场景下默认的新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合使用。

### Serial Old 收集器

![Serial Old 收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595248571.png)

>是 Serial 收集器的老年代版本，也是给 Client 场景下的虚拟机使用。

如果用在 Server 场景下，它有两大用途：

- 在 JDK 1.5 以及之前版本（Parallel Old 诞生以前）中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

### Parallel Scavenge 收集器

与 ParNew 一样是多线程收集器。

其它收集器目标是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，因此它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户程序的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的：新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略（GC Ergonomics），就不需要手工指定新生代的大小（-Xmn）、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

### Parallel Old 收集器

![Parallel Old 收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595249035.png)

是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

### CMS 收集器

![CMS 收集器](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595249126.png)

>CMS（Concurrent Mark Sweep），Mark Sweep 指的是标记 - 清除算法。

分为以下四个流程：

- 初始标记：仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记：进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记：为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除：不需要停顿。

>在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

- 吞吐量低：低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

### G1 收集器

G1（Garbage-First），它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 G1 可以直接对新生代和老年代一起回收。

![堆空间结构](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595249916.png)

G1 把堆划分成多个大小相等的独立区域（Region），新生代和老年代不再物理隔离。

![G1堆收集](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1595249983.png)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤：

- 初始标记
- 并发标记
- 最终标记：为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
- 筛选回收：首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点：

空间整合：整体来看是基于“标记 - 整理”算法实现的收集器，从局部（两个 Region 之间）上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
可预测的停顿：能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

### ``总结``

垃圾收集器可大致分为以下几种：

- 按照串行/并行：除了 CMS 和 G1 以外其他垃圾收集器都是串行
- 按照垃圾回收的区域：
  - Serial ，Parallel Scavenge ，ParNew为年轻代收集器
  - Serial Old，Parallel Old ，CMS 为老年代收集器
  - G1即可以收集老年代也可以收集年轻代

总的来说有：

- Serial，Serial Old，ParNew（Serial的多线程版本）
- Parallel，Parallel Old
- CMS ，G1

这7种。

### Minor GC 和 Full GC

Minor GC：回收新生代，因为新生代对象存活时间很短，因此 Minor GC 会频繁执行，执行的速度一般也会比较快。

Full GC：回收老年代和新生代，老年代对象其存活时间长，因此 Full GC 很少执行，执行速度会比 Minor GC 慢很多。
