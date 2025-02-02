# 1. 并发编程简介

## 1. 基础理论

***并发和并行的区别?***

* 并发指一个时间段内多个任务同时进行, 但在任一时刻只有一个任务进行
* 并行指任意时刻有多个任务进行

***并发编程(线程安全)三大问题是什么?***

* 原子性: 确保自增自减等操作的原子性 `(Java中除了对long/double的写操作不是原子的, 其他所有单次读写操作都是原子的)`
* 可见性: 先进行的操作对后进行的操作可见
* 有序性

***什么是死锁, 死锁的四个必要条件是什么?***

死锁: 锁的等待链形成环导致环中每个线程/进程都永久阻塞

四个必要条件:

```
* 锁互斥
* 锁不可剥夺
* 占有锁且等待锁
* 循环等待锁
```

## 2. 线程

***进程, 线程, 协程的区别?***

```
* 进程: 操作系统内存, CPU时间片等资源分配的基本单位
* 线程(Java中的Thread): 多个线程共享给进程分配的资源, 但每个线程执行时拥有独立的栈和CPU核心(CPU核心中寄存器中的内容和栈称为线程上下文)
* 协程(Java中的Fiber): 多个协程共享线程的CPU核心, 协程调度通过应用程序实现
```

***内核级线程和用户级线程的区别?***

```
内核级线程: 操作系统内核中的线程(又称Task), 每个内核级线程执行时会被分配给一个CPU核心的时间片

用户级线程: 应用程序虚拟出来的线程, 实际上多个用户级线程共享一个内核级线程的执行核心

Java线程具体是内核级还是用户级线程取决于JVM, 普遍的JVM采用内核级线程
```

***为什么切换线程比切换进程快, 切换协程比线程快?***

```
进程切换时需要额外切换:  页表(将虚拟地址映射到主存上新进程的物理地址),   切换了页表会造成缓存失效, 缓存命中率降低

切换协程不用进入操作系统内核态, 进入内核态需要保存线程断点且加载操作系统系统调用的上下文, 所以更快
```

***什么情况下使用协程/线程/进程?***

* 协程使用场景: IO密集型的并发任务, 当一个协程正在IO阻塞时, 可以最快速地切换到其他协程并执行
* 线程使用场景: 并发任务之间需要共享资源/进行通信, CPU密集型任务(充分利用多核CPU的并发能力)
* 进程使用场景: 并发任务之间需要资源不共享, 需要保证各个任务的安全性

***线程的五态模型, 状态之间如何转换的?***

```
new ->(start函数) ready ->(被调度) running
 ->(wait等函数, io开始) blocked ->(io结束, notify) running ->  dead
```

***为什么要使用线程池?***

* 使用线程池: 避免频繁创建和销毁线程造成的开销,  便于统一管理线程

## 3. 锁

***乐观锁和悲观锁的区别?***

```
乐观锁读取数据时不会加锁, 写入数据时通过版本号/时间戳判断是否冲突, 冲突的话再进行回滚或放弃

悲观锁读取数据时加互斥锁, 适合频繁发生并发冲突的场景
```

***公平锁和非公平锁的区别?***

```
公平锁: 先阻塞的线程先获得锁

非公平锁: 不论先后, 线程获取锁的机会相同
```

***可重入锁和非可重入锁的区别?***

```
对于可重入锁, 已经获取了锁的线程还可以再获取
非可重入锁相反
```

# 2. Java并发编程

## 1. Thread

***创建一个线程的方式有哪些?***

* 实现Thread的子类,
* 传入runnable对象构造一个Thread

两种方式区别: 一个采用**继承**, 一个采用**组合**, 采用组合的方式灵活性更高

```java
// 组合方式
            Thread helloThread = new Thread(new Runnable() {
                @Override
                public void run() {
                    System.out.println("Hello, world!");
                }
            });
            helloThread.start();

// 继承方式
            class MyThread extends Thread {
                @Override
                public void run() {
                    System.out.println("my thread!");
                }
            }
            new MyThread().start();
```

***线程常用的API有什么?***

```java
            Thread.currentThread();
            Thread.sleep(1000);
            helloThread.setName("hello 线程");
            helloThread.setPriority(Thread.MAX_PRIORITY);
```

***如何监测线程的执行状况?***

使用jdk工具 `jstack <pid>` 下载线程快照文件, 使用可视化的工具进行监测

***sleep(1)的作用?***

```
当前线程让出CPU执行权, 其他线程可以在此时竞争
```

***runnable和callable的区别?sumbit()和execute()的区别?***

