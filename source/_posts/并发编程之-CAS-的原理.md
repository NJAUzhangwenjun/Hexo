---
title: 并发编程之 CAS 和 AQS 的原理
date: 2020-01-30 17:39:27
tags: 6、CAS 和 AQS 的原理
category: 并发
summary: 并发编程之 CAS 和 AQS 的原理
top: true
cover: true
author: 张文军
---

![](/images/favicon.png)

# 并发编程之 CAS 的原理

## 一、什么是CAS：

>​	CAS（compareAndSwap），中文叫比较交换，一种无锁原子算法。过程是这样：它包含 3 个参数 CAS（V，E，N），V表示要更新变量的值，E表示预期值，N表示新值。仅当 V值等于E值时，才会将V的值设为N，如果V值和E值不同，则说明已经有其他线程做两个更新，则当前线程则什么都不做。最后，CAS 返回当前V的真实值。CAS 操作时抱着乐观的态度进行的，它总是认为自己可以成功完成操作。
>
>​	CAS的全称为Compare And Swap，直译就是比较交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，其实现方式是基于硬件平台的汇编指令，在intel的CPU中，使用的是`cmpxchg`指令，就是说CAS是靠硬件实现的，从而在硬件层面提升效率。
>
>​	
>
>​	当多个线程同时使用CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会挂起，仅是被告知失败，并且允许再次尝试，当然也允许实现的线程放弃操作。基于这样的原理，CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰。
>
>​	与锁相比，使用CAS会使程序看起来更加复杂一些，但由于其非阻塞的，它对死锁问题天生免疫，并且，线程间的相互影响也非常小。更为重要的是，使用无锁的方式完全没有锁竞争带来的系统开销，也没有线程间频繁调度带来的开销，因此，他要比基于锁的方式拥有更优越的性能。
>
>
>
>​	简单的说，CAS  需要你额外给出一个期望值，也就是你认为这个变量现在应该是什么样子的。如果变量不是你想象的那样，哪说明它已经被别人修改过了。你就需要重新读取，再次尝试修改就好了。



## 二、CAS底层原理

### 1、
CAS 要归功于硬件指令集的发展，实际上，我们可以使用同步将这两个操作变成原子的，但是这么做就没有意义了。所以我们只能靠硬件来完成，硬件保证一个从语义上看起来需要多次操作的行为只通过一条处理器指令就能完成。

### 2、这类指令常用的有： 

  >     1.  测试并设置（Tetst-and-Set） 
  >     2. 获取并增加（Fetch-and-Increment） 
  >     3. 交换（Swap） 
  >     4. 比较并交换（Compare-and-Swap） 
  >     5. 加载链接/条件存储（Load-Linked/Store-Conditional）

### 3、CPU 实现原子指令有2种方式：

1. 通过总线锁定来保证原子性。

>总线锁定其实就是处理器使用了总线锁，所谓总线锁就是使用处理器提供的一个 LOCK# 信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住，那么该处理器可以独占共享内存。但是该方法成本太大。因此有了下面的方式。
>

2、通过缓存锁定来保证原子性。

>所谓 缓存锁定 是指内存区域如果被缓存在处理器的缓存行中，并且在Lock 操作期间被锁定，那么当他执行锁操作写回到内存时，处理器不在总线上声言 LOCK# 信号，同时修改内部的内存地址，并允许他的缓存一致性机制来保证操作的原子性，因为缓存一致性机制会阻止同时修改两个以上处理器缓存的内存区域数据（这里和 volatile 的可见性原理相同），当其他处理器回写已被锁定的缓存行的数据时，会使缓存行无效。

注意：有两种情况下处理器不会使用缓存锁定：

> 1.  当操作的数据不能被缓存在处理器内部，或操作的数据跨多个缓存行时，则处理器会调用总线锁定。 
>
> 2.  有些处理器不支持缓存锁定，对于 Intel 486 和 Pentium 处理器，就是锁定的内存区域在处理器的缓存行也会调用总线锁定。

### 3、CAS 底层概述：

在JAVA中，sun.misc.Unsafe 类提供了硬件级别的原子操作来实现这个CAS。 java.util.concurrent 包下的大量类都使用了这个 Unsafe.java 类的CAS操作。Unsafe 是 CAS的核心类，Java无法直接访问底层操作系统，而是通过本地（native）方法来访问。不过尽管如此，JVM还是开了一个后门：Unsafe，它提供了硬件级别的原子操作。

