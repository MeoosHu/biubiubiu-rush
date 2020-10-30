### 1.ArrayList跟LinkedList的底层实现及扩容机制

ArrayList

- 底层实现：底层使用的是Object数组

- 扩容机制

  ArrayList创建对象时，若未指定集合大小初始化大小为0；若已指定大小，集合大小为指定的大小；

  当第一次调用add方法时，集合长度变为**10**和***\*addAll内容之间\****的**较大值**；

  之后再调用add方法，**先将集合扩大1.5倍**，如果仍然不够，新长度为传入**集合大小；**

- 适用场景：适用于查询比较多的情况

LinkedList

- 底层实现：由于底层由双向链表实现，没有初始化大小，也没有扩容机制

ArrayList和LinkedList的异同

- 都是不同步的，都不能保证线程安全；vector是线程安全的
- 实现了RandomAccess接口的List，优先选择普通for循环，其次foreach；没有实现RandomAccess接口的list优先选择iterator循环

### 2.Hashmap与Hashtable

1. 线程是否安全：HashMap是非线程安全的，Hashtable是线程安全的；Hashtable内部的方法基本都经过synchronized修饰（若需保证线程安全请使用ConcurrentHashMap）

2. 效率：因为线程安全的问题，HashMap要比Hashtable效率高一点，另外，Hashtable基本被淘汰了

3. 对Null Key和Null Value的支持：HashMap中，null可以作为键，这样的键只有一个，可以有一个或多个键对应的值为null。但是在Hashtable中put进键值只要有一个null，直接抛出NullPointerException

4. 初试容量大小和每次扩充容量大小的不同：（1）创建时如果不指定容量初始值，Hashtable默认的初试大小为11，之后每次扩充，容量变为原来的2n+1.HashMap默认的初始化大小为16。之后每次扩容，容量变为原来的2倍。（2）创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小（HashMap中的tableSizeFor（）方法保证），也就是说HashMap总是使用2的幂作为哈希表的大小；以2的幂次方是为了进行hashcode计算时尽量让键对应的值在数组 均匀分布

5. 底层数据结构：JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable没有这样的机制。

6. 红黑树为什么指定8为阈值：红黑树的平均查找长度为log(n)，log(8)=3，而链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，才有转换为树的必要；中间的差值7是为了避免链表和树进行频繁的转换；若小于等于6，则转换为链表

   ​							**加上源码分析**

### 3.静态代理和动态代理的异同？

​		java反射：JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

​		静态代理：相当于在编译期就确定了被代理的类是哪一个；一般被代理类必须要实现某个接口，代理类也需要实现该接口，并且调用被代理类的方法

​		动态代理：在代码运行期间加载被代理的类，如Spring AOP机制或RPC框架；通过实现InvocationHandler接口，重写invoke方法，可以通过java反射的机制生成代理类，在通过代理类调用被代理类的方法时，最终是在invoke中执行

### 4.String、StringBuilder、StringBuffer有什么区别？String为什么是不可变的？

​		String：不可变对象，由final修饰

​		StringBuilder：线程不安全，字符串变量

​		StringBuffer：线程安全，其方法上大多加了synchronized关键字

### 5.ConcurrentHashMap是如何保证线程安全的？

​		锁分离思想；CAS锁；volatile

### 6.JDK有哪几种线程池？在自定义一个线程时它的参数列表是怎样的？

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

   ``

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

### 7.JDK对应的数组跟链表的线程安全的类库？

![img](https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1533816262839&di=a51c0622a7953f68ee7875dda2430a7c&imgtype=jpg&src=http%3A%2F%2Fimg1.imgtn.bdimg.com%2Fit%2Fu%3D3046597456%2C4072050019%26fm%3D214%26gp%3D0.jpg)

### 8.Collections的排序？

​		Arrays.sort()：主要是针对各种数据类型的数组元素排序，其中排序方法sort()实现了Comparator<T>接口，需要对其内部的比较函数compare()重写

​		Collections.sort()：主要针对集合框架中的动态数组、链表、树】哈希表等进行排序，主要也是对compare的重写，通过源码可知也是调用了Arrays.sort()

### 9.HashMap的并发问题以及解决思路？（问题如何产生，能否用别的方法解决并发问题，用当前方法有什么优势，列举出其他集合类的线程并发问题）

### 10.JDK8的新特性

1. lambad表达式

   ​		将函数本身作为参数传递给另一个函数----函数式编程

   ​		Lambda表达式是一种匿名函数

2. 函数式接口

   1. Supplier<T> 生产者：无输入，生产一个T类型的值
   2. Consumer<T> 消费者：输入一个T类型的值，无输出
   3. Function<T,R> 函数：输入一个T类型的值，返回一个R类型的值
   4. Predicate<T> 断言：输入一个T类型的值，返回true/false

3. Stream

   ​		Stream是对集合对象功能的增强，专注于对集合对象进行各种非常便利、高效的聚合操作，或者大批量数据操作

   ​		Stram API借助Lambda表达式能够进行串行、并行两种方式进行汇聚操作

   ​		步骤：获取数据源->数据转换->执行操作获取想要的结果；每次转换原有Stream对象不改变，返回一个新的Stream对象（可以多次转换），这就允许对其操作可以像链条一样排列，形成一个管道

   ![image-20200706010242960](C:\Users\Huff\AppData\Roaming\Typora\typora-user-images\image-20200706010242960.png)

### 11.hashcode、equals、==

​		hashcode()：默认继承Object类，其实也是根据内存地址值进行hash计算得出一个int值，通常来说，由于所有对象的地址都不同，所以需要重写；因为equals()比较的内容比较全面复杂，所以一般用hashcode()进行一次过滤；不过常见类型用equals已经足够

​		equals():对于数据类型，比较值，对于引用数据类型，比较内存地址；常用类都重写了equals()方法；假若不进行重写，则默认使用的就是"=="

​		==：对于数据类型，比较值，对于引用数据类型，则比较内存地址

### 12、Java IO