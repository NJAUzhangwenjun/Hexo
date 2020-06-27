---
title: Volatile关键字实现原理
date: 2020-01-30 08:45:21
tags: 4、Volatile关键字实现原理
category: 多线程
summary: Volatile关键字实现原理
top: true
cover: true
author: 张文军
---

![logo](/images/favicon.png)

## 1、认识volatile关键字

程序举例

用一个线程读数据，一个线程改数据

存在数据的不一致性

## 2、机器硬件CPU与JMM

（1）CPU Cache模型
![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085000864.png)


![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085048228.png)

程序的局部执行原理

（2）CPU缓存的一致性问题

![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085205248.png)

 


解决方案：

1）总线加锁（粒度太大）

2）MESI（）

a.         读操作：不做任何事情，把Cache中的数据读到寄存器

b.         写操作：发出信号通知其他的CPU讲改变量的Cache line置为无效，其他的CPU要访问这个变量的时候，只能从内存中获取。

Cache line  CPU的cache中会增加很多的Cache line

![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085431024.png)

（3）Java内存模型

![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085502135.png)

1)         主存中的数据所有线程都可以访问（共享数据）

2)         每个线程都有自己的工作空间，（本地内存）（私有数据）

3)         工作空间数据：局部变量、内存的副本

4)         线程不能直接修改内存中的数据，只能读到工作空间来修改，修改完成后刷新到内存

![](http://myblog-1258908231.cos.ap-shanghai.myqcloud.com/Volatile%E5%85%B3%E9%94%AE%E5%AD%97%E5%AE%9E%E7%8E%B0%E5%8E%9F%E7%90%86/20200130085601323.png)

## 3、Volatile关键字的语义分析

volatile作用：让其他线程能够马上感知到某一线程多某个变量的修改

（1）保证可见性

对共享变量的修改，其他的线程马上能感知到

不能保证原子性  读、写、（i++）

（2）保证有序性

重排序（编译阶段、指令优化阶段）

输入程序的代码顺序并不是实际执行的顺序

重排序后对单线程没有影响，对多线程有影响

Volatile

Happens-before

volatile规则：

对于volatile修饰的变量：

​	（1）volatile之前的代码不能调整到他的后面

​	（2）volatile之后的代码不能调整到他的前面（as if seria）

​	（3）霸道（位置不变化）

（3）volatile的原理和实现机制(锁、轻量级)   

  HSDIS   --反编译---汇编

  Java --class---JVM---》ASM文件

  Volatile  int  a ;

 Lock :a

## 4、Volatile的使用场景

（1）状态标志（开关模式）

```java
public class ShutDowsnDemmo extends Thread{

​    private volatile boolean started=false;

 

​    @Override

​    public void run() {

​        while(started){

​            dowork();

​        }

​    }

​    public void shutdown(){

​        started=false;

​    }

}
```



（2）双重检查锁定（double-checked-locking）

DCL (单例模式)（7）

```java
public class Singleton {

​    private volatile static Singleton instance;

​    public static Singleton getInstance(){

​        if(instance==null){

​            synchronized (Singleton.class){

​                instance=new Singleton();

​            }

​        }

​        return instance;

​    }

}
```



（3）需要利用顺序性

## 5、volatile与synchronized的区别

（1）使用上的区别

Volatile只能修饰变量，synchronized只能修饰方法和语句块

（2）对原子性的保证

synchronized可以保证原子性，Volatile不能保证原子性

（3）对可见性的保证

都可以保证可见性，但实现原理不同

Volatile对变量加了lock，synchronized使用monitorEnter和monitorexit  monitor  JVM

（4）对有序性的保证

Volatile能保证有序，synchronized可以保证有序性，但是代价（重量级）并发退化到串行

（5）其他

synchronized引起阻塞

Volatile不会引起阻塞