### 4、问题

​	CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，但是在多处理器上实现就有点儿复杂了 。

1. 缓存加锁：其实针对于多个 CPU 情况我们只需要保证在同一时刻对某个内存地址的操作是原子性的即可。缓存加锁就是缓存在内存区域的数据如果在加锁期间，当它执行锁操作写回内存时，处理器不在输出LOCK#信号，而是修改内部的内存地址，利用缓存一致性协议来保证原子性。缓存一致性机制可以保证同一个内存区域的数据仅能被一个处理器修改，也就是说当CPU1修改缓存行中的i时使用缓存锁定，那么CPU2就不能同时缓存了i的缓存行

2. CAS缺点：

   CAS虽然高效地解决了原子操作，但是还是存在一些缺陷的，主要表现在三个方法：循环时间太长、只能保证一个共享变量原子操作、ABA问题。

   - 循环时间太长

     如果CAS一直不成功呢？这种情况绝对有可能发生，如果自旋CAS长时间地不成功，则会给CPU带来非常大的开销。在JUC中有些地方就限制了CAS自旋的次数，例如BlockingQueue的SynchronousQueue。

   - 只能保证一个共享变量原子操作

     看了CAS的实现就知道这只能针对一个共享变量，如果是多个共享变量就只能使用锁了，当然如果你有办法把多个变量整成一个变量，利用CAS也不错。例如读写锁中state的高地位

- **ABA**问题

  CAS需要检查操作值有没有发生改变，如果没有发生改变则更新。但是存在这样一种情况：如果一个值原来是A，变成了B，然后又变成了A，那么在CAS检查的时候会发现没有改变，但是实质上它已经发生了改变，这就是所谓的ABA问题。对于ABA问题其解决方案是加上版本号，即在每个变量都加上一个版本号，每次改变时加1，即A —> B —> A，变成1A —> 2B —> 3A。(可通过添加版本号来解决，如 AtomicStampedReference类)

## 三、**CAS**算法在JDK中的应用

​	在原子类变量中，如 *java.util.concurrent.atomic* 中的 **AtomicXXX** ，都使用了这些底层的**JVM**支持为数字类型的引用类型提供一种高效的CAS操作，而在 *java.util.concurrent* 中的大多数类在实现时都直接或间接的使用了这些原子变量类。

# AQS (AbstractQueuedSynchronizer)

## 一、什么是AQS

AQS(AbstractQueuedSynchronizer) :同步发生器 ，用来构建Lock。

​	AQS是JDK下提供的一套用于实现基于FIFO等待队列的阻塞锁和相关的同步器的一个同步框架。这个抽象类被设计为作为一些可用原子int值来表示状态的同步器的基类。（FIFO：先进先出）

