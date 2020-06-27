---
title: Java内存模型
date: 2020-01-17 06:22:22
tags: 1、Java内存模型
category: JVM
summary: Java内存模型
top: true
cover: true
author: 张文军
---
# Java内存模型

![](/images/favicon.png)

## 1、线程和进程的区别

   [进程与线程的一个简单解释](http://www.ruanyifeng.com/blog/2013/04/processes_and_threads.html)
   ----
   1、 进程是执行着的应用程序，线程是进程中的执行单元。
   2、 进程是资源分配单元，线程是执行单元
   3、 进程之间相互独立，同一进程的线程共用进程资源。
   4、 进程间通讯通过IPC，同一进程间线程通讯通过写入进程数据段来通讯，需要用到sychronized与voltaile等线程同步手段保持数据的一致性。
   5、线程切换比进程切换快而且所需资源较少。


## 2、线程、进程与jvm之间的关系

 > Java 编写的程序都运行在在 Java虚拟机（JVM）中，每用java命令启动一个java应用程序，就会启动一个JVM进程。
 > 
 > 在同一个JVM进程中，有且只有一个进程，就是它自己。在这个JVM环境中，所有程序代码的运行都是以线程来运行的。
 > 
 > JVM 找到程序程序的入口点 main()，然后运行 main() 方法，这样就产生了一个线程，这个线程称之为主线程。
 > 
 > 当 main 方法结束后，主线程运行完成。JVM 进程也随即退出。
 > 
 > 
 > (JVM什么时候启动？类被调用    JVM线程---》其他的线程（main）)
![JVM](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1587851385.png)

## 3、JVM内存区域
![JVM内存区域](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1587851785.png)

JVM 内存区域主要分为线程私有区域【程序计数器、虚拟机栈、本地方法区】、线程共享区域【JAVA 堆、方法区】、直接内存。

- 方法区：类信息、常量、static 、JIT（JAVA即时编译） 、运行时常量池（信息共享）
- Java堆区：实例对象  数组  GC   （信息共享，垃圾收集的最重要的内存区域）   (OOM)
- VM stack：Java方法在运行的内存模型   (OOM)
- PC：java线程的私有数据，这个数据就是执行下一条指令的地址
- Native method stack:  JVM的native ( NIO 提供了基于 Channel 与 Buffer 的 IO 方式, 它可以使用 Native 函数库直接分配堆外内存, 然后使用DirectByteBuffer 对象作为这块内存的引用进行操作)

线程私有数据区域生命周期与线程相同, 依赖用户线程的启动/结束 而 创建/销毁(在 HotspotVM 内, 每个线程都与操作系统的本地线程直接映射, 因此这部分内存区域的存/否跟随本地线程的生/死对应)。
线程共享区域随虚拟机的启动/关闭而创建/销毁。
![JVM内存区域](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1587852337.png)

## 4、Java内存模型   Java memory model   JMM(规范,抽象的模型) 

![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/20200119030815740.png)

  1）主内存：共享的信息
  2）工作内存：私有信息，基本数据类型，直接分配到工作内存，引用的地址存放在工作内存，引用的对象存放在堆中
  3）工作方式：
   - A  线程修改私有数据，直接在工作空间修改
   -  B  线程修改共享数据，把数据复制到工作空间中去，在工作空间中修改，修改完成以后，刷新内存中的数据

## 5、硬件内存架构与java内存模型

### 1、硬件架构
   ![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/20200119033806019.png)
   a)	CPU缓存的一致性问题：并发处理的不同步
   b)	解决方案：
   - 	总线加锁： 降低CPU的吞吐量
   - ii.	缓存上的一致性协议（MESI）

     >当CPU在CACHE中操作数据时，如果该数据是共享变量，数据在CACHE读到寄存器中，进行新修改，并更新内存数据, CaCHE  LINE置无效，其他的CPU就从内存中读数据

### 2、Java线程与硬件处理器
![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/20200119035148842.png)

### 3、Java内存模型与硬件内存架构的关系
![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Java%E5%86%85%E5%AD%98%E6%A8%A1%E5%9E%8B/20200119035324519.png)
 *交叉：数据的不一致*

### 4、Java内存模型的必要性
   Java内存模型的作用：**规范内存数据和工作空间数据的交互**


## 6、并发编程的三个重要特性

  1. 原子性：不可分割 x=1
  2. 可见性：程序只能操作自己工作空间中的数据
  3. 有序性：程序中的顺序不一定就是程序执行的顺序
    - 编译重排序
    - 指令重排序
      （都是为了提高效率）

## 7、JMM对三个特征的保证

### 1、JMM 与原子性 ：
A) X=10  写  原子性   如果是私有数据具有原子性，如果是共享数据没原子性（读写）  

B) Y=x  没有原子性
   -  a)	 把数据X读到工作空间（原子性）
   -  b)	 把X的值写到Y（原子性）

C) I++ 没有原子性
   -  a)	读i到工作空间
   -  b)	+1；
   -  c)	刷新结果到内存

D) Z=z+1 没有原子性
   -  a)	读z到工作空间
   -  b)	+1；
   -  c)	刷新结果到内存

**多个原子性的操作合并到一起没有原子性**
 保证方式：
 - Synchronized
 - JUC： Lock的lock

### 2、JMM 与可见性 ：

- Volatile:在JMM模型上实现MESI协议

- Synchronized:加锁

- JUC    Lock的lock

### 3、JMM与有序性

- Volatile：
- Synchronized：
- Happens-before原则：
    1）程序次序原则
    2）锁定原则  ：后一次加锁必须等前一次解锁
    3）Volatile原则：霸道原则
    4）传递原则：A---B ---C    A--C



## 8、总结

JVM内存区域和JMM的关系

JMM和硬件的关系

JMM和并发编程三个重要特征（有序性  as-if-seria【单线程中重排后不影响执行的结果，多线程】   happens-before  ）


