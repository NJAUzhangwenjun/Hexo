---
title: 并发-JUC工具包
top: false
cover: true
author: 张文军
date: 2020-07-19 18:38:40
tags: 
    - 并发-JUC工具包
    - 并发
    - Atomic
    - CylicBarrier
    - CountDownLatch
    - Semaphore
category: 并发
summary: 并发- Atomic- CylicBarrier- CountDownLatch- Semaphore
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

```java

package fuxi.concurrency;

import org.junit.Test;

import java.util.concurrent.*;
import java.util.concurrent.atomic.AtomicInteger;

import static java.lang.System.out;
import static java.lang.Thread.currentThread;
import static java.util.concurrent.TimeUnit.SECONDS;

/**
 * @Description: TODO
 * @author 张文军
 * @Company: njauit.cn
 * @version: 1.0
 * @date 2020/7/1816:17
 */
public class JUC {

		private Executor executor = ThreadPool.getThreadPool();

	/**
	 * Atomic 原子操作
	 *
	 * 分类：
	 *
	 * 基本类型：
	 * 		整形（AtomicInteger）、长整型（AtomicLong）、布尔型（AtomicBoolean）
	 *
	 * 数组类型：使用原子的方式更新数组里的某个元素
	 * 		整形（AtomicIntegerArray）、长整型（AtomicLongArray）、引用类型（AtomicReferenceArray）
	 *
	 * 引用类型
	 * 		AtomicReference：引用类型原子类、AtomicStampedReference：原子更新引用类型里的字段原子类、
	 * 		AtomicMarkableReference ：原子更新带有标记位的引用类型
	 */

	@Test
	public void atomic() {
		AtomicInteger atomicInteger = new AtomicInteger();
		out.println("atomicInteger = " + atomicInteger.incrementAndGet());
	}


	/**
	 * Semaphore ：信号量：共享资源的互斥使用，控制并发数
	 */

	@Test
	public void semaphoreTest() {
		/**
		 * 一个厕所有三个坑，每次只能最多同时有三个人使用
		 */
		Semaphore toilet = new Semaphore(3);

		/**
		 * 上厕所的人数
		 */
		int beauty = 20;
		while (beauty-- > 0) {
			new Thread(() -> {
				//抢坑
				try {
					toilet.acquire();
					out.println(currentThread().getName() + "抢到了坑");
					out.println(currentThread().getName() + "正在上厕所。。。。。。。。。。");
					try { SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
				} catch (InterruptedException e) {
					e.printStackTrace();
				} finally {
					out.println(currentThread().getName() + "厕所上完了");
					toilet.release();
				}

			}, "美女--" + beauty + " 号").start();
		}

		while (true)
			;

	}


	/**
	 * CountDownLatch : 倒数计数器：让一个线程阻塞等待其他线程执行完成后才执行
	 */
	@Test
	public void countDownLatch() {
		/**
		 * 上完晚自习后班长锁门：只有当学生都走完了他才可以锁门
		 */

		CountDownLatch student = new CountDownLatch(10);

		int studentsNumber = 10;

		while (studentsNumber-- > 0) {
			new Thread(() -> {
				// TODO:
				out.println(currentThread().getName() + " 关门离开了！");
				student.countDown();
			}, studentsNumber + "号学生").start();
		}
		new Thread(() -> {
			// TODO:
			try {
				/**
				 * 如果计数器不为0，班长就一直阻塞等待；
				 */
				student.await();
				out.println(currentThread().getName() + "锁门离开！");
			} catch (InterruptedException e) {
				e.printStackTrace();
			}
		}, "monitor").start();

		while (true)
			;
	}


	/**
	 * CyclicBarrier : 循环栅栏：
	 */
	@Test
	public void cyclicBarrier() {
		/**
		 * 银行保险柜有五把钥匙，分别由5位经理保管，只有五把钥匙同时插入的时候才能打开，
		 * 但五位经理来上班的时间各不相同，所以要等经理们都到了才可以开启保险柜；
		 */

		CyclicBarrier bankKey = new CyclicBarrier(5);

		int numberOfManagers = 5;

		while (numberOfManagers-- > 0) {
			new Thread(() -> {
				out.println(currentThread().getName() + "正在赶来的路上。。。");
				try { TimeUnit.SECONDS.sleep(((int) (Math.random() * 10)) % 3); } catch (InterruptedException e) {
					e.printStackTrace();
				}
				out.println(currentThread().getName() + "到了！");
				try {
					bankKey.await();
					out.println(currentThread().getName() + "开完保险柜后去干其他事了！");
				} catch (Exception e) {
					e.printStackTrace();
				}
			}, numberOfManagers + "号经理").start();
		}

		while (true) {

		}
	}

}


```
