---
title: JAVA多线程并发
top: false
cover: true
author: 张文军
date: 2020-04-30 01:18:28
tags: JAVA多线程并发
category: 并发
summary: JAVA多线程并发
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----

# JAVA多线程并发

![JAVA 并发知识库](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588180884.png)

## 进程和线程的区别

**进程是资源分配的最小单位，线程是CPU调度的最小单位。**

### Java进程和线程的关系

- Java对操作系统提供的功能进行封装，包括进程和线程
- 运行一个程序会产生一个进程，进程包含至少—个线程
- 每个进程对应一个JVM 实例，多个线程共享JVM里的堆
- Java采用单线程编程模型，程序会自动创建主线程
- 主线程可以创建子线程，原则上要后于子线程完成执行

### 进程和线程联系

① 线程是进程的最小执行和分配单元，不能独立运动，必须依赖于进程，这也就可以说众多的线程组成了进程。

② 资源分配给进程，同一进程的所有线程共享该进程的所有资源。

### 进程和线程区别

① 调度：线程作为调度和分配的基本单位，进程作为拥有资源的基本单位

② 并发性：不仅进程之间可以并发执行，同一个进程的多个线程之间也可并发执行

③ 拥有资源：进程是拥有资源的一个独立单位，线程不拥有系统资源，但可以访问隶属于进程的资源.

④ 系统开销：在创建或撤消进程时，由于系统都要为之分配和回收资源，导致系统的开销明显大于创建或撤消线程时的开销。

## Thread中的star和run方法的区别

- 调用 start（）方法会创建一个新的子线程并启动
- run（）方法只是 Thread的一个普通方法的调用

 >启动线程的唯一方法就是通过 Thread 类的 start()实例方法。 start()方法是一个 native 方法，它将启动一个新线程，并执行 run()方法。

## Thread和 Runnable是什么关系

- Thread是实现了 Runnable接口的类，使得run支持多线程
- 因类的单一继承原则，推荐多使用 Runnable接口

## 如何实现处理线程的返回值

实现的方式主要有三种:

- 主线程等待法(在主线程中写一个等待机制，如while()循环，直到要返回的结果不为空为止，即子线程执行完毕为止)
- 使用 Thread类的join()阻塞当前线程以等待子线程处理完毕
- 通过 Callable接口实现：通过 FutureTask or 线程池获取
  >有返回值的任务必须实现 Callable 接口，通过Callable接口的call()方法将结果返回。
  >将Callable实例传入一个Future对象中，如 FutureTask 对象，在该对象上调用 get 就可以获取到 Callable 任务返回的 Object 了。

## 线程的状态（线程生命周期）

当线程被创建并启动以后，它既不是一启动就进入了执行状态，也不是一直处于执行状态。在线程的生命周期中，它要经过新建(New)、就绪（Runnable）、运行（Running）、阻塞(Blocked)和死亡(Dead)5 种状态。尤其是当线程启动以后，它不可能一直"霸占"着 CPU 独自运行，所以 CPU 需要在多条线程之间切换，于是线程状态也会多次在运行、阻塞之间切换。

![线程的状态（线程生命周期）](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588288448.png)

>1. 新建状态（NEW）:
>使用 new 关键字和 Thread 类或其子类建立一个线程对象后，该线程对象就处于新建状态,此时仅由 JVM 为其分配内存，并初始化其成员变量的值。它保持这个状态直到程序 start() 这个线程。
>
>2. 就绪状态（RUNNABLE）：
>当线程对象调用了start()方法之后，该线程就进入就绪状态。Java 虚拟机会为其创建方法调用栈和
程序计数器,就绪状态的线程处于就绪队列中，要等待JVM里线程调度器的调度。
>
>3. 运行状态（RUNNING）：
>如果处于就绪状态的线程获得了 CPU，开始执行 run()方法的线程执行体，则该线程处于运行状态。处于运行状态的线程最为复杂，它可以变为阻塞状态、就绪状态和死亡状态。
>
>4. 阻塞状态（BLOCKED）：
>阻塞状态是指线程因为某种原因放弃了 cpu 使用权，也即让出了 cpu timeslice，暂时停止运行。直到线程进入可运行(runnable)状态，才有机会再次获得 cpu timeslice 转到运行(running)状态。阻塞的情况分三种:
>
> >- 等待阻塞（o.wait->等待对列） ：
> >  运行(running)的线程执行 o.wait()方法， JVM 会把该线程放入等待队列(waitting queue)中。
> >- 同步阻塞(lock->锁池)
> >  运行(running)的线程在获取对象的同步锁时，若该同步锁被别的线程占用，则 JVM 会把该线程放入锁池(lock pool)中。
> >- 其他阻塞(sleep/join)
> >  运行(running)的线程执行 Thread.sleep(long ms)或 t.join()方法，或者发出了 I/O 请求时，JVM 会把该线程置为阻塞状态。当 sleep()状态超时、 join()等待线程终止或者超时、或者 I/O处理完毕时，线程重新转入可运行(runnable)状态。
>
>5. 死亡状态（DEAD）:
>一个运行状态的线程完成任务或者其他终止条件发生时，该线程就切换到终止状态
>
> >- 正常结束
> >- 1. run()或 call()方法 或者是 main()方法 执行完成，线程正常结束。
> >- 异常结束
> >- 2. 线程抛出一个未捕获的 Exception 或 Error。
> >- 调用 stop
> >- 3. 直接调用该线程的 stop()方法来结束该线程—该方法通常容易导致死锁，不推荐使用。
> 已经死亡的线程不能在复活，如果在调用死亡的线程，会抛出：`java.long.IllegalThreadStateException`