```
* runnable不能有返回值, 不能抛出受检异常
* callable可以有返回值, 可以抛出受检异常

* sumbit(), 接收runnable和callable参数, 会将两者封装为FutureTask再执行, 将在子线程内部未捕获的异常和返回值并封装为FutureTask返回, 此时未捕获的异常不会在控制台打印, 除非调用futureTask.get())

* execute(), 只能执行runnable, 会在控制台打印未捕获异常
```

***如何捕获(线程池中的)子线程中的异常?***

```
* 子线程内部捕获: try catch

* 通过线程工厂设置线程池中的线程的未捕获异常处理器: thread.setUncaughtExceptionHandler()

* 重写线程池的afterExecution(Runnable r, Throwable t), 调用((FutureTask<?>) r).get()
```

## 2. 线程通信

***Java中如何保证线程安全?***

java中使用**synchronized/lock/volatile**等机制保障原子性, 可见性, 有序性,   从而保障线程安全

***Java中多线程协作的API有哪些?***

```java
obj.wait()
obj.notify()
thread.join();
Thread.yield()
Thread.sleep()
JUC中的Condition提供的condition.await(); condition.signal();
JUC提供的同步工具类 CountDownLatch CyclicBarrier Semaphore
```

***如何实现一个生产者消费者模式的两个线程?***

使用synchronized+List+notifyAll + wait实现

## 3. Java Memory Model

***讲一下JMM?***

JMM全称Java内存模型, JMM定义了Java多线程编程下相关概念的抽象, **用于屏蔽底层硬件细节(包括重排序, 缓存模型)**

JMM定义了:

```
* 工作内存: 每个线程的私有缓存, 保存了该线程的数据副本, 对应L1/L2缓存
* 主内存: 存储线程共享的数据, 对应主存
* volatile: 保证了对变量操作的可见性
* synchornized: 保证了操作的原子性和可见性
```

## 4. Load Store Fence

***重排序分为哪两种?造成重排序的原因是什么?***

重排序分为指令重排序和内存重排序

* 指令重排序 `(又细分为编译器重排序和CPU重排序)`**是为了提高执行效率, 但不会改变程序在单线程下的语义**
* 内存重排序是由于使用了CPU缓存和缓存的读写缓冲器造成的**读写延迟** `(读写延迟具体会造成LoadLoad, StoreStore, LoadStore, StoreLoad重排序)`

***什么是Happens-Before原则?***

虽然编译器会对指令进行重排序, 但重排序遵循一些Happens-Before规则以**保障并发编程的正确性**

***内存屏障指令是什么, Java中的内存屏障指令有哪些?***

CPU提供了内存屏障指令用来阻止内存屏障指令前后的指令重排序

Java对硬件的内存屏障指令进行了抽象, 分为四种内存屏障指令:

```
LoadLoad
StoreStore
LoadStore
StoreLoad
```

## 5. synchronized

***synchronized如何使用?***

修饰静态方法时锁对象为Class对象, 修饰成员方法时锁对象为 this, 同步代码块中可以自己指定锁对象

```java
                synchronized (lock) {
                    while (curThreadId != 1) lock.wait();
                    System.out.println("thread1!");
                    lock.notifyAll();
                }
```

***synchronized的作用, 原理是什么?***

synchronized等价于在同步代码块前后添加了**内存屏障和两条指令** `monitorEnter/ monitorExit`, 这两条指令操作的是的 `对象头中的Markword的锁指针指向的ObjectMonitor`

```Java
monitorEnter
(内存屏障, 将主内存数据刷新到工作内存)
// 同步代码块
monitorExit
(内存屏障, 将工作内存的数据刷新到主内存)
```

ObjectMonitor保存了竞争锁的线程集合Entry Set, 当前持有线程和锁计数, 等待锁的线程集合Wait Set

![1692086023645](image/java并发/1692086023645.png)

***synchronized锁升级过程是怎样的?***

```
无锁 -> 偏向锁 -> 轻量级锁 -> 重量级锁
```

偏向锁: `记录锁偏向的线程, 如果获取锁时通过CAS发现锁偏向的线程是当前线程, 则不加锁, 否则升级为轻量级锁`

轻量级锁: `尝试通过CAS+自旋一定次数获取锁, 如果获取锁失败, 则升级为重量级锁`

重量级锁: `获取锁时进入操作系统内核态, 获取锁失败时直接切换线程上下文`

## 6. volatile和CAS

***volatile实现原理?***

使用了**内存屏障**保证了对volatile变量的写入, 会立刻从工作内存刷新到主内存, 对volatile变量的读取, 会先从主内存中读取到工作内存

***什么是CAS, CAS实现原理?***

CPU提供的硬件指令, 可以原子地进行compare and swap操作 `(比试结果相同时则写入, 否则不做操作)`

