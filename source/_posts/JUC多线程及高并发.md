---
title: JUC多线程及高并发
date: 2020-02-09 13:36:19
tags: JUC多线程及高并发
category: 并发
summary: JUC多线程及高并发
top: false
cover: true
author: 张文军
---

<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# JUC多线程及高并发

## 1、谈谈你对volatile的理解

- valtile 是JVM提供的一种轻量级的锁；
   - 保证可见性
   - 禁止指令重排
   - 不保证原子性


## 2、JMM关于同步的规定：
  1.线程解锁前，必须把共享变量的值刷新回主内存。
  2.线程加锁前，必须读取主内存的最新值到自己的工作内存。
  3.加锁解锁是同一把锁。

  由于JVM运行程序的实体是线程，而每个线程创建时JVM都会为其创建一个工作内存（有些地方称为栈空间），工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储到主内存，主内存是共享内存区域，所有线程都可以访问，但线程对变量的操作（读取、复制等）必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作内存空间，然后对变量进行操作，操作完成后再将变量写回主内存，不能直接操作主内存中的变量，各个线程中的工作内存中存储着主内存中的变量副本拷贝，因此不同的线程间无法访问对方的工作内存，线程间的通信（传值）必须通过主内存来完成。
  ![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210060456607.png)
  - 2.1可见性

      通过前面对JMM的介绍，我们知道
        各个线程对主内存中共享变量的操作都是各个线程各自拷贝到自己的工作内存进行操作后再写回主内存中的。

        这就可能存在一个线程A修改了共享变量X的值但还未写回主内存时，另一个线程B又对准内存中同一个共享变量X进行操作，但此时A线程工作内存中共享变量X对线程B来说并不是可见，这种工作内存与主内存同步存在延迟现象就造成了可见性问题。

   - 2.2原子性
     number++在多线程下是非线程安全的,如何不加synchronized解决?
     ![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210060910743.png)
   - 2.3VolatileDemo代码演示可见性+原子性
   - 2.4有序性

	  计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排，一般分一下3种：
	  ![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210061009926.png)
	  源代码->编译器优化的重排->指令并行的重排->内存系统的重排->最终执行的指令
	
	  单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。
	
	  处理器在进行重排序时必须考虑指令之间的数据依赖性。
	
	  多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测。


  - 禁止指令重排小总结 ：

    - 1、
    ```java
    public void mySort(){
    int x=11;//语句1
    int y=12;//语句2
    x=x+5;//语句3
    y=x*x;//语句4
    }
    1234
    2134
    1324
    问题:
    请问语句4 可以重排后变成第一条码?
    存在数据的依赖性 没办法排到第一个
    ```
    - 2、

        int a ,b ,x,y=0;
        线程1	线程2
        x=a;	y=b;
        b=1;	a=2;

        x=0 y=0	
         如果编译器对这段代码进行执行重排优化后,可能出现下列情况:
        线程1	线程2
        b=1;	a=2;
        x=a;	y=b;

        x=2 y=1	
         这也就说明在多线程环境下,由于编译器优化重排的存在,两个线程使用的变量能否保持一致是无法确定的.
         ![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210061802089.png)

    

volatile实现禁止指令重排优化，从而避免多线程环境下程序出现乱序执行的现象。
		  
先了解一个概念，内存屏障又称内存栅栏，是一个CPU指令，它的作用有两个：
   一是保证特定操作的执行顺序
   二是保证某些变量的内存可见性（利用该特性实现volatile的内存可见性）
		  
由于编译器和处理器都能执行指令重排优化。如果在指令间插入一条Memory Barrier则告诉编译器和CPU，不管什么指令都不能和这条Memory Barrier指令重新排序，也就是说通过插入内存屏障禁止在内存屏障前后的指令执行重排序优化。内存屏障另外一个作用是强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。

## 3、你在哪些地方用过volatile？


- 3.1单例模式DCL代码
- 3.2单例模式volatile分析

DCL（双端检锁）机制不一定线程安全，原因是有指令重排序的存在，加入volatile可以禁止指令重排。
	  