![线程的状态（线程生命周期）](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588287305.png)

### sleep和wait的区别

1. sleep是 Thread类的方法，wait是 Object类中定义的方法
2. sleep()方法可以在任何地方使用,wait()方法只能在 synchronized方法或 synchronized块中使用。
3. 最主要的本质区别

- Thread.seep只会让出CPU，不会导致锁行为的改变
- Object. wait.不仅让出CPU，还会释放已经占有的同步资源锁（只有有锁，才能释放，所以wait()只能在同步代码快或同步方法中）

### notify和 notify的区别

锁池和等待池的概念：

>锁池 EntryList：
>>假设线程A已经拥有了某个对象（不是类）的锁，而其它线程B、C想要调用这个对象的某个 synchronized方法（或者块），由于B、C线程在进入对象的 synchronized方法（或者块）之前必须先获得该对象锁的拥有权，而恰巧该对象的锁目前正被线程A所占用，此时B、C线程就会被阻塞，进入一个地方去等待锁的释放，这个地方便是该对象的锁池
>
>等待池 Waitset:
>>当一个对象调用wait()方法无限期等待后，此对象将暂时放弃CPU的使用权和锁对象，并且进入一个地方等待其他线程唤醒，而这个地方就是等待池。

notify和 notify的区别:

- notifyAll会让所有处于等待池的线程全部进入锁池去竞争获取锁的机会
- notify只会随机选取一个处于等待池中的线程进入锁池去竞争获取锁的机会。

### yield

概念:当调用 Thread.yield()函数时，会给线程调度器一个当前线程愿意让出CPU使用的暗示，但是线程调度器可能会忽略这个暗示。(Thread.yield不会对锁有任何影响)

### 如何中断线程

1. 通过调用stop()方法停止线程:
   该方法通常容易导致死锁，不推荐使用。

2. 通过调用 suspend()和 resume()方法

3. 调用 interrupt（），通知线程应该中断了：

   ①如果线程处于被阻塞状态，那么线程将立即退出被阻塞状态，并抛出一个 interrupted Exception异常。
   ②如果线程处于正常活动状态，那么会将该线程的中断标志设置为true。被设置中断标志的线程将继续正常运行，不受影响。

### start 与 run 区别

1. start（） 方法来启动线程，真正实现了多线程运行。这时无需等待 run 方法体代码执行完毕，可以直接继续执行下面的代码。
2. 通过调用 Thread 类的 start()方法来启动一个线程， 这时此线程是处于就绪状态， 并没有运行。
3. 方法 run()称为线程体，它包含了要执行的这个线程的内容，线程就进入了运行状态，开始运行 run 函数当中的代码。 Run 方法运行结束， 此线程终止。然后 CPU 再调度其它线程。

## JAVA 锁

### 乐观锁

乐观锁是一种乐观思想，即认为读多写少，遇到并发写的可能性低，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，采取在写时先读出当前版本号，然后加锁操作（比较跟上一次的版本号，如果一样则更新），如果失败则要重复读-比较-写的操作。
java 中的乐观锁基本都是通过 CAS 操作实现的， CAS 是一种更新的原子操作， 比较当前值跟传入值是否一样，一样则更新，否则失败。

### 悲观锁

悲观锁是就是悲观思想，即认为写多，遇到并发写的可能性高，每次去拿数据的时候都认为别人会修改，所以每次在读写数据的时候都会上锁，这样别人想读写这个数据就会 block 直到拿到锁。
java中的悲观锁就是 Synchronized,AQS框架下的锁则是先尝试 cas乐观锁去获取锁，获取不到，才会转换为悲观锁，如 RetreenLock。

### 自旋锁

许多情况下，共亨数据的锁定状态持续时间较短，切换线程不值得，通过让线程执行忙循环等待锁的释放，不让出CPU(让CPU做无用功)

