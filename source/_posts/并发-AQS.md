---
title: 并发-AQS
top: false
cover: true
author: 张文军
date: 2020-07-19 18:31:40
tags: 
    - 并发
    - AQS
category: 并发
summary: 并发-AQS
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

```java
package fuxi.concurrency;

import java.util.concurrent.locks.AbstractQueuedSynchronizer;

/**
 * @Description: TODO
 * @author 张文军
 * @Company: njauit.cn
 * @version: 1.0
 * @date 2020/7/1813:35
 */
public class AQS extends AbstractQueuedSynchronizer {
	/**
	 * AQS : abstractQueueSynchronizer 抽象队列同步器:
	 * 		是实现同步所的框架，子类继承实现AQS模板方法，然后将子类作为组件的内部类；
	 *
	 *  - state （static volatile 共享资源，也就是多线程获取锁的时候需要获得的）
	 *  - CLH （FIFO 双向等待队列）
	 *
	 *
	 * 锁模式：
	 *
	 * 	- 独占模式：资源只能由一个线程获得，如 ReentrantLook
	 * 	- 共享模式：资源可以被多个线程所获取并执行，如：Semaphore，CountDownLatch等
	 *
	 * 获取锁的方式 CAS
	 *
	 *
	 * 一个线程获取资源的步骤：
	 *
	 *  - tryAcquire(arg)尝试获取共享资源，如果成功，放回true，没成功，则将当前线程以独占或共享模式添加到 等待队列中
	 *  - 当线程被加入到等待队列后，acquireQueue（arg）会通过自旋CAS的方式，不断的尝试获取资源，直到获取成功；
	 *  - 当获取成功，将会自我中断自旋 并acquireQueue返回为true
	 *  - 当线程被中断，也会 acquireQueue 也将返回为true
	 *
	 *
	 * 释放资源：tryRelease（int）
	 *
	 * - 获取当前节点的下一个节点
	 * - 调整等待队列
	 * - 中断 当前节点的 acquireQueue
	 *
	 */


	@Override
	protected boolean tryAcquire(int arg) {
		return super.tryAcquire(arg);
	}


	@Override
	protected boolean tryRelease(int arg) {
		return super.tryRelease(arg);
	}


	/**
	 * synchronized 和 ReEntrantLook 对比：
	 *
	 * - 两者都是可重入锁（可重入锁：一个线程可以多次获取已经获得的同一个资源（锁））
	 *
	 * - synchronized依赖于JVM 而ReentrantLook是Java API级别的
	 * 		synchronized是依赖于JVM实现的，对外是不透明的，而ReentrantLook是Java实现的，可以看到源码；
	 *
	 * - ReentrantLook比synchronized增加了一些功能；- 等待可中断；- 可实现公平锁；- 可实现选择性通知（锁可以绑定多个条件）
	 *
	 */

}

```
