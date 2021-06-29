---
title: 并发-Synchronized详解
top: false
cover: true
author: 张文军
date: 2020-07-19 18:23:58
tags: 
    - 并发
    - synchronized
    - 单例模式
    - 饿汉模式
    - 懒汉模式
category: 并发
summary: 并发-Synchronized详解-单例模式（饿汉-懒汉）
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

```java

package fuxi.concurrency;

import java.util.concurrent.Executor;
import java.util.concurrent.Executors;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;

import static fuxi.concurrency.HungrySingleton.getInstance;
import static java.lang.System.out;
import static java.util.concurrent.TimeUnit.SECONDS;

/**
 * @Description: TODO
 * @author 张文军
 * @Company: njauit.cn
 * @version: 1.0
 * @date 2020/7/1622:46
 */
public class synchronizedTest {

	public static Executor executor = null;


	public synchronizedTest() {
		executor = new ThreadPoolExecutor(1, 1, 0, SECONDS, new LinkedBlockingQueue<Runnable>(100),
				Executors.defaultThreadFactory(), new ThreadPoolExecutor.AbortPolicy());
	}

	public static void threadTest(String[] args) {

		/**
		 * 首先，run方法是Native方法，属于系统级别的调用，而start是Java实现的方法
		 * 其次是Start方法后该线程只是开启了可执行状态，也就是到了Runnable状态，
		 * 而在执行Run方法后，该线程处于Running状态，这时候才是真正的执行；
		 *
		 * ThreadPoolExecutor ：
		 * 	核心线程数，最大线程数，空闲线程存活时间，时间单位，阻塞队列，线程工厂，拒绝策略
		 *
		 *
		 * 	AQS：AbstractSynchronizedQueue 同步队列器；底层是由一个双向的HCL双向队列组成，
		 * 	规定了Lock锁的基本规则和属性方法
		 *
		 */


		synchronizedTest sy = new synchronizedTest();
		int i = 100;
		while (i-- > 0) {
			int finalI = i;
			executor.execute(new Runnable() {
				@Override
				public void run() {
					// TODO:
					out.println("i = " + finalI);
				}
			});
			out.println("finalI = " + finalI);
		}

	}


	/**
	 * 说一说自己对于 synchronized 关键字的了解?
	 *
	 * 	1. synchronized 关键字是JVM层次的重量级锁，它可以使线程同步，保本线程的原子性；
	 * 	2. synchronized 是依赖对象的monitor 对象实现的，底层是操作系通的 Mutex Lock
	 * 		实现的，每次调用，操作系统都需要从用户态转为内核态，而这种转变是非常消耗时间的；
	 * 	3. 不过在 Java6 以后对 synchronized 进行了大量的优化，如：借助AQS和CAS技术，自
	 * 		旋锁，锁消除，锁粗化，偏向所，轻量级锁等技术，大大减少了synchronized 的对系统
	 * 		资源的开销；
	 *
	 */

	/**
	 *
	 * synchronized 底层：
	 *		1. 当 synchronized 标注在同步代码块上的时候，底层将会使用对象的Monitor 对象
	 *			（每个对象都有一个monitor对象，这个对象中有个计数器）
	 *			- 当要执行代码块的时候，对象进入代码快的是后，将会使用 MonitorEnter 指令；
	 *				该指令视图获得锁，也就是获得Monitor持有权；此时如果monitor对象的计数器是0，
	 *				表示可以获得，获得后将计数器+1；如果不是0，就不能获的锁，此时该线程就处于等待状态；
	 *			- 当代码块执行结束，将会执行MonitorExit 指令释放锁资源，即将monitor对象的计数器-1；
	 *		2. 当 synchronized 标注在方法上时，在系统调用该方法时，首先会查看 ACC_SYNCHRONIZED关键字，
	 *			有ACC_SYNCHRONIZED 关键字，标明这是一个同步方法，即可以获得对应的monitor对象，进而和上面的
	 *			获得和释放锁的方式一致；
	 *
	 *
	 * 总结：
	 * 	- 同步方法和同步代码块底层都是通过monitor来实现同步的。
	 * 	- 两者的区别：同步方式是通过方法中的access_flags中设置ACC_SYNCHRONIZED标志来实现；
	 * 		同步代码块是通过monitorenter和monitorexit来实现
	 * 	- 我们知道了每个对象都与一个monitor相关联。而monitor可以被线程拥有或释放。
	 *
	 *
	 */

	/**
	 * JDK1.6之后，对于synchronized多的优化：
	 *
	 * 锁状态：无锁；偏向锁，轻量级锁，重量级锁；
	 *
	 * 偏向锁：
	 * 	 1. 当只有一个线程执行，而没有其他线程竞争的时候，将会使用偏向所，（偏向与第一次执行的线程），
	 * 	 	第一次的时候会进行CAS操作，之后都不需要CAS操作
	 *
	 * 轻量级锁：当锁只有两个线程竞争的时候，获取锁资源的时候将会惊醒CAS操作，当一个线程A持有所，另外一个
	 * 			线程B想要获取锁资源的时候，将会进行CAS操作，如果失败，不会进入阻塞状态，而是进入自旋状态
	 * 			（自旋锁）；（自旋超时，也会进入阻塞状态）
	 * 重量级：也就是每个没有获取到锁的线程都会进入阻塞状态，（唤醒阻塞状态的线程需要操作系统从用户态转为内核态）
	 * 		这时候也就是使用的是Synchronized锁操作机制；
	 *
	 *
	 */

	/**
	 * synchronized 关键字的使用方法：
	 *  一、修饰实例方法：相当于给当前实例加锁，进入方法之前，首先要或的当前对象的锁；如下
	 */
	public synchronized void sy1Test(int num) {
		int f = num;
		new Thread(() -> {
			// TODO:
			out.println("synchronizedTest.sy1Test: " + f);
			try { SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
		}, "synchronizedTest").start();
	}

	public static void main0(String[] args) {
		int num = 100;
		synchronizedTest sy = new synchronizedTest();
		while (num-- > 0) {
			sy.sy1Test(num);
		}
	}


	/**
	 * 二，修饰静态方法：相当于给当前类加锁，所以此类的实例也都会加锁，即此时相当于类锁；
	 */

	public static synchronized void sy1Test1(int num) {
		int f = num;
		new Thread(() -> {
			// TODO:
			out.println("synchronizedTest.sy1Test: " + f);
			try { SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
		}, "synchronizedTest").start();
	}

	public static void main1(String[] args) {
		int num = 100;
		while (num-- > 0) {
			synchronizedTest.sy1Test1(num);
		}
	}

	/**
	 * 三、修饰同步代码块：指定锁对象，执行代码快之前，要先获得此实例锁；
	 */

	public void sy1Test2(int num) {
		int f = num;
		synchronized (this) {
			new Thread(() -> {
				// TODO:
				out.println("synchronizedTest.sy1Test: " + f);
				try { SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
			}, "synchronizedTest").start();
		}
	}

	public static void main3(String[] args) {
		int num = 100;
		while (num-- > 0) {
			new synchronizedTest().sy1Test2(num);
		}
	}


	public static void main(String[] args) {
		int i = 20;
		while (i-- > 0) {
			int finalI = i;
			new Thread(() -> {
				// TODO:
				HungrySingleton lazySingleton = getInstance();
				lazySingleton.name = " " + finalI;
				synchronized (lazySingleton) {
					out.println("lazySingleton.name = " + lazySingleton.name);
					out.println("lazySingleton = " + lazySingleton);
				}
			}, "synchronizedTest-" + i).start();
		}

	}

}

	/**
	 * 总结：synchronized关键字：
	 *   加在 static 方法上和 synchronized（XXX.class） 都是相当于类锁；
	 *   加载普通方法上和 synchronized（this| object）都是对象锁；
	 *
	 *   即： synchronized 加载不同的方法上和使用不同的对象，会产生不同的锁；
	 *   > 应为 static 和 XXX.Class 都是一个类独有的，所有该类对象共有的，所以会是类锁；
	 *   > 而 普通方法和 实例对象（this）都是对象的，因对象而不同，因此是对象锁；
	 */


	/**
	 * volatile :
	 *
	 * - 可见性 ：当一个变量对共享变量进行了修改，那么另外的线程都是立即可以看到修改后的最新值。
	 * 	volatile 关键字可以保证共享变量的可见性。
	 * - 有序性 ：代码在执行的过程中的先后顺序，Java 在编译器以及运行期间的优化，
	 * 	代码的执行顺序未必就是编写代码时候的顺序。volatile 关键字可以禁止指令进行重排序优化。
	 */


	/**
	 *  单例模式：
	 *
	 *  DCL 双重检测锁
	 *
	 *  1. 构造方法私有化
	 *  2. 提供外接获取单例对象的方法
	 *  3. 创建单例对象
	 *
	 * > 注： 懒汉模式需要加上 volatile 关键字，防止指令重排序，
	 * >	原因：singleton = new LazySingleton() 分为三步：
	 * 			1、为singleton对象分配内存空间；
	 * 			2、初始化singleton 对象；
	 * 			3、将 singleton 指向初始化好的内存空间；
	 * 		加上volatile 关键字后可阻止上三步发生指令重排序；
	 *
	 *  懒汉式
	 *  饿汉式
	 */


	/**
	 * 懒汉式
	 */
	class LazySingleton {
		private LazySingleton() {
		}

		public String name;

		private static volatile LazySingleton singleton;

		public static LazySingleton getInstance() {
			if (singleton == null) {
				synchronized (LazySingleton.class) {
					if (singleton == null) {
						singleton = new LazySingleton();
					}
				}
			}
			return singleton;
		}

	}


	/**
	 * 饿汉式
	 */
	class HungrySingleton {

		public volatile String name;

		private HungrySingleton() {
		}

		private static HungrySingleton singleton = new HungrySingleton();

		public static HungrySingleton getInstance() {
			return singleton;
		}

	}


```