缺点：若锁被其他线程长时间占用，会带来许多性能上的开销

### 自旋锁时间阈值（1.6 引入了适应性自旋锁）

自旋锁的目的是为了占着 CPU 的资源不释放，等到获取到锁立即进行处理。但是如何去选择自旋的执行时间呢？如果自旋执行时间太长，会有大量的线程处于自旋状态占用 CPU 资源，进而会影响整体系统的性能。因此自旋的周期选的额外重要！

JVM 对于自旋周期的选择， jdk1.5 这个限度是一定的写死的， 在 1.6 引入了`适应性自旋锁`。

### 适应性自旋锁

适应性自旋锁意味着自旋的时间不在是固定的了，而是由**前一次在同一个锁上的自旋时间以及锁的拥有者的状态来决定。**

### 锁优化

#### 锁消除

JIT（just in time:即时编译）编译时，对运行上下文进行扫描，去除不可能存在竞争的锁。
（去除不必要的锁）

#### 锁粗化

通过扩大加锁的范围，避免反复加锁和解锁

## Synchronized 同步锁

synchronized是Java提供的一个并发控制的关键字，作用于对象上。主要有两种用法，分别是同步方法(访问对象和clss对象）和同步代码块（需要加入对象），保证了代码的原子性和可见性以及有序性，但是不会处理重排序以及代码优化的过程，但是在一个线程中执行肯定是有序的，因此是有序的。
synchronized 它可以把任意一个非 NULL 的对象当作锁。 他属于独占式的悲观锁，同时属于可重入锁。

自旋锁的开启
JDK1.6 中-XX:+UseSpinning 开启；
-XX:PreBlockSpin=10 为自旋次数；
JDK1.7 后，去掉此参数，由 jvm 控制；

### Synchronized 作用范围

根据获取的锁的分类：获取对象锁和获取类锁

获取对象锁的两种用法：

1. 同步代码块（ synchronized（this）， synchronized（类实例对象）），锁是小括号()中的实例对象
2. 同步非静态方法（ synchronized method），锁是当前对象的实例对象。

获取类锁的两种用法：

1. 同步代码块（ synchronized（类 class），锁是小括号0中的类对象（Cass对象）。
2. 同步静态方法（ synchronized static method），锁是当前对象的类对象（ Class对象）【当作用于静态方法时，锁住的是 Class实例，又因为 Class的相关数据存储在永久带PermGen（jdk1.8 则是 metaspace），永久带是全局共享的，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程；】。

### 实现 synchronized的基础

synchronized对象锁的（monitor）机制：

在 Java 中，每个对象都会有一个 monitor 对象，监视器：

1. 某一线程占有这个对象的时候，先monitor 的计数器是不是0，如果是0还没有线程占有，这个时候线程占有这个对象，并且对这个对象的monitor+1；如果不为0，表示这个线程已经被其他线程占有，这个线程等待。当线程释放占有权的时候，monitor-1；

2. 同一线程可以对同一对象进行多次加锁，+1，+1

### Synchronized 核心组件

1) WaitSet(等待池)：哪些调用 wait 方法被阻塞的线程被放置在这里；
2) Contention List： 竞争队列，所有请求锁的线程首先被放在这个竞争队列中；
3) EntryList(锁池)： Contention List 中那些有资格成为候选资源的线程被移动到 Entry List 中；
4) OnDeck：任意时刻， 最多只有一个线程正在竞争锁资源，该线程被成为 OnDeck；
5) Owner：当前已经获取到所资源的线程被称为 Owner；
6) !Owner：当前释放锁的线程。

![Synchronized 实现](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588332553.png)

### synchronized 的四种状态（锁的状态）

> 无锁、偏向锁、轻量级锁、重量级锁

锁膨胀(升级)方向：无锁→偏向锁→轻量级锁→重量级锁

#### 重量级锁（Mutex Lock）

Synchronized 是通过对象内部的一个叫做监视器锁（monitor）来实现的。但是监视器锁本质又是依赖于底层的操作系统的 Mutex Lock 来实现的。而操作系统实现线程之间的切换这就需要从用户态转换到核心态，这个成本非常高，状态之间的转换需要相对比较长的时间，这就是为什么Synchronized 效率低的原因。因此， 这种依赖于操作系统 Mutex Lock 所实现的锁我们称之为 “重量级锁” 。 JDK 中对 Synchronized 做的种种优化，其核心都是为了减少这种重量级锁的使用。JDK1.6 以后，为了减少获得锁和释放锁所带来的性能消耗，提高性能，引入了“轻量级锁”和
“偏向锁”。

#### 轻量级锁