原因在于某一个线程执行到第一个检测，读取到的instance不为null时，instance的引用对象可能没有完成初始化。
	  
指令重排只会保证串行语义的执行一致性（单线程），但并不会关心多线程间的语义一致性。
	  
所以当一条线程访问instance不为null时，由于instance实例未必已初始化完成，也就造成了线程安全问题。
      
懒汉的双重加锁机制 （DCL ：Double-Check-Locking）：
      
```java

    package cn.njauit;

    public class Singleton { 
        /*
        * 利用静态变量来记录Singleton的唯一实例 
        * volatile 关键字确保：当uniqueInstance变量被初始化成Singleton实例时，
        * 多个线程正确地处理uniqueInstance变量(保证了有序性，解决了编译重排重排和运行重排问题)
        * 
        */
        private volatile static Singleton uniqueInstance; 

        /* * 构造器私有化，只有Singleton类内才可以调用构造器 */
        private Singleton() {
        } 

        /* * * 检查实例，如果不存在，就进入同步区域 */
        public static Singleton getInstance() {
            if (uniqueInstance == null) {
                synchronized (Singleton.class) { //进入同步区域
                    if (uniqueInstance == null) { //在检查一次，如果为null，则创建
                        uniqueInstance = new Singleton();
                    }
                }
            }
            return uniqueInstance;
        }
    }

```

## 4、CAS（compare and swap）你知道吗

### 1、CASDemo
```java
/**
 * Description
 *
 * @author veliger@163.com
 * @version 1.0
 * @date 2019-04-12 9:57
 * 1.什么是CAS ? ===> compareAndSet
 *  比较并交换
 **/
public class CASDemo {
    public static void main(String[] args) {
        AtomicInteger atomicInteger = new AtomicInteger(5);
        System.out.println(atomicInteger.compareAndSet(5, 2019)+"\t current"+atomicInteger.get());
        System.out.println(atomicInteger.compareAndSet(5, 2014)+"\t current"+atomicInteger.get());
    }
}
```

### 2、CAS底层原理？如果知道，谈谈你对UnSafe的理解

```java
atomicInteger.getAndIncrement()方法的源代码:

/**
 * Atomically increments by one the current value.
 *
 * @return the previous value
 */
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```
### 3、引出来一个问题:UnSafe类是什么?
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210103008599.png)

1. 是CAS的核心类 由于Java 方法无法直接访问底层 ,需要通过本地(native)方法来访问,UnSafe相当于一个后面,基于该类可以直接操作特额定的内存数据.UnSafe类在于sun.misc包中,其内部方法操作可以向C的指针一样直接操作内存,因为Java中CAS操作的助兴依赖于UNSafe类的方法.
注意UnSafe类中所有的方法都是native修饰的,也就是说UnSafe类中的方法都是直接调用操作底层资源执行响应的任务
2. 变量ValueOffset,便是该变量在内存中的偏移地址,因为UnSafe就是根据内存偏移地址获取数据的
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210103041110.png)
3. 变量value和volatile修饰,保证了多线程之间的可见性

### 4、CAS是什么：

- CAS的全称为Compare-And-Swap ,它*是一条CPU并发原语*。

- 它的功能是判断内存某个位置的值是否为预期值,如果是则更新为新的值,这个过程是原子的.

- CAS并发原语提现在Java语言中就是sun.miscUnSaffe类中的各个方法.调用UnSafe类中的CAS方法,JVM会帮我实现CAS汇编指令.这是一种完全依赖于硬件 功能,通过它实现了原子操作,再次强调,由于CAS是一种系统原语,原语属于操作系统用于范畴,是由若干条指令组成,用于完成某个功能的一个过程,并且原语的执行必须是连续的,在执行过程中不允许中断,也即是说CAS是一条原子指令,不会造成所谓的数据不一致的问题.
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210075112354.png)
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210075615834.png)
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210075705767.png)

- unSafe.getAndIncrement()
    > var1 AtomicInteger对象本身.
    > var2 该对象值的引用地址
    > var4 需要变动的数值
    > var5 是用过var1 var2找出内存中绅士的值
    > 用该对象当前的值与var5比较
    > 如果相同,更新var5的值并且返回true
    > 如果不同,继续取值然后比较,直到更新完成


  ![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210080317840.png)
  假设线程A和线程B两个线程同时执行getAndAddInt操作(分别在不同的CPU上):
 
