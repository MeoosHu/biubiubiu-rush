### 1.ThreadLocal类

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

1. newCachedThreadPool:创建一个可缓存线程池，如果线程池长度超过处理需要，可灵活回收空闲线程，若无可回收，则新建线程
2. newFixedThreadPool:创建一个定长线程池，可控制线程最大并发数，超出的线程会在队列中等待
3. newScheduledThreadPool:创建一个可定期或者延时执行任务的定长线程池，支持定时及周期性任务执行
4. newSingleThreadExecutor:创建一个单线程化的线程池，它只会用唯一的工作线程来执行任务，保证所有任务按照指定顺序(FIFO, LIFO, 优先级)执行


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