>轻量级锁是为了在线程交替执行同步块时提高性能

“轻量级” 是相对于使用操作系统互斥量来实现的传统锁而言的。但是，首先需要强调一点的是，轻量级锁并不是用来代替重量级锁的，它的本意是在没有多线程竞争的前提下，减少传统的重量级锁使用产生的性能消耗。先明白一点，轻量级锁所适应的场景是线程交替执行同步块的情况，如果存在同一时间访问同一锁的情况，就会导致轻量级锁膨胀为重量级锁。
如果没有竞争，轻量级锁使用CAS操作避免了使用互斥量的开销，但如果存在锁竞争，除了互斥量的开销外，还额外发生了CAS操作，因此在有竞争的情况下，轻量级锁比传统的重量级锁更慢。

#### 偏向锁：減少同一线程获取锁的代价

> 大多数情况下，锁不存在多线程竞争，总是由同一线程多次获得
> (只有一个线程执行同步块时进一步提高性能。)

偏向锁的目的:某个线程获得锁之后，消除这个线程锁重入（CAS）的开销，看起来让这个线程得到了偏护。

>引入偏向锁是为了在无多线程竞争的情况下尽量减少不必要的轻量级锁执行路径，因为轻量级锁的获取及释放依赖多次 CAS 原子指令， 而偏向锁只需要在置换 ThreadID 的时候依赖一次 CAS 原子指令（由于一旦出现多线程竞争的情况就必须撤销偏向锁，而频繁的撤销偏向锁操作对性能损耗也很大，所以，不适用于锁竞争比较激烈的多线程场合）。

## synchronized 和 ReentrantLock 的区别

java除了使用关键字synchronized外，还可以使用ReentrantLock实现独占锁的功能。而且ReentrantLock相比synchronized而言功能更加丰富，使用起来更为灵活，也更适合复杂的并发场景。

### 两者的共同点

1. 都是用来协调多线程对共享对象、变量的访问
2. 都是可重入锁，同一线程可以多次获得同一个锁
3. 都保证了可见性和互斥性

### 两者的不同点

1. ReentrantLock 显示的获得、释放锁， synchronized 隐式获得释放锁
2. ReentrantLock 可响应中断、可轮回， synchronized 是不可以响应中断的，为处理锁的不可用性提供了更高的灵活性
3. ReentrantLock 是 API 级别的， synchronized 是 JVM 级别的
4. ReentrantLock 可以实现公平锁
5. ReentrantLock 通过 Condition 可以绑定多个条件
6. 底层实现不一样， synchronized 是同步阻塞，使用的是悲观并发策略， lock 是同步非阻塞，采用的是乐观并发策略
7. Lock 是一个接口，而 synchronized 是 Java 中的关键字， synchronized 是内置的语言实现。
8. synchronized 在发生异常时，会自动释放线程占有的锁，因此不会导致死锁现象发生；而 Lock 在发生异常时，如果没有主动通过 unLock()去释放锁，则很可能造成死锁现象，因此使用 Lock 时需要在 finally 块中释放锁。
9. Lock 可以让等待锁的线程响应中断，而 synchronized 却不行，使用 synchronized 时，等待的线程会一直等待下去，不能够响应中断。
10. 通过 Lock 可以知道有没有成功获取锁，而 synchronized 却无法办到。
11. Lock 可以提高多个线程进行读操作的效率，既就是实现读写锁等。

## Java内存模型JMM

Java内存模型（即 Java Memory Model，简称JMM )本身是一种抽象的概念并不真实存在，它描述的是一组规则或规范，通过这组规范定义了程序中各个变量（包括实例字段，静态字段和构成数组对象的元素）的访问方式。
Java 里面进行多线程通信的主要方式就是共享内存的方式，共享内存主要的关注点有两个：可见
性和有序性原子性。 Java 内存模型（JMM）解决了可见性和有序性的问题，而锁解决了原子性的
问题， 理想情况下我们希望做到“同步”和“互斥”。

![JMM](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588575952.png)

![JVM 内存区域](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588576260.png)

### JMM中的主内存

- 存储Java实例对象（Java堆）
- 包括成员变量、类信息、常量、静态变量等（方法区）

这些属于数据共享的区域，多线程并发操作时会引发线程安全问题。

### MM中的工作内存

- 存储当前方法的所有本地变量信息（虚拟机栈）
- 本地变量对其他线程不可见字节码行号指示器（PC程序计数器）、 Native方法信息（本地方法栈）

这些都属于线程私有数据区域，不存在线程安全问题。

### 主內存与工作内存的数据存储类型以及操作方式归纳

JMM与Java内存区域划分是不同的概念层次：JMM描述的是一组规则，围绕原子性，有序性、可见性展开。相似点：存在共享区域和私有区域

