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

4. 初试容量大小和每次扩充容量大小的不同：（1）创建时如果不指定容量初始值，Hashtable默认的初试大小为11，之后每次扩充，容量变为原来的2n+1.HashMap默认的初始化大小为16。之后每次扩容，容量变为原来的2倍。（2）创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小（HashMap中的tableSizeFor（）方法保证），也就是说HashMap总是使用2的幂作为哈希表的大小（**为什么使用2的幂次方作为哈希表的大小？**）

5. 底层数据结构：JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树，以减少搜索时间。Hashtable没有这样的机制。

6. 红黑树为什么指定8为阈值：红黑树的平均查找长度为log(n)，log(8)=3，而链表的平均查找长度为n/2，当长度为8时，平均查找长度为8/2=4，才有转换为树的必要；中间的差值7是为了避免链表和树进行频繁的转换；若小于等于6，则转换为链表

   ​							**加上源码分析**

### 3.静态代理和动态代理的异同？

​		java反射：JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

### 4.String、StringBuilder、StringBuffer有什么区别？String为什么是不可变的？

​		StringBuffer线程安全

### 5.ConcurrentHashMap是如何保证线程安全的？

### 6.在自定义一个线程时它的参数列表是怎样的？

### 7.JDK对应的数组跟链表的线程安全的类库？

### 8.Collections的排序算法？----比较器等

### 9.HashMap的并发问题以及解决思路？（问题如何产生，能否用别的方法解决并发问题，用当前方法有什么优势，列举出其他集合类的线程并发问题）

### 10.JDK8的新特性

### 11.LinkedHashMap及其他的集合类

### 12.hashcode、equals、==