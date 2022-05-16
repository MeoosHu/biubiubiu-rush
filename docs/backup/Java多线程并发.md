---
typora-root-url: ..\..\picture
---

### 1.简述线程、程序、进程的基本概念，以及它们之间的关系？

​		**线程**：与进程相似，是比进程更小的执行单位。一个进程在其执行的过程中可以产生多个线程。与进程不同的是同类的多个线程在共享同一块内存空间和一组系统资源，所以系统在产生一个线程，或是在各个线程之间作切换工作时，负担要比进程小得多，正因如此，线程也被称为轻量级进程。

​		**程序**：是含有指令和数据的文件，被存储在磁盘或其他的数据存储设备中，也就是说程序是静态的代码。

​		**进程**：是程序的一次执行错恒，是系统运行程序的基本单位，因此进程是动态的。系统运行一个程序即是一个进程从创建、运行到消亡的过程。简单来说，一个进程就是一个执行中的程序，它在计算机中一个指令接着一个指令地执行着，同时，每个进程还占有某些系统资源如CPU时间，内存空间，文件，输入输出设备的使用权等。即程序在执行时，将会被操作系统载入内存中。线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上各进程是独立的，而各线程则不一定，因为同一进程中的线程极有可能会相互影响。从另一角度，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

​		**线程的基本状态：**

![](thread_status.jpg)

![](thread_status_process.jpg)

​		线程创建之后它将出于**NEW（新建）**状态，调用start()方法后开始运行，线程这时候处于**READY（可运行）**。可运行状态的线程获得了cpu时间片（timeslice）后就处于**RUNNING（运行）**状态。

​		当线程执行wait()方法之后，线程进入**WAITING（等待）**状态。进入等待状态的线程需要依靠其他线程的通知才能够返回到运行状态，而**TIME_WAITING（超时等待）**状态相当于在等待状态的基础上增加了超时限制，比如通过sleep（long millis）方法或wait（long millis）方法可以将Java线程置于**TIMED WAITING**状态。当超时时间到达后Java线程将会返回到RUNNABLE状态。当线程调用同步方法后，在没有获取到锁的情况下，线程将会进入到**BLOCKER（阻塞）**状态。线程在执行Runnable的run（）方法之后将会进入到**TERMINATED（终止）**状态。

### 2.ThreadLocal类

​		简单概述，多线程在访问共享变量时经常会出现并发问题，由此，ThreadLocal类提供了一个不需要加锁而可以规避线程不安全的方法，即提供一个线程本地变量，在实际操作的时候实际上操作的是自己本地线程中的变量

### 2.volatile关键字

​		被volatile修饰的变量能够保证每个线程能够获取该变量的最新值，从而避免脏数据。

​		Lock前缀的指令

1. 将当前处理器缓存行的数据写回系统内存；
2. 这个写回内存的操作会使得其他CPU里缓存了该内存地址的数据无效

​		假如对一个volatile变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。同时，为了保证各个处理器的缓存一致，就会实现**缓存一致性协议**，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置为无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

1. Lock前缀的指令会引起处理器缓存写回内存；
2. 一个处理器的缓存回写到内存会导致其他处理器的缓存失效；
3. 当处理器发现本地缓存失效后，就会从内存中重读该变量数据，即可以获取当前最新值。

### 3.多线程的线程池的拒绝策略

1. ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常

2. ThreadPoolExecutor.DiscardPolicy：丢弃任务，但是不抛出异常

3. ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新提交被拒绝的任务

4. ThreadPoolExecutor.CallerRunsPolicy：由调用线程（提交任务的线程）处理该任务

   线程池的默认拒绝策略为AbortPolicy

### 4.启动线程方式有哪几种，线程池有哪几种

1. 继承Thread类 （真正意义上的线程类），是Runnable接口的实现。
2. 实现Runnable接口，并重写里面的run方法
3. 应用程序可以使用Executor框架来创建线程池。Executor框架是juc里提供的线程池的实现。
4. 实现Callable接口通过FutureTask包装器来创建Thread线程

**线程池**：

1. 固定线程数的线程池

   `ExecutorService newFixedThreadPool = Executors.newFixedThreadPool(5);`

   ​         这种线程池里面的线程被设计成存放固定数量的线程，具体线程数可以考虑为**CPU核数\*N**（N可大可小，取决于并发的线程数，计算机可用的硬件资源等）。可以通过下面的代码来获取当前计算机的CPU的核数。

   ```
    int processors = Runtime.getRuntime().availableProcessors();
   ```

    FixedThreadPool 是通过 java.util.concurrent.Executors 创建的 ThreadPoolExecutor 实例。这个实例会复用 固定数量的线程处理一个共享的无边界队列 。任何时间点，最多有 nThreads 个线程会处于活动状态执行任务。如果当所有线程都是活动时，有多的任务被提交过来，那么它会一致在队列中等待直到有线程可用。如果任何线程在执行过程中因为错误而中止，新的线程会替代它的位置来执行后续的任务。所有线程都会一致存于线程池中，直到显式的执行 ExecutorService.shutdown() 关闭。由于阻塞队列使用了LinkedBlockingQueue，是一个无界队列，**因此永远不可能拒绝任务**。LinkedBlockingQueue在入队列和出队列时使用的是不同的Lock，意味着他们之间不存在互斥关系，在多CPU情况下，他们能正在在同一时刻既消费，又生产，真正做到并行。**因此这种线程池不会拒绝任务，而且不会开辟新的线程**，**也不会因为线程的长时间不使用而销毁线程**。这是典型的生产者----消费者问题，这种线程池**适合用在稳定且固定的并发场景**，比如**服务器**。