1.AtomicInteger里面的value原始值为3,即主内存中AtomicInteger的value为3,根据JMM模型,线程A和线程B各自持有一份值为3的value的副本分别到各自的工作内存.
 
2.线程A通过getIntVolatile(var1,var2) 拿到value值3,这是线程A被挂起.
 
3.线程B也通过getIntVolatile(var1,var2) 拿到value值3,此时刚好线程B没有被挂起并执行compareAndSwapInt方法比较内存中的值也是3 成功修改内存的值为4 线程B打完收工 一切OK.
 
 4.这是线程A恢复,执行compareAndSwapInt方法比较,发现自己手里的数值和内存中的数字4不一致,说明该值已经被其他线程抢先一步修改了,那A线程修改失败,只能重新来一遍了.
 
 5.线程A重新获取value值,因为变量value是volatile修饰,所以其他线程对他的修改,线程A总是能够看到,线程A继续执行compareAndSwapInt方法进行比较替换,直到成功.

unSafe底层汇编代码：
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210080745402.png)
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210080818929.png)
### CAS缺点：
- 循环时间长开销大
如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210081122464.png)
- 只能保证一个共享变量的原子性
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210081243704.png)
- 存在ABA问题
CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。(可通过添加版本号来解决，如 AtomicStampedReference类)

- 解决ABA问题
  时间戳原子引用
	- AtomicStampledReference
	![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200210102416475.png)
    
```java

    /**
 * Description: ABA问题的解决
 *
 * @author veliger@163.com
 * @date 2019-04-12 21:30
 **/
public class ABADemo {
    private static AtomicReference<Integer> atomicReference=new AtomicReference<>(100);
    private static AtomicStampedReference<Integer> stampedReference=new AtomicStampedReference<>(100,1);
    public static void main(String[] args) {
        System.out.println("===以下是ABA问题的产生===");
        new Thread(()->{
            atomicReference.compareAndSet(100,101);
            atomicReference.compareAndSet(101,100);
        },"t1").start();

        new Thread(()->{
            //先暂停1秒 保证完成ABA
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            System.out.println(atomicReference.compareAndSet(100, 2019)+"\t"+atomicReference.get());
        },"t2").start();
        try { TimeUnit.SECONDS.sleep(2); } catch (InterruptedException e) { e.printStackTrace(); }
        System.out.println("===以下是ABA问题的解决===");

        new Thread(()->{
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 第1次版本号"+stamp+"\t值是"+stampedReference.getReference());
            //暂停1秒钟t3线程
            try { TimeUnit.SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }

            stampedReference.compareAndSet(100,101,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 第2次版本号"+stampedReference.getStamp()+"\t值是"+stampedReference.getReference());
            stampedReference.compareAndSet(101,100,stampedReference.getStamp(),stampedReference.getStamp()+1);
            System.out.println(Thread.currentThread().getName()+"\t 第3次版本号"+stampedReference.getStamp()+"\t值是"+stampedReference.getReference());
        },"t3").start();

        new Thread(()->{
            int stamp = stampedReference.getStamp();
            System.out.println(Thread.currentThread().getName()+"\t 第1次版本号"+stamp+"\t值是"+stampedReference.getReference());
            //保证线程3完成1次ABA
            try { TimeUnit.SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            boolean result = stampedReference.compareAndSet(100, 2019, stamp, stamp + 1);
            System.out.println(Thread.currentThread().getName()+"\t 修改成功否"+result+"\t最新版本号"+stampedReference.getStamp());
            System.out.println("最新的值\t"+stampedReference.getReference());
        },"t4").start();
    }

```

## 5、ArrayList线程不安全之写时复制
### 1、异常：`java.util.ConcurrentModificationException` 并发修改异常