> JDK :
>
> 为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器（信号量、事件，等等）提供一个框架。此类的设计目标是成为依靠单个原子  `int`  值来表示状态的大多数同步器的一个有用基础。子类必须定义更改此状态的受保护方法，并定义哪种状态对于此对象意味着被获取或被释放。假定这些条件之后，此类中的其他方法就可以实现所有排队和阻塞机制。子类可以维护其他状态字段，但只是为了获得同步而只追踪使用  [`getState()`](../../../../java/util/concurrent/locks/AbstractQueuedSynchronizer.html#getState())、[`setState(int)`](../../../../java/util/concurrent/locks/AbstractQueuedSynchronizer.html#setState(int))  和 [`compareAndSetState(int,  int)`](../../../../java/util/concurrent/locks/AbstractQueuedSynchronizer.html#compareAndSetState(int, int)) 方法来操作以原子方式更新的 `int` 值。  
>
> 应该将子类定义为非公共内部帮助器类，可用它们来实现其封闭类的同步属性。类 `AbstractQueuedSynchronizer`  没有实现任何同步接口。而是定义了诸如 [`acquireInterruptibly(int)`](../../../../java/util/concurrent/locks/AbstractQueuedSynchronizer.html#acquireInterruptibly(int))  之类的一些方法，在适当的时候可以通过具体的锁和相关同步器来调用它们，以实现其公共方法。

即：AQS 非公共内部帮助器类，可用它们来实现其封闭类的同步属性,提供了获取锁和释放锁的功能，可以说他是一个模板、我们可以根据这个模板去定义我们自己的锁(私有内部类继承AQS )。

### 一、CLH同步队列

![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B-CAS-%E7%9A%84%E5%8E%9F%E7%90%86/20200201021840832.png)


CLH ()同步队列是一个FIFO双向队列，AQS 依赖它来完成同步状态的管理，当前线程如果获取同步状态失败时，AQS则会将当前线程已经等待状态等信息构造成一个节点（Node）并将其加入到CLH同步队列，同时会阻塞当前线程，当同步状态释放时，会把首节点唤醒（公平锁），使其再次尝试获取同步状态。

在CLH(Craig, Landin, Hagersten  三个人名) 同步队列中，一个节点表示一个线程，它保存着线程的引用（thread）、状态（waitStatus）、前驱节点（prev）、后继节点（next）。
![](http://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/%E5%B9%B6%E5%8F%91%E7%BC%96%E7%A8%8B%E4%B9%8B-CAS-%E7%9A%84%E5%8E%9F%E7%90%86/20200201023459640.png)

```java
static final class Node {
        /** 共享模式 */
        static final Node SHARED = new Node();
        /** 独占模式 */
        static final Node EXCLUSIVE = null;
 
        /** 因为超时或者是中断，节点会处于删除状态，处于删除状态的节点不会转变成其他状态
         *  ，会一直处于删除状态
         */
        static final int CANCELLED =  1;
        /** 等待状态，用于表示后继节点是否需要被唤醒 */
        static final int SIGNAL    = -1;
        /** 节点再等待队列中，节点的线程将会处于Condition等待状态，只有其他线程调用了Condition
         *  的signal()后，该节点由等待队列进入到同步队列，尝试获取同步状态
         */
        static final int CONDITION = -2;
        /**
         * 表示下一次共享式同步状态获取将会无条件传播下去
         */
        static final int PROPAGATE = -3;
 
        /**
         * 等待状态:
         *   SIGNAL:     表示这个节点的后继节点被阻塞，到时需要通知它。
         *   CANCELLED:  由于中断和超市，这个节点处于被删除的状态。处于被删除状态的节点不会转 
         *               换成其他状态，这个节点将被踢出同步队列，被GC回收。
         *   CONDITION:  表示这个节点再条件队列中，因为等待某个条件而被阻塞。
         *   PROPAGATE:  使用在共享模式头结点有可能牌处于这种状态，表示锁的下一次获取可以无条件 
         *               传播.
         *   0:          新节点会处于这种状态。
         *
         */
        volatile int waitStatus;
 
        /**
         * 队列中，节点的前一个节点
         */
        volatile Node prev;
 
        /**
         * 节点的后继节点.
         */
        volatile Node next;
 
        /**
         * 节点所拥有的线程.
         */
        volatile Thread thread;
 
        /**
         * 条件队列中，节点的下一个等待节点.
         */
        Node nextWaiter;
 
        /**
         * 判断节点时候是共享模式.
         */
        final boolean isShared() {
            return nextWaiter == SHARED;
        }
 
        /**
         * 获取前一个节点
         */
        final Node predecessor() throws NullPointerException {
            Node p = prev;
            if (p == null)
                throw new NullPointerException();
            else
                return p;
        }
 
        Node() {    // Used to establish initial head or SHARED marker
        }
 
        Node(Thread thread, Node mode) {     // Used by addWaiter
            this.nextWaiter = mode;
            this.thread = thread;
        }
 
        Node(Thread thread, int waitStatus) { // Used by Condition
            this.waitStatus = waitStatus;
            this.thread = thread;
        }
    }
```



## 二、AQS 实现

AQS 有两种模式：共享模式，独占模式；

独占模式：

   -  

     `acquire (int arg)`             以独占模式获取对象，忽略中断。

   -   

      `tryAcquire (int arg)`             试图在独占模式下获取对象状态。（是 CLH 同步队列自旋的基础）

-  如果未未那道锁，就会被添加到CLH队列中

     ```java
     //以独占模式获取对象，忽略中断
     public final void acquire(int arg) {
             if (!tryAcquire(arg) &&
                 acquireQueued(addWaiter(Node.EXCLUSIVE), arg))//试图在独占模式下获取对象状态,如果没有获取到，就被添加到CLH队列中
                 selfInterrupt();
     }
     ```

     ```java
     
     	//将未获得锁的线程添加到CHL队列中等待
         private Node addWaiter(Node mode) {
             //新建Node
             Node node = new Node(Thread.currentThread(), mode);
             //快速尝试添加尾节点
             Node pred = tail;
             if (pred != null) {
                 node.prev = pred;
                 //CAS设置尾节点
                 if (compareAndSetTail(pred, node)) {
                     pred.next = node;
                     return node;
                 }
             }
             //多次尝试
             enq(node);
             return node;
         }
     ```

     ```java
     
     //addWaiter(Node node)先通过快速尝试设置尾节点，如果失败，则调用enq(Node node)方法设置尾节点
     
         private Node enq(final Node node) {
             //多次尝试，直到成功为止
             for (;;) {
                 Node t = tail;
                 //tail不存在，设置为首节点
                 if (t == null) {
                     if (compareAndSetHead(new Node()))
                         tail = head;
                 } else {
                     //设置为尾节点
                     node.prev = t;
                     if (compareAndSetTail(t, node)) {
                         t.next = node;
                         return t;
                     }
                 }
             }
         }
     //enq(Node node)方法中，AQS通过“死循环”的方式来保证节点可以正确添加，只有成功添加后，当前线程才会从该方法返回，否则会一直执行下去。
     ```

     

共享模式：

-  

  `acquireShared (int arg)`             以共享模式获取对象，忽略中断。

-   

   `tryAcquireShared (int arg)`             试图在共享模式下获取对象状态

   > 此方法返回值：
   > 在失败时返回负值；如果共享模式下的获取成功但其后续共享模式下的获取不能成功，则返回 
   > 0；如果共享模式下的获取成功并且其后续共享模式下的获取可能够成功，则返回正值，在这种情况下，后续等待线程必须检查可用性。（对三种返回值的支持使得此方法可以在只是有时候以独占方式获取对象的上下文中使用。）在成功的时候，此对象已被获取。

-  
   以共享模式获取锁，如果尝试失败，就将该线程添加到等待队列CLH中

   ```java
       public final void acquireShared(int arg) {
           if (tryAcquireShared(arg) < 0)
               doAcquireShared(arg);
       }
   
   	//---------------------------
   	//添加到CLH中
       private void doAcquireShared(int arg) {
           final Node node = addWaiter(Node.SHARED);
           boolean failed = true;
           try {
               boolean interrupted = false;
               for (;;) {
                   final Node p = node.predecessor();
                   if (p == head) {
                       int r = tryAcquireShared(arg);
                       if (r >= 0) {
                           setHeadAndPropagate(node, r);
                           p.next = null; // help GC
                           if (interrupted)
                               selfInterrupt();
                           failed = false;
                           return;
                       }
                   }
                   if (shouldParkAfterFailedAcquire(p, node) &&
                       parkAndCheckInterrupt())
                       interrupted = true;
               }
           } finally {
               if (failed)
                   cancelAcquire(node);
           }
       }
   ```

释放锁：

- `release(int arg)`               以独占模式释放对象。
- `releaseShared (int arg)`             以共享模式释放对象。

## 三、自定义锁

（实现Lock接口，私有内部类继承 AQS 实现相应方法（如获取锁和释放锁） ）

自定义独占锁：

```java
package cn.njauit;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.AbstractQueuedSynchronizer;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;

/**
 * @author 张文军
 * @Description: 自定义独占锁
 * @Company: njauit.cn
 * @version: 1.0
 * @date 2020/2/13:55
 */
public class MyLock implements Lock {
    /**
     * 帮助器类
     * 可用它们来实现其封闭类的同步属性,提供了获取锁和释放锁的功能，
     * 可以说他是一个模板、我们可以根据这个模板去定义我们自己的锁(私有内部类继承AQS )
     */
    private class Helper extends AbstractQueuedSynchronizer {

        //获取锁
        @Override
        protected boolean tryAcquire(int arg) {

            //获取当前锁所处状态
            int state = getState();
            if (state == 0) {
                //利用CAS原理修改锁的状态
                if (compareAndSetState(0, arg)) {
                    //设置当前线程占有资源
                    setExclusiveOwnerThread(Thread.currentThread());
                    return true;
                }
            } else if (getExclusiveOwnerThread() == Thread.currentThread()) {//可重入性
                setState(getState()+arg);
            }
            return false;
        }

        //释放锁

        @Override
        protected boolean tryRelease(int arg) {
            int state = getState() - arg;

            if (state == 0) {
                setExclusiveOwnerThread(null);
                setState(state);
                return true;
            }
            //释放未成功，重入锁
            setState(state);
            return false;
        }


        //条件锁 ....此处简写了

        private Condition newConditionObject() {
            return new ConditionObject();
        }
    }

    private Helper helper = new Helper();


    @Override
    public void lock() {
        helper.acquire(1);
    }

    @Override
    public void lockInterruptibly() throws InterruptedException {
        helper.acquireInterruptibly(1);
    }

    @Override
    public boolean tryLock() {
        return helper.tryAcquire(1);
    }

    @Override
    public boolean tryLock(long time, TimeUnit unit) throws InterruptedException {
        return helper.tryAcquireNanos(1, unit.toNanos(time));
    }

    @Override
    public void unlock() {
        helper.release(1);
    }

    @Override
    public Condition newCondition() {
        return helper.newConditionObject();
    }
}

```

（重入性：同一锁，同一个资源占有时，直接分配给这个线程）

## 四、[java.util.concurrent.locks](java/util/concurrent/locks/package-frame.html)  常用类

1、ReentrantLock 可重入锁 ：公平锁和非公平所（默认是非公平锁）

2、ReentrantReadWriteLock 读写锁：会将读取者优先或写入者优先强加给锁访问的排序。但是，它确实支持可选的公平 策略。

## 五、并发工具

### 1、CountDownLatch ：倒计时器 （`java.util.concurrent.CountDownLatch`）

  一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。 

- `countDownLatch `这个类使一个线程等待其他线程各自执行完毕后再执行。
- 是通过一个计数器来实现的，计数器的初始值是线程的数量。每当一个线程执行完毕后，计数器的值就-1，当计数器的值为0时，表示所有线程都执行完毕，然后在闭锁上等待的线程就可以恢复工作了。
- 用给定的计数 初始化 CountDownLatch。由于调用了 countDown() 方法，所以在当前计数到达零之前，await 方法会一直受阻塞。之后，会释放所有等待的线程，await 的所有后续调用都将立即返回。
- 用途：
  CountDownLatch 是一个通用同步工具，它有很多用途。将计数 1 初始化的 CountDownLatch 用作一个简单的开/关锁存器，或入口：在通过调用 countDown() 的线程打开入口前，所有调用 await 的线程都一直在入口处等待。用 N 初始化的 CountDownLatch 可以使一个线程在 N 个线程完成某项操作之前一直等待，或者使其在某项操作完成 N 次之前一直等待。 
- CountDownLatch 的一个有用特性是，它不要求调用 countDown 方法的线程等到计数到达零时才继续，而在所有线程都能通过之前，它只是阻止任何线程继续通过一个 await。


### 2. CyclicBarrier ：循环栅栏

- 也是一个同步辅助类，它允许一组线程相互等待，直到到达某个公共屏障点（common barrier point）。
- 通过它可以完成多个线程之间相互等待，只有当每个线程都准备就绪后，才能各自继续往下执行后面的操作。
- 类似于CountDownLatch，它也是通过计数器来实现的。当某个线程调用await方法时，该线程进入等待状态，且计数器加1，当计数器的值达到设置的初始值时，所有因调用await进入等待状态的线程被唤醒，继续执行后续操作。
- 因为CyclicBarrier在释放等待线程后可以重用，所以称为循环barrier。
- CyclicBarrier支持一个可选的Runnable，在计数器的值到达设定值后（但在释放所有线程之前），该Runnable运行一次，注，Runnable在每个屏障点只运行一个。
- CyclicBarrier可以将屏障重置为其初始状态，进而重复使用。

### 3、Semaphore  信号量

- 一个计数信号量。从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。 
- Semaphore 通常用于限制可以访问某些资源（物理或逻辑的）的线程数目