CAS硬件的实现原理

```
* 锁定总线: 当读取完数据后直接锁定总线, 比较完成, 写入数据时才解锁总线

* MESI缓存一致性协议: 这个协议保证了当一个核心的缓存被写入时, 其他核心的缓存会失效
```

***CAS会有什么问题, 如何解决?***

ABA问题无法确保数据在锁定期间被写入, 只能保障数据和期望的一致, **使用版本号解决ABA问题**

# 3. JUC

## 1. Lock&Condition

***Lock和Synchronized的区别有哪些?***

```
* Lock可以是公平锁,  Synchornized只能是非公平锁
* Synchornized内部有异常会自动释放锁, Lock不会
* Lock更灵活, 可以使用tryLock(), 即使获取不到锁也不会阻塞, 也可以使用Condition进行更细粒度的唤醒
```

***Condition如何使用?***

```java
Condition c = lock.newCondition();
c.await()
c.singal()
```

## 2. JUC工具类

***CountDownLatch和CyclicBarrier的区别?***

CountDownLatch: `CountDownLatch用于一个线程等待其他多个线程执行完成, 内部是一个倒数计数器 `

CyclicBarrier: `CyclicBarrier用于多个线程之间互相在同步点等待`

## 3. ThreadLocal

***讲一下ThreadLocal实现原理?***

get()方法调用时, 会获取当前线程对象的 `ThreadLocalMap` , map中的key是 `ThreadLocal` 的**弱引用**, 值是当前线程的副本值

***使用ThreadLocal会有什么问题, 如何解决?***

使用线程池中的线程时, 如果忘记 `remove()` 会造成内存泄漏

## 4. Thread Pool

***JUC Executors提供的线程池有哪些?***

```
固定大小的线程池: Executors.newFixedThreadPool
动态大小的线程池: Executors.newCachedThreadPool()
单线程的线程池: Executors.newSingleThreadExecutor()
```

***创建线程池的参数有哪些, 各自含义?***

```
* 核心线程数: 常驻的线程数量
* 最大线程数
* 空闲线程存活时间: 当线程数量大于核心线程数时, 线程空闲多长就被销毁
* 阻塞队列: 待执行的任务的阻塞队列
* 线程工厂
* 拒绝策略
```

***如何设置线程池的参数?***

核心线程数

```
* 针对 IO 密集型：线程池的核心线程数量要相对较多，约等于 CPU 核心数*2。

* 针对计算密集型：线程池的核心数量尽量要少，约等于 CPU 的核心数。
```

选用什么阻塞队列看业务

```
使用不定长的阻塞队列要确保高并发时不会OOM
```

***拒绝处理策略有哪些?***

```
直接抛出异常: AbortPolicy
直接丢弃任务: DiscardPolicy
丢弃队列中最老的任务: DiscardOldestPolicy
```

***常见的阻塞队列有哪些?***

```
基于数组的有界阻塞队列: ArrayBlockingQueue
基于链表的无界/有界阻塞队列: LinkedBlockingQueue
待优先级的阻塞队列: PriorityBlockingQueue
延迟阻塞队列: DelayQueue
```

***线程池submit后的流程?***

```
1. 如果当前线程数 < 核心线程数, 新建线程并执行任务
2. 如果当前线程数 >= 核心线程数, 将任务放入阻塞队列
3. 如果最大线程数 >= 当前线程数 >= 核心线程数并且阻塞队列已满, 新建线程执行任务
4. 如果当前线程数 > 最大线程数, 执行拒绝策略
```

## 5. AQS

***讲一下AQS的原理和核心思想?***

AQS内部维护一个 `通过双向链表实现的FIFO队列` 和一个 `volatile int 的state变量`

* 当线程获取锁时 `(acquire)` 会调用抽象方法 `tryAcquire`尝试获取锁, 获取锁时使用 `CAS` 改变state变量, 如果获取锁失败会将线程插入队列

***JUC中哪些类使用了AQS?***

```
CountDownLatch中的静态内部类Sync
ReentrantLock中的FairSync和NonFairSync
Semaphore中的Sync
```

***如何使用AQS实现独占锁和共享锁, 如何基于AQS实现自己的锁控制?***

独占锁: 继承AQS重写 `tryAcquire` 和  `tryRelease` 方法, 独占锁CAS(0, 1)

共享锁: 重写 `tryAcquireShared` 和 `tryReleaseShared` 方法CAS(state+1, state)

## 6. Concurrent Queue

***无阻塞队列ConcurrentLinkedQueue实现原理?***

***阻塞队列ArrayBlockingQueue实现原理?***