- 解决方案1：Vector、Collections.synchronizedList
- 限制不可以用Vector和Collections工具类解决方案2

	- new CopyOnWriteArrayList<>()

	  写时复制 CopyOnWrite容器即写时复制的容器。往一个容器添加元素的时候，不直接往当前容器Object[]添加，而是先将当前object[]进行Copy，复制出一个新的容器Object[] newElements，然后新的容器Object[] newElements
里添加元素，添加完元素之后，再将原容器的引用指向新的容setArray（newElements）;这样做的好处是可以对copyonwrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。所以copyonwrite容器也是一种读写分离的思想，读和写不同的容器。

```java
**
 * Description: 集合类不安全的问题
 *
 * @author veliger@163.com
 * @date 2019-04-12 22:15
 **/
public class ContainerNotSafeDemo {
    /**
     * 笔记
     * 写时复制 copyOnWrite 容器即写时复制的容器 往容器添加元素的时候,不直接往当前容器object[]添加,而是先将当前容器object[]进行
     * copy 复制出一个新的object[] newElements 然后向新容器object[] newElements 里面添加元素 添加元素后,
     * 再将原容器的引用指向新的容器 setArray(newElements);
     * 这样的好处是可以对copyOnWrite容器进行并发的读,而不需要加锁 因为当前容器不会添加任何容器.所以copyOnwrite容器也是一种
     * 读写分离的思想,读和写不同的容器.
     *         public boolean add(E e) {
     *         final ReentrantLock lock = this.lock;
     *         lock.lock();
     *         try {
     *             Object[] elements = getArray();
     *             int len = elements.length;
     *             Object[] newElements = Arrays.copyOf(elements, len + 1);
     *             newElements[len] = e;
     *             setArray(newElements);
     *             return true;
     *         } finally {
     *             lock.unlock();
     *         }
     *     }
     * @param args
     */
    public static void main(String[] args) {
        List<String> list= new CopyOnWriteArrayList<>();
        for (int i = 1; i <=30; i++) {
            new Thread(()->{
                list.add(UUID.randomUUID().toString().substring(1,8));
                System.out.println(list);
            },String.valueOf(i)).start();
        }
        /**
         * 1.故障现象
         *  java.util.ConcurrentModificationException
         * 2.导致原因
         *    并发争抢修改导致
         * 3.解决方案
         *  3.1 new Vector<>()
         *  3.2 Collections.synchronizedList(new ArrayList<>());
         *  3.3 new CopyOnWriteArrayList<>();
         *
         *
         * 4.优化建议
         */
    }

 
```

同样其他有：
- List：CopyOnwriteArrayList
- set ：CopyOnWriteHashSet
- map： ConcurrentHashMap


## 6.公平锁/非公平锁/可重入锁/递归锁/自旋锁谈谈你的理解？请手写一个自旋锁

### 1、公平所和非公平锁
- 公平锁和非公平锁

- 是什么

	 - 公平锁：是指多个线程按照申请锁的顺序来获取锁，类似排队打饭，先来后到。 
	 - 非公平锁：是指多个线程获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。在高并发的情况下，有可能会造成优先级反转或者饥饿现象。

- 两者区别:
  公平锁/非公平锁：并发包中ReentrantLock的创建可以指定构造函数的boolean类型来得到公平锁或非公平锁，默认是非公平锁。

  关于两者区别：
  公平锁：Threads acquire a fair lock in the order in which they requested it.
  公平锁，就是很公平，在并发情况下，每个线程在获取锁时会查看此锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则就会加入到等待队列中，以后会按照FIFO的规则从队列中取到自己。

  非公平锁：非公平锁比较粗鲁，上来就直接尝试占有锁，如果尝试失败，就再采取类似公平锁那种方式。

- 题外话

  Java ReentrantLock而言，通过构造函数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大。

  对于Synchronized而言，也是一种非公平锁。

### 2、可重入锁（又名递归锁）

- 是什么可重入锁

  可重入锁（也就是递归锁）：指的是同一个线程外层函数获得锁之后，内层递归函数仍然能获取该锁的代码，在同一线程在外层方法获取锁的时候，在进入内层方法会自动获取锁。

  也就是说，线程可以进入任何一个它已经拥有的锁所有同步着的代码块。