- 方法里的基本数据类型本地变量将直接存储在工作內存的栈帧结构中
- 引用类型的本地变量：引用存储在工作內存中，实例存储在主內存中
- 成员变量、 static变量、类信息均会被存储在主內存中
- 主內存共享的方式是线程各拷贝一份数据到工作内存，操作完成后刷新回主内存

### JMM如何解决可见性问题

![JMM数据可见性](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588577092.png)

volatile：JVM提供的轻量级同步机制:

- 保证被 volatile 修饰的共享变量对所有线程总是可见的
- 禁止指令重排序优化

1. volatile变量为何立即可见？

- 当写一个 volatile变量时，JMM会把该线程对应的工作内存中的共享变量值刷新到主内存中。
- 当读取一个 volatile变量时，JMM会把该线程对应的工作内存置为无效。

2. volatile如何禁止重排优化
  
  > 内存屏障（ Memory Barrier）：
  >
  > 1.保证特定操作的执行顺序
  > 2.保证某些变量的内存可见性

- 通过插入内存屏障指令禁止在内存屏障前后的指令执行重排序优化
- 强制刷出各种CPU的缓存数据，因此任何CPU上的线程都能读取到这些数据的最新版本。

## volatile和 synchronized的区别

1. volatile本质是在告诉八M当前变量在寄存器（工作內存）中的值是不确定的，需要从主存中读取； synchronized则是锁定当前变量，只有当前线程可以访问该变量，其他线程被阻塞住直到该线程完成变量操作为止
2. volatile仅能使用在变量级别； synchronized则可以使用在变量、方法和类级别
3. volatile仅能实现变量的修改可见性，不能保证原子性；而synchronized则可以保证变量修改的可见性和原子性
4. volatile不会造成线程的阻塞； synchronized可能会造成线程的阻塞
5. volatile标记的变量不会被编译器优化； synchronized标记的变量可以被编译器优化

## CAS（比较并交换-乐观锁机制-锁自旋）

CAS（Compare And Swap/Set）比较并交换， CAS 算法的过程是这样：它包含 3 个参数CAS(V,E,N)。 V 表示要更新的变量(内存值)， E 表示预期值(旧的)， N 表示新值。当且仅当 V 值等于 E 值时，才会将 V 的值设为 N，如果 V 值和 E 值不同，则说明已经有其他线程做了更新，则当前线程什么都不做。最后， CAS 返回当前 V 的真实值。

CAS 操作是抱着乐观的态度进行的(乐观锁)，它总是认为自己可以成功完成操作。 当多个线程同时使用 CAS 操作一个变量时，只有一个会胜出，并成功更新，其余均会失败。失败的线程不会被挂起，仅是被告知失败，并且允许再次尝试，当然也允许失败的线程放弃操作。基于这样的原理，CAS 操作即使没有锁，也可以发现其他线程对当前线程的干扰，并进行恰当的处理。

ABA 问题：CAS 会导致“ABA 问题”。 CAS 算法实现一个重要前提需要取出内存中某时刻的数据，而在下时刻比较并替换，那么在这个时间差类会导致数据的变化。
可以通过版本号（version）的方式来解决 ABA 问题，乐观锁每次在执行数据的修改操作时，都会带上一个版本号，一旦版本号和数据的版本号一致就可以执行修改操作并对版本号执行+1 操作，否则就执行失败。因为每次操作的版本号都会随之增加，所以不会出现 ABA 问题，因为版本号只会增加不会减少。

## 原子包 java.util.concurrent.atomic（锁自旋）

JDK1.5 的原子包： java.util.concurrent.atomic 这个包里面提供了一组原子类。其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性， 即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由 JVM 从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。相对于对于 synchronized 这种阻塞算法， CAS 是非阻塞算法的一种常见实现。 由于一般 CPU 切换时间比 CPU 指令集操作更加长， 所以 J.U.C 在性能上有了很大的提升。

## AQS（抽象的队列同步器）

AbstractQueuedSynchronizer 类如其名，抽象的队列式的同步器， AQS 定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的 ReentrantLock/Semaphore/CountDownLatch。

![AQS（抽象的队列同步器）](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588579821.png)

它维护了一个 volatile int state（代表共享资源）和一个 FIFO 线程等待队列（多线程争用资源被阻塞时会进入此队列）。

## Java线程池

线程池做的工作主要是控制运行的线程的数量，处理过程中将任务放入队列，然后在线程创建后启动这些任务，如果线程数量超过了最大数量超出数量的线程排队等候，等其它线程执行完毕，再从队列中取出任务来执行。 他的主要特点为： **线程复用；控制最大并发数；管理线程**。

### 线程复用

