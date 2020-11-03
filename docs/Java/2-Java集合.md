

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

Vector类的所有方法都是同步的，可以由两个线程安全地访问一个Vector对象，但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间。而ArrayList和LinkedList是线程不安全的。

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

### 11.集合中线程安全的类

Vector、Stack、Hashtable、java.util.concurrent包下所有的集合类