- ReentrantLock/Synchronized就是一个典型的可重入锁
- 可重入锁最大的作用是避免死锁
- ReenterLockDemo

    - 参考1

```java

package cn.njauit;
class Phone{
public synchronized void sendSms() throws Exception{
    System.out.println(Thread.currentThread().getName()+"\tsendSms");
    sendEmail();
}
public synchronized void sendEmail() throws Exception{
    System.out.println(Thread.currentThread().getName()+"\tsendEmail");
}

}
/**
* Description:
*  可重入锁(也叫做递归锁)
*  指的是同一先生外层函数获得锁后,内层敌对函数任然能获取该锁的代码
*  在同一线程外外层方法获取锁的时候,在进入内层方法会自动获取锁
*
*  也就是说,线程可以进入任何一个它已经标记的锁所同步的代码块
*
* @author veliger@163.com
* @date 2019-04-12 23:36
**/
public class ReenterLockDemo {
/**
 * t1 sendSms
 * t1 sendEmail
 * t2 sendSms
 * t2 sendEmail
 * @param args
 */
public static void main(String[] args) {
    Phone phone = new Phone();
    new Thread(()->{
        try {
            phone.sendSms();
        } catch (Exception e) {
            e.printStackTrace();
        }
    },"t1").start();
    new Thread(()->{
        try {
            phone.sendSms();
        } catch (Exception e) {
            e.printStackTrace();
        }
    },"t2").start();
}
}

```
重入性：同一锁，同一个资源占有时，直接分配给这个线程

### 3、自旋锁

  自旋锁：是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，这样的好处是减少线程上下切换的消耗，缺点是循环会消耗CPU。
  - 示例


```java
package cn.njauit;

import java.util.concurrent.atomic.AtomicReference;

import static java.util.concurrent.TimeUnit.SECONDS;

/**
 * @author 张文军
 * @Description: TODO
 * @Company: njauit.cn
 * @version: 1.0
 * @date 2020/2/115:06
 */
public class SpinLock {
    static AtomicReference reference = new AtomicReference<Thread>();

    public static void lock() {
        while (!reference.compareAndSet(null, Thread.currentThread())) {

        }
    }

    public static void unlock() {
        reference.compareAndSet(Thread.currentThread(), null);
    }
    
    /**
     * test
     *
     * @param args
     */
    public static void main(String[] args) {
        new Thread(() -> {
            // TODO:
            lock();
            System.out.println(Thread.currentThread().getName()+"---------- in");
            try { SECONDS.sleep(3); } catch (InterruptedException e) { e.printStackTrace(); }
            unlock();
        }, "SpinLock1").start();

        new Thread(() -> {
            // TODO:
            lock();
            System.out.println(Thread.currentThread().getName()+"---------- in");
            try { SECONDS.sleep(1); } catch (InterruptedException e) { e.printStackTrace(); }
            unlock();
        }, "SpinLock2").start();

    }

}

```

### 4、独占锁/共享锁

  独占锁：指该锁一次只能被一个线程所持有。对ReentrantLock和Synchronized而言都是独占锁。
  
  共享锁：指该锁可被多个线程所持有。
  对ReentrantReadWriteLock，其读锁是共享锁，其写锁是独占锁。读锁的共享锁可保证并发读是非常高效的，读写，写读，写写的过程是互斥的。
示例：