每一个 Thread 的类都有一个 start 方法。 当调用 start 启动线程时 Java 虚拟机会调用该类的 run
方法。 那么该类的 run() 方法中就是调用了 Runnable 对象的 run() 方法。 我们可以继承重写
Thread 类，在其 start 方法中添加不断循环调用传递过来的 Runnable 对象。 这就是线程池的实
现原理。 循环方法中不断获取 Runnable 是用 Queue 实现的，在获取下一个 Runnable 之前可以
是阻塞的。（即 线程start后，一直while循环，从ruanble队列中取runalbe,调用run()）

### 线程池的组成

一般的线程池主要分为以下 4 个组成部分：

1. 线程池管理器：用于创建并管理线程池
2. 工作线程：线程池中的线程
3. 任务接口：每个任务必须实现的接口，用于工作线程调度其运行
4. 任务队列：用于存放待处理的任务，提供一种缓冲机制

Java 中的线程池是通过 Executor 框架实现的，该框架中用到了 Executor， Executors，
ExecutorService， ThreadPoolExecutor ， Callable 和 Future、 FutureTask 这几个类

![线程池的组成](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588582069.png)

ThreadPoolExecutor 的构造方法如下：

```java
public ThreadPoolExecutor(int corePoolSize,int maximumPoolSize, long keepAliveTime,
   TimeUnit unit, BlockingQueue<Runnable> workQueue) {
   this(corePoolSize, maximumPoolSize, keepAliveTime, unit, workQueue,
   Executors.defaultThreadFactory(), defaultHandler);
}
```

1. corePoolSize：指定了线程池中的线程数量。
2. maximumPoolSize：指定了线程池中的最大线程数量。
3. keepAliveTime：当前线程池数量超过 corePoolSize 时，多余的空闲线程的存活时间，即多次时间内会被销毁。
4. unit： keepAliveTime 的单位。
5. workQueue：任务队列，被提交但尚未被执行的任务。
6. threadFactory：线程工厂，用于创建线程，一般用默认的即可。
7. handler：拒绝策略，当任务太多来不及处理，如何拒绝任务。

拒绝策略:
>线程池中的线程已经用完了，无法继续为新任务服务，同时，等待队列也已经排满了，再也塞不下新任务了。这时候我们就需要拒绝策略机制合理的处理这个问题。JDK 内置的拒绝策略如下：
>
>1. AbortPolicy ： 直接抛出异常，阻止系统正常运行。
>2. CallerRunsPolicy ：“调用者运行”一种调节机制，该策略既不会抛弃任务，也不会抛出异常，而是将某些任务回退到调用者，从而降低新任务的流量
>3. DiscardOldestPolicy ： 丢弃最老的一个请求，也就是即将被执行的一个任务，并尝试再次提交当前任务。
>4. DiscardPolicy ： 该策略默默地丢弃无法处理的任务，不予任何处理。如果允许任务丢失，这是最好的一种方案。
>
>以上内置拒绝策略均实现了 RejectedExecutionHandler 接口，若以上策略仍无法满足实际需要，完全可以自己扩展 RejectedExecutionHandler 接口

### Java 线程池工作过程

1. 线程池刚创建时，里面没有一个线程。任务队列是作为参数传进来的。不过，就算队列里面有任务，线程池也不会马上执行它们。
2. 当调用 execute() 方法添加一个任务时，线程池会做如下判断：
a) 如果正在运行的线程数量小于 corePoolSize，那么马上创建线程运行这个任务；
b) 如果正在运行的线程数量大于或等于 corePoolSize，那么将这个任务放入队列；
c) 如果这时候队列满了，而且正在运行的线程数量小于 maximumPoolSize，那么还是要创建非核心线程立刻运行这个任务；
d) 如果队列满了，而且正在运行的线程数量大于或等于 maximumPoolSize，那么线程池会抛出异常 RejectExecutionException。
3. 当一个线程完成任务时，它会从队列中取下一个任务来执行。
4. 当一个线程无事可做，超过一定的时间（keepAliveTime）时，线程池会判断，如果当前运行的线程数大于 corePoolSize，那么这个线程就被停掉。所以线程池的所有任务完成后，它最终会收缩到 corePoolSize 的大小。

![线程池工作过程](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588583529.png)

