**Lock和Condition**

并发编程主要是为了解决两大问题

- **互斥**，即同一时刻只允许一个线程访问共享资源--->>>原子性
- **同步**，即线程之间如何通信、协作--->>>可见性、有序性

Java SDK 并发包通过 Lock 和 Condition 两个接口来实现管程，其中 Lock 用于解决互斥问题，Condition 用于解决同步问题。

**管程是怎么解决互斥与同步的问题的？**

synchronized是管程的一种默认实现，待补充





**为什么还需要有Lock和Condition呢？**

synchronized无法**破坏不可抢占**条件，因为synchronized申请资源时，申请不到就会直接进入阻塞状态，这时候也无法释放自身所占有的资源。而我们希望的是当申请不到资源时能够主动释放它占有的资源。

若需要设计这样一把互斥锁来解决synchronized的问题，也就是能够**破坏不可抢占**条件，有以下三种方式：

1. **能够响应中断**：即阻塞中的线程能够响应中断信息，即能够被唤醒，这样就能释放它所占有的资源
2. **超时机制**：即线程在一段时间内没有获取到锁，不是进入阻塞状态，而是返回一个错误码
3. **非阻塞地获取锁**：若尝试获取锁失败，不进入阻塞状态，而是直接返回

Lock接口的API则用这三种方式完美解决了synchronized的问题

```java
//支持中断
void lockInterruptibly() throws InterruptedException;
//支持超时
boolean tryLock(long time, TimeUnit unit) throws InterruptedException;
//支持非阻塞
boolean tryLock();
```

那synchronized为什么一开始不设计成这样的呢

**Lock如何保证可见性？**  原子性、有序性呢？



**什么是可重入锁？**

可重入锁：线程可以重复获取同一把锁

可重入函数：多个线程可以同时调用该函数

**什么是公平锁与非公平锁？**

```java
//无参构造函数：默认非公平锁
public ReentrantLock() {
    sync = new NonfairSync();
}
//根据公平策略参数创建锁
public ReentrantLock(boolean fair){
    sync = fair ? new FairSync() 
                : new NonfairSync();
}
```

每把锁都对应着一个等待队列，当锁被释放，会去等待队列中唤醒一个线程，若是公平锁，则谁等待的时间长就唤醒谁；若是非公平锁，则随机唤醒。

**用锁的最佳实践**

- 永远只在更新对象的成员变量时加锁
- 永远只在访问可变的成员变量时加锁
- 永远不在调用其他对象的方法时加锁

**Condition**

待补充