```java
package cn.njauit;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 资源类
 */
class MyCaChe {
    /**
     * 保证可见性
     */
    private volatile Map<String, Object> map = new HashMap<>();
    private ReentrantReadWriteLock reentrantReadWriteLock = new ReentrantReadWriteLock();

    /**
     * 写
     *
     * @param key
     * @param value
     */
    public void put(String key, Object value) {
        reentrantReadWriteLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t正在写入" + key);
            //模拟网络延时
            try {
                TimeUnit.MICROSECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t正在完成");
        } finally {
            reentrantReadWriteLock.writeLock().unlock();
        }
    }

    /**
     * 读
     *
     * @param key
     */
    public void get(String key) {
        reentrantReadWriteLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t正在读取");
            //模拟网络延时
            try {
                TimeUnit.MICROSECONDS.sleep(300);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            Object result = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t正在完成" + result);
        } finally {
            reentrantReadWriteLock.readLock().unlock();
        }
    }

    public void clearCaChe() {
        map.clear();
    }

}

/**
 * Description:
 * 多个线程同时操作 一个资源类没有任何问题 所以为了满足并发量
 * 读取共享资源应该可以同时进行
 * 但是
 * 如果有一个线程想去写共享资源来  就不应该有其他线程可以对资源进行读或写
 * <p>
 * 小总结:
 * 读 读能共存
 * 读 写不能共存
 * 写 写不能共存
 * 写操作 原子+独占 整个过程必须是一个完成的统一整体 中间不允许被分割 被打断
 *
 * @author veliger@163.com
 * @date 2019-04-13 0:45
 **/
public class ReadWriteLockDemo {
    public static void main(String[] args) {
        MyCaChe myCaChe = new MyCaChe();
        for (int i = 1; i <= 5; i++) {
            final int temp = i;
            new Thread(() -> {
                myCaChe.put(temp + "", temp);
            }, String.valueOf(i)).start();
        }
        for (int i = 1; i <= 5; i++) {
            int finalI = i;
            new Thread(() -> {
                myCaChe.get(finalI + "");
            }, String.valueOf(i)).start();
        }
    }
}

```
## 7、小总结 ：synchronized和lock有什么区别？用新的lock有什么好处？你举例说说

  1.原始构成：
  	synchronized是关键字，属于JVM层面，monitorenter（底层是通过monitor对象来完成，其实wait/notify等方法也依赖于monitor对象只有在同步块或者方法中才能调用wait/notify等方法）
  	Lock是具体类（java.util.concurrent.locks.lock）是api层面的锁。
  
  2.使用方法
  synchronized不需要用户去手动释放锁，当synchronized代码执行完后系统会自动让线程释放对锁的占用。
  ReentrantLock则需要用户去手动释放锁，若没有主动释放锁，就有可能导致出现死锁现象。需要lock()和unlock()方法配合try/finally语句块来完成。
  
  3.等待是否可中断
  synchronized不可中断，除非抛出异常或者正常运行完成。
  ReentrantLock可中断，1.设置超时方法 tryLock(long timeout,TimeUnit unit)；2.lockInterruptibly()放代码块中，调用interrupt()方法可中断。
  
  4.加锁是否公平
  synchronized非公平锁
  ReentrantLock两者都可以，默认非公平锁，构造方法可以传入boolean值，true为公平锁，false为非公平锁。
  
  5.锁绑定多个条件Condition
  synchronizedmeiyou
  ReentrantLock用来实现分组唤醒需要唤醒的线程，可以精确唤醒，而不是像synchronized要么随机唤醒一个要么唤醒全部线程。

## 8、CountDownLatch/CyclicBarrier/Semaphore
### 1、CountDownLatch （同步计数器）

- 让一些线程阻塞直到另一个线程完成一系列操作后才被唤醒
- CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，调用线程会被阻塞。其他线程调用countDown方法会将计数器减1（调用CountDown方法的线程不会阻塞），当计数器的值变为0时，因调用await方法被阻塞的线程会被唤醒，继续执行。

示例：

```java
public class CountDownLatchDemo {
    public static void main(String[] args) throws Exception {
        closeDoor();

    }

   /**
     * 关门案例
     * @throws InterruptedException
     */
    private static void closeDoor() throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + "\t" + "上完自习");
                countDownLatch.countDown();
            }, String.valueOf(i)).start();
        }
        countDownLatch.await();
        System.out.println(Thread.currentThread().getName() + "\t班长锁门离开教室");
    }

  
} 
```

### 2、CyclicBarrier（循环栅栏）

- CyslicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会打开门，所有被屏障拦截的线程才会继续干活，线程进入屏障通过CyclicBarrier的await（）方法。

示例：

```java
public class CyclicBarrierDemo {
    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier=new CyclicBarrier(7,()->{
            System.out.println("召唤神龙");
        });

        for (int i = 1; i <=7; i++) {
            final int temp = i;
            new Thread(()->{
             System.out.println(Thread.currentThread().getName()+"\t 收集到第"+ temp +"颗龙珠");
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            },String.valueOf(i)).start();
        }
    }
}
```