### 线程池架构

 Java中的线程池是通过Executor框架实现的，该框架中用到了Executor，Executors，ExecutorService，ThreadPoolExecutor这几个类

 ![线程池架构](https://img-blog.csdnimg.cn/20191022212747970.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzQxMDAxMDcx,size_16,color_FFFFFF,t_70)

### 常见创建线程池方式

- Executors.newCachedThreadPool()：无限线程池。

  ```java
      public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                      60L, TimeUnit.SECONDS,
                                      new SynchronousQueue<Runnable>());
      }
  ```

- Executors.newFixedThreadPool(nThreads)：创建固定大小的线程池。

  ```java
      public static ExecutorService newFixedThreadPool(int nThreads) {
        return new ThreadPoolExecutor(nThreads, nThreads,
                                      0L, TimeUnit.MILLISECONDS,
                                      new LinkedBlockingQueue<Runnable>());
      }
  ```

- Executors.newSingleThreadExecutor()：创建单个线程的线程池。

  ```java
      public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
      }
  ```

### 线程池7大参数

```java
/**7大参数*/
    public ThreadPoolExecutor(int corePoolSize,//1、corePoolSize：线程池中的常驻核心线程数
                              int maximumPoolSize,//2、maximumPoolSize：线程池中能够容纳同时执行的最大线程数，此值必须大于等于1
                              long keepAliveTime,//3、keepAliveTime：多余的空闲线程的存活时间当前池中线程数量超过corePoolSize时，当空闲时
                              //间达到keepAliveTime时，多余线程会被销毁直到只剩下corePoolSize个线程为止
                              TimeUnit unit,//4、unit：keepAliveTime的单位 
                              BlockingQueue<Runnable> workQueue,//5、workQueue：任务队列，被提交但尚未被执行的任务
                              ThreadFactory threadFactory,//6、threadFactory：表示生成线程池中工作线程的线程工厂，用于创建线程，一般默认的即可
                              RejectedExecutionHandler handler//7、handler：拒绝策略，表示当队列满了，并且工作线程大于
                              //等于线程池的最大线程数（maximumPoolSize）时如何来拒绝请求执行的runnable的策略
                              ) {
        if (corePoolSize < 0 ||
            maximumPoolSize <= 0 ||
            maximumPoolSize < corePoolSize ||
            keepAliveTime < 0)
            throw new IllegalArgumentException();
        if (workQueue == null || threadFactory == null || handler == null)
            throw new NullPointerException();
        this.acc = System.getSecurityManager() == null ?
                null :
                AccessController.getContext();
        this.corePoolSize = corePoolSize;
        this.maximumPoolSize = maximumPoolSize;
        this.workQueue = workQueue;
        this.keepAliveTime = unit.toNanos(keepAliveTime);
        this.threadFactory = threadFactory;
        this.handler = handler;
    }
```

### 几种常见的阻塞队列

1. **ArrayBlockingQueue**：由数组结构组成的有界阻塞队列。
2. **LinkedBlockingQueue**：由链表结构组成的有界（但大小默认值为**integer.MAX_VALUE**）阻塞队列。
3. **SynchronousQueue**：不存储元素的阻塞队列，也即单个元素的队列。

![阻塞队列家族](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588584436.png)

### 线程池5状态

```java
    private static final int RUNNING    = -1 << COUNT_BITS;
    private static final int SHUTDOWN   =  0 << COUNT_BITS;
    private static final int STOP       =  1 << COUNT_BITS;
    private static final int TIDYING    =  2 << COUNT_BITS;
    private static final int TERMINATED =  3 << COUNT_BITS;

```

![线程池状态](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1588587115.png)

### 执行任务 executer()

```java
public void execute(Runnable command) {
        if (command == null)
            throw new NullPointerException();
        int c = ctl.get();//获取当前线程池的状态 
        if (workerCountOf(c) < corePoolSize) {//当前线程数量小于 coreSize 时创建一个新的线程运行
            if (addWorker(command, true))
                return;
            c = ctl.get();
        }
        if (isRunning(c) && workQueue.offer(command)) {//如果当前线程处于运行状态，并且写入阻塞队列成功
            int recheck = ctl.get();
            if (! isRunning(recheck) && remove(command))  //双重检查，再次获取线程状态；如果线程状态变了（非运行状态）就需要从阻塞队列移除任务，并尝试判断线程是否全部执行完毕。同时执行拒绝策略。
                reject(command);
            else if (workerCountOf(recheck) == 0)  //如果当前线程池为空就新创建一个线程并执行。
                addWorker(null, false);
        }
        else if (!addWorker(command, false)) //如果在第三步的判断为非运行状态，尝试新建线程，如果失败则执行拒绝策略
            reject(command);
}

```

### 自定义线程池

实现demo：

```java

class MyThreadPoolDemo {

    public static void main(String[] args) {
        ExecutorService threadPool = new ThreadPoolExecutor(
                2,
                5,
                2L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<Runnable>(3),
                Executors.defaultThreadFactory(),
                //new ThreadPoolExecutor.AbortPolicy()
                //new ThreadPoolExecutor.CallerRunsPolicy()
                //new ThreadPoolExecutor.DiscardOldestPolicy()
                new ThreadPoolExecutor.DiscardOldestPolicy()
        );
        //10个顾客请求
        try {
            for (int i = 1; i <= 10; i++) {
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "\t 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }

    }

    private static void threadPool() {
        //List list = new ArrayList();
        //List list = Arrays.asList("a","b");
        //固定数的线程池，一池五线程

//       ExecutorService threadPool =  Executors.newFixedThreadPool(5); //一个银行网点，5个受理业务的窗口
//       ExecutorService threadPool =  Executors.newSingleThreadExecutor(); //一个银行网点，1个受理业务的窗口
        ExecutorService threadPool = Executors.newCachedThreadPool(); //一个银行网点，可扩展受理业务的窗口

        //10个顾客请求
        try {
            for (int i = 1; i <= 10; i++) {
                threadPool.execute(() -> {
                    System.out.println(Thread.currentThread().getName() + "\t 办理业务");
                });
            }
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            threadPool.shutdown();
        }
    }
}

```

### 合理配置线程池你是如何考虑的?

1. CPU密集型：线程数=按照核数或者核数+1设定
2. I/O密集型：线程数=CP∪核数*（1+平均等待时间/平均工作时间）

## 死锁编程及定位分析

Java发生死锁的根本原因是：在申请锁时发生了交叉闭环申请。即线程在获得了锁A并且没有释放的情况下去申请锁B，这时，另一个线程已经获得了锁B，在释放锁B之前又要先获得锁A，因此闭环发生，陷入死锁循环。

一旦出现死锁，整个程序既不会发生任何错误，也不会给出任何提示，只是所有线程处于阻塞状态，无法继续。java虚拟机没有提供检测，也没有采取任何措施来处理死锁的情况，所以多线程编程中，必须手动应该采取措施避免死锁。

### 死锁产生的四个必要条件

1. 互斥使用，即当资源被一个线程使用(占有)时，别的线程不能使用
2. 不可抢占，资源请求者不能强制从资源占有者手中夺取资源，资源只能由资源占用者主动释放
3. 请求和保持，即当资源的请求者在请求其他的资源的同时保持对原有资源的占有
4. 循环等待，即存在一个等待队列: P1占有P2的资源，P2占有P3的资源，P3占有P1的资源。这样就形成了一个等待环路。

当上述四个条件都成立的时候，便形成死锁。当然，死锁的情况下如果打破上述任何一个条件，便可让死锁消失。

### 死锁的预防

一般的，解决死锁的方法分为死锁的预防，避免，检测与恢复三种。
死锁的预防是保证系统不进入死锁状态的一种策略。它的基本思想就是要求进程申请资源时遵循某种协议，从而打破产生死锁的四个必要条件中的一个或者多个，保证系统不会进入死锁状态。
     1）打破互斥条件。即为允许进程同时访问某些资源。
     2）打破不可抢占条件。即为允许进程强行从占有者那里夺取某些资源。
     3）打破占有且申请条件。即为可以实行资源预分配策略。
        缺点：
        1，在许多情况下，进程在执行之前不可能知道它所需要的全部资源。
        2，资源利用率低。
        3，降低了进程的并发性。
   4）打破循环等待条件，实行资源有序分配策略。
     缺点：
      1，限制了进程对资源的请求
      2，为了遵循按编号申请的次序，暂不使用的资源也需要提前申请，从而增加了进行对资源的占用时间。

### 死锁的避免

对进程发出的每一个申请资源命令加以动态的检查，并根据检查结果决定是否进行资源分配。就是说，在资源分配的过程中预测有发生死锁的可能性，则加以避免。这种方法的关键是确定资源分配的安全性。
       1，安全序列：
       所谓系统是安全的，是指系统中的所有进程能够按照某一种次序分配资源，并且依次的运行完毕，这种进程序列{P1,P2,..,Pn}就是安全序
列。安全序列是这样组成的：若对于每一个进程Pi,它需要的附加资源可以被系统中当前可用资源加上所有进程Pj当前占有资源之和所满足，则{P1,P2,...,Pn}为一个
安全序列，这时系统处于安全状态，不会进入死锁状态。
      2，银行家算法：
      银行家算法就是从当前状态出发，逐个按安全序列检查各客户谁能完成其工作，然后嘉定其完成工作且归还全部贷款，再检查下一个能完>成工作的客户，....。如果所有客户都能完成工作，则找到一个安全序列，银行家才安全

### 死锁的检测与恢复

死锁检测与恢复是指系统设有专门机构，当死锁发生时，该机构能够检测到死锁发生的位置和原因，并能通过外力破发生的必要条件，从而>使得进程从死锁的状态中恢复出来。