2. 缓存的线程池

   `ExecutorService newCachedThreadPool = Executors.newCachedThreadPool();`

   ​        核心池大小为0，线程池最大线程数目为最大整型，**这意味着所有的任务一提交就会加入到阻塞队列中**。当线程池中的线程60s没有执行任务就终止，阻塞队列为SynchronousQueue。SynchronousQueue的take操作需要put操作等待，put操作需要take操作等待，否则会阻塞（线程池的阻塞队列不能存储，**所以当目前线程处理忙碌状态时，所以开辟新的线程来处理请求**），线程进入wait set。总结下来：①这是一个可以无限扩大的线程池；②**适合处理执行时间比较小的任务；**③线程空闲时间超过60s就会被杀死，所以长时间处于空闲状态的时候，这种线程池几乎不占用资源；④阻塞队列没有存储空间，只要请求到来，就必须找到一条空闲线程去处理这个请求，找不到则在线程池新开辟一条线程。**如果主线程提交任务的速度远远大于CachedThreadPool的处理速度，则CachedThreadPool会不断地创建新线程来执行任务，这样有可能会导致系统耗尽CPU和内存资源，所以在使用该线程池是，一定要注意控制并发的任务数，否则创建大量的线程可能导致严重的性能问题。**

3. 单个线程的线程池

   `ExecutorService newSingleThreadExecutor = Executors.newSingleThreadExecutor();`

   SingleThreadExecutor是使用单个worker线程的Executor，作为单一worker线程的线程池，SingleThreadExecutor把corePool和maximumPoolSize均被设置为1，和FixedThreadPool一样使用的是无界队列LinkedBlockingQueue,所以带来的影响和FixedThreadPool一样。对于newSingleThreadExecutor()来说，也是**当线程运行时抛出异常的时候会有新的线程加入线程池替他完成接下来的任务**。创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，**保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行，所以这个比较适合那些需要按序执行任务的场景。**比如：一些不太重要的收尾，日志等工作可以放到单线程的线程中去执行。日志记录一般情况会比较慢（数据量大一般可能不写入数据库），顺序执行会拖慢整个接口，堆积更多请求，还可能会对数据库造成影响（事务在开启中），所以**日志记录完全可以扔到单线程的线程中去，一条条的处理**，也可以认为是一个单消费者的生产者消费者模式。

4. 固定个数的线程池

   `ExecutorService newScheduledThreadPool = Executors.newScheduledThreadPool(3);`

   相比于第一个固定个数的线程池强大在 **①可以执行延时任务，②也可以执行带有返回值的任务**

   **线程池自定义参数：**

   ```
   public ThreadPoolExecutor(int corePoolSize,
                             int maximumPoolSize,
                             long keepAliveTime,
                             TimeUnit unit,
                             BlockingQueue<Runnable> workQueue,
                             ThreadFactory threadFactory,
                             RejectedExecutionHandler handler)
   ```

- corePoolSize 线程池核心线程大小

- maximumPoolSize 线程池最大线程数量

- keepAliveTime 空闲线程存活时间

- unit 空间线程存活时间单位

- workQueue 工作队列（一个阻塞队列，用来存储等待执行的任务，这个参数的选择也很重要，会对线程池的运行过程产生重大影响，一般来说，这里的阻塞队列有以下几种选择：

  ArrayBlockingQueue;

  LinkedBlockingQueue;

  SynchronousQueue;）

- threadFactory 线程工厂

- handler 拒绝策略（表示当拒绝处理任务时的策略，默认有以下四种取值：

  ThreadPoolExecutor.AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。

  ThreadPoolExecutor.DiscardPolicy：也是丢弃任务，但是不抛出异常。

  ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）

  ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务）

### 5.关键字synchronized和volatile的比较

​			（线程安全包含**原子性**和**可见性**两个方面，Java的同步机制都是围绕这两个方面来确保线程安全的）

1. 关键字volatile是线程同步的轻量级实现，所以volatile性能肯定比synchronized要好，并且volatile只能修饰变量，而synchronized可以修饰方法，以及代码块。随着JDK新版本的发布，synchronized关键字在执行效率上得到很大提升，在开发中使用synchronized关键字的比率还是非常大的
2. 多线程访问volatile不会发生阻塞，而synchronized会发生阻塞
3. volatile能保证数据的可见性，但不能保证原子性；而synchronized可以保证原子性，也可以间接保证可见性，因为它会将私有内存和公共内存中的数据做同步
4. 关键字volatile解决的是变量在多个线程之间的可见性；而synchronized关键字解决的是多个线程之间访问资源的同步性

### 6.死锁产生的充要条件，以及规避方法

- 互斥条件
- 请求与保持条件
- 不可剥夺条件
- 循环等待条件

### 7.可重入锁

​		ReentrantLock：可重入锁，即一个线程可多次获得同一个锁，其他线程仍然多次不可获得

​		公平锁：保证等待时间最长的线程优先获得锁

​		非公平锁：随机获得锁，一般采用此方法-----可以具体分析一下java源码