### 3、 Semaphore（信号量）

- 信号量主要用于两个目的，一个是用于多个共享资源的互斥使用，另一个用于并发线程数的控制。（如：争车位）

示例：

```java

public class SemaphoreDemo {
    public static void main(String[] args) {
        //模拟3个停车位
        Semaphore semaphore = new Semaphore(3);
        //模拟6部汽车
        for (int i = 1; i <= 6; i++) {
            new Thread(() -> {
                try {
                    //抢到资源
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "\t抢到车位");
                    try {
                        TimeUnit.SECONDS.sleep(3);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + "\t 停3秒离开车位");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    //释放资源
                    semaphore.release();
                }
            }, String.valueOf(i)).start();
        }
    }
}
```


## 9、阻塞队列知道吗

### 1、队列+阻塞队列
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200213024335581.png)
  阻塞队列，首先它是一个队列，而一个阻塞队列在数据结构中所起的作用大致是：线程1往阻塞队列中添加元素，而线程2从阻塞队列中移除元素。
  
  当阻塞队列是空时，从队列中获取元素的操作将被阻塞。
  当阻塞队列是满时，往队列里添加元素的操作将被阻塞。

### 2、为什么用？有什么好处？

  在多线程领域：所谓阻塞，在某些情况下会挂起线程（即阻塞），一旦满足条件，被挂起的线程又会自动被唤醒。
  
  为什么需要BlockingQueue？
  好处是我们不需要关心什么时候需要阻塞线程，什么时候需要唤醒线程，因为这一切BlockingQueue都给你包办了。
  
  在concurrent包发布之前，在多线程环境下，我们每个程序员都必须去自己控制这些细节，尤其还要兼顾效率和线程安全，而这会给程序带来不小的复杂度。

### 3、BlockingQueue的核心方法
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200213024825132.png)


- 抛出异常：当阻塞队列满时,再往队列里面add插入元素会抛IllegalStateException: Queue full ，当阻塞队列空时,再往队列Remove元素时候回抛出NoSuchElementException
- 特殊值：插入方法,成功返回true 失败返回false，移除方法,成功返回元素,队列里面没有就返回null
- 一直阻塞：当阻塞队列满时,生产者继续往队列里面put元素,队列会一直阻塞直到put数据or响应中断退出。当阻塞队列空时,消费者试图从队列take元素,队列会一直阻塞消费者线程直到队列可用.
- 超时退出：当阻塞队列满时,队列会阻塞生产者线程一定时间,超过后限时后生产者线程就会退出

### 4、架构梳理+种类分析

- 架构介绍
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200213025353650.png)
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/JUC%E5%A4%9A%E7%BA%BF%E7%A8%8B%E5%8F%8A%E9%AB%98%E5%B9%B6%E5%8F%91/20200213031012958.png)

  BlockingQueue和list都是Collections的接口。

- 种类分析

    - ArrayBlockingQueue:由数组结构组成的有界阻塞队列
    - LinkedBlockingQueue:由链表结构组成的有界（但大小默认值为Integer.MAX_VALUE）阻塞队列。
    - PriorityBlockingQueue:支持优先级排序的无界阻塞队列。
    - DelayQueue:使用优先级队列实现的延迟无界阻塞队列。
    - SynchronousQueue:不存储元素的阻塞队列，也即单个元素的队列。

        - 理论
          SynchronousQueue没有容量。

          与其他BlockingQueue不同，SynchronousQueue是一个不存储元素的BlockingQueue。
          每一个put操作必须要等待一个take操作，否则不能继续添加元素，反之亦然。

    - LinkedTransferQueue：由链表结构组成的无界阻塞队列。
    - LinkedBlockingDeque：由链表结构组成的双向阻塞队列。

### 5、用在哪里

1、 生产者消费者模式

- 传统版

    - ProdConsumer_TraditionDemo

- 阻塞队列版

    - ProdConsumer_BlockQueueDemo

2、线程池
3、消息中间件











