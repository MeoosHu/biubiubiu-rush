# 设计原则

## 单一职责原则

## 依赖倒转原则

# 设计模式

## 创建型

### 单例模式

# 单例模式

> 单例模式（Single Design Pattern）：一个类只允许创建唯一一个对象（或者实例），那这个类就是一个单例类，这种设计模式就叫做单例设计模式

## 为什么要使用单例

### **1、处理资源访问冲突**

```
public class Logger {
  private FileWriter writer;
  
  public Logger() {
    File file = new File("/Users/wangzheng/log.txt");
    writer = new FileWriter(file, true); //true表示追加写入
  }
  
  public void log(String message) {
    writer.write(message);
  }
}

// Logger类的应用示例：
public class UserController {
  private Logger logger = new Logger();
  
  public void login(String username, String password) {
    // ...省略业务逻辑代码...
    logger.log(username + " logined!");
  }
}

public class OrderController {
  private Logger logger = new Logger();
  
  public void create(OrderVo order) {
    // ...省略业务逻辑代码...
    logger.log("Created an order: " + order.toString());
  }
}
```

此处FileWriter的write方法自带对象级别的锁，可这并不能解决在多线程环境下并发写入的问题。

- 采取类锁

- 分布式锁

- 并发队列（如Java中的BlockingQueue），即多个线程同时往并发线程里写入日志，一个单独的线程负责将并发队列中的数据写入到文件

而单例的处理思路则相对简单很多，并且不用频繁的创建对象，节省了内存空间和系统文件句柄。

```
public class Logger {
  private FileWriter writer;
  private static final Logger instance = new Logger();

  private Logger() {
    File file = new File("/Users/wangzheng/log.txt");
    writer = new FileWriter(file, true); //true表示追加写入
  }
  
  public static Logger getInstance() {
    return instance;
  }
  
  public void log(String message) {
    writer.write(mesasge);
  }
}

// Logger类的应用示例：
public class UserController {
  public void login(String username, String password) {
    // ...省略业务逻辑代码...
    Logger.getInstance().log(username + " logined!");
  }
}

public class OrderController {  
  public void create(OrderVo order) {
    // ...省略业务逻辑代码...
    Logger.getInstance().log("Created a order: " + order.toString());
  }
}
```

### **2、表示全局唯一类**

如配置信息类、唯一递增ID生成器

```
import java.util.concurrent.atomic.AtomicLong;
public class IdGenerator {
  // AtomicLong是一个Java并发库中提供的一个原子变量类型,
  // 它将一些线程不安全需要加锁的复合操作封装为了线程安全的原子操作，
  // 比如下面会用到的incrementAndGet().
  private AtomicLong id = new AtomicLong(0);
  private static final IdGenerator instance = new IdGenerator();
  private IdGenerator() {}
  public static IdGenerator getInstance() {
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}

// IdGenerator使用举例
long id = IdGenerator.getInstance().getId();
```

## 如何实现一个单例

设计一个单例的要点一般有以下几点：

1. 构造函数是否为private权限，避免外部通过new创造实例
2. 考虑对象创建时的线程安全问题
3. 考虑是否支持延迟加载
4. 考虑getInstance()性能是否高（是否加锁）

### **饿汉式**

```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static final IdGenerator instance = new IdGenerator();
  private IdGenerator() {}
  public static IdGenerator getInstance() {
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

- 优点：实现简单，在类加载的时候是线程安全的

- 缺点：不支持延迟加载，如果实例**占用资源多**（比如占用内存多）或**初始化耗时长**（比如需要加载各种配置文件），提前初始化实例是一种浪费资源的行为；但是对于初始化耗时长，其实更应该在程序启动时就初始化好，否则真正使用的时候会因为响应时间过长导致超时；对于实例占用资源多，按照 fail-fast 的设计原则（有问题及早暴露），那我们也希望在程序启动时就将这个实例初始化好。

### **懒汉式**

```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static IdGenerator instance;
  private IdGenerator() {}
  public static synchronized IdGenerator getInstance() {
    if (instance == null) {
      instance = new IdGenerator();
    }
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

- 优点：支持延迟加载

- 缺点：synchronized保证了线程安全，但并发度为1，相当于串行操作。若频繁使用，频繁加锁、释放锁及并发度低等问题，会导致性能瓶颈

### **双重检测**

```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private static IdGenerator instance;
  private IdGenerator() {}
  public static IdGenerator getInstance() {
    if (instance == null) {
      synchronized(IdGenerator.class) { // 此处为类级别的锁
        if (instance == null) {
          instance = new IdGenerator();
        }
      }
    }
    return instance;
  }
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

- 优点：保证了延迟加载，又支持高并发
- 缺点：因为指令重排序，可能会导致 IdGenerator 对象被 new 出来，并且赋值给 instance 之后，还没来得及初始化（执行构造函数中的代码逻辑），就被另一个线程使用了

可以给 instance 成员变量加上 **volatile** 关键字，禁止指令重排序就行了；其实高版本的 Java 已经在 JDK 内部实现中解决了这个问题（解决的方法很简单，只要把对象 new 操作和初始化操作设计为原子操作，就自然能禁止重排序）。

### **静态内部类**

类似饿汉式，但又做到了延迟加载。

```
public class IdGenerator { 
  private AtomicLong id = new AtomicLong(0);
  private IdGenerator() {}

  private static class SingletonHolder{
    private static final IdGenerator instance = new IdGenerator();
  }
  
  public static IdGenerator getInstance() {
    return SingletonHolder.instance;
  }
 
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

外部类被加载时，并不会创建IdGenerator对象，只有调用getInstance方法时，实例才会被加载。instance的唯一性，创建过程的线程安全性，都由JVM保证，既做到了延迟加载，又保证了线程安全。

### **枚举**

```
public enum IdGenerator {
  INSTANCE;
  private AtomicLong id = new AtomicLong(0);
 
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

由枚举本身的特性（每一个枚举类型极其定义的枚举变量在JVM中都是唯一的），保证了实例创建的线程安全性和实例的唯一性。

## **单例存在的问题**

### **单例对OOP特性的支持不友好**

单例这种设计模式对于其中的抽象、继承、多态都支持得不好，如：

```
public class Order {
  public void create(...) {
    //...
    long id = IdGenerator.getInstance().getId();
    //...
  }
}

public class User {
  public void create(...) {
    // ...
    long id = IdGenerator.getInstance().getId();
    //...
  }
}
```

IdGenerator的使用违背了面向接口而非实现的原则，即**违背了抽象特性**，如假如未来希望针对不同的业务实现不同的ID生成算法，则所有用到IdGenerator的地方都需要修改。

单例对继承、多态特性的支持也不友好（而非“完全不支持”），单例类也可以被继承、也可以实现多态，只是实现起来会非常奇怪，会导致代码的可读性变差。一旦你选择将某个类设计成到单例类，也就意味着放弃了继承和多态这两个强有力的面向对象特性，也就相当于损失了可以应对未来需求变化的扩展性。

### **单例会隐藏类之间的依赖关系**

单例类不需要显示创建、不需要传递参数依赖，直接调用即可。若代码较复杂，这种调用关系就会比较隐蔽，导致**可读性变差**。

### **单例对代码的扩展性不友好**

众所周知，单例类只有一个实例对象，假如有一天需要创建多个对象呢？

如数据库连接池，若有些SQL语句在执行时非常慢，我们希望将慢SQL与其他SQL隔离，即建立两个数据库连接池，这样会影响代码的**扩展性、灵活性**，实际上，开源的数据库连接池、线程池也确实没有设计成单例类。

### **单例对代码的可测试性不友好**

如果单例类依赖比较重的外部资源，比如 DB，我们在写单元测试的时候，希望能通过 mock 的方式将它替换掉。而单例类这种硬编码式的使用方式，导致无法实现 mock 替换。

如果单例类持有成员变量（比如 IdGenerator 中的 id 成员变量），那它实际上相当于一种全局变量，被所有的代码共享。如果这个全局变量是一个可变全局变量，也就是说，它的成员变量是可以被修改的，那我们在编写单元测试的时候，还需要注意不同测试用例之间，修改了单例类中的同一个成员变量的值，从而导致测试结果互相影响的问题。

### **单例不支持有参数的构造函数**

比如我们创建一个连接池的单例对象，我们没法通过参数来指定连接池的大小，可通过以下方式解决：

**第一种**

```
public class Singleton {
  private static Singleton instance = null;
  private final int paramA;
  private final int paramB;

  private Singleton(int paramA, int paramB) {
    this.paramA = paramA;
    this.paramB = paramB;
  }

  public static Singleton getInstance() {
    if (instance == null) {
       throw new RuntimeException("Run init() first.");
    }
    return instance;
  }

  public synchronized static Singleton init(int paramA, int paramB) {
    if (instance != null){
       throw new RuntimeException("Singleton has been created!");
    }
    instance = new Singleton(paramA, paramB);
    return instance;
  }
}

Singleton.init(10, 50); // 先init，再使用
Singleton singleton = Singleton.getInstance();
```

创建完实例之后，再调用 init() 函数传递参数。需要注意的是，我们在使用这个单例类的时候，要先调用 init() 方法，然后才能调用 getInstance() 方法。

**第二种**

```
public class Singleton {
  private static Singleton instance = null;
  private final int paramA;
  private final int paramB;

  private Singleton(int paramA, int paramB) {
    this.paramA = paramA;
    this.paramB = paramB;
  }

  public synchronized static Singleton getInstance(int paramA, int paramB) {
    if (instance == null) {
      instance = new Singleton(paramA, paramB);
    }
    return instance;
  }
}

Singleton singleton = Singleton.getInstance(10, 50);
```

假若我们执行两次getInstance() 方法，会导致第二次的方法结果仍然为第一次的结果。

**第三种**

```
public class Config {
  public static final int PARAM_A = 123;
  public static final int PARAM_B = 245;
}

public class Singleton {
  private static Singleton instance = null;
  private final int paramA;
  private final int paramB;

  private Singleton() {
    this.paramA = Config.PARAM_A;
    this.paramB = Config.PARAM_B;
  }

  public synchronized static Singleton getInstance() {
    if (instance == null) {
      instance = new Singleton();
    }
    return instance;
  }
}
```

将参数放到另外一个全局变量中。具体的代码实现如下。Config 是一个存储了 paramA 和 paramB 值的全局变量。里面的值既可以像下面的代码那样通过静态常量来定义，也可以从配置文件中加载得到。实际上，这种方式是最值得推荐的。

## **单例有什么替代方案**

### 静态方法

```
// 静态方法实现方式
public class IdGenerator {
  private static AtomicLong id = new AtomicLong(0);
  
  public static long getId() { 
    return id.incrementAndGet();
  }
}
// 使用举例
long id = IdGenerator.getId();
```

静态方法可以保证全局唯一，但并支持延迟加载。

### 另外一种使用单例的方式

```
// 1. 老的使用方式
public demofunction() {
  //...
  long id = IdGenerator.getInstance().getId();
  //...
}

// 2. 新的使用方式：依赖注入
public demofunction(IdGenerator idGenerator) {
  long id = idGenerator.getId();
}
// 外部调用demofunction()的时候，传入idGenerator
IdGenerator idGenerator = IdGenerator.getInsance();
demofunction(idGenerator);
```

这**只能解决单例隐藏类之间依赖关系的问题**。不过，对于单例存在的其他问题，比如对 OOP 特性、扩展性、可测性不友好等问题，还是无法解决。

**总结**

实际上，类的全局唯一性还可以通过**工厂模式**、**IOC 容器**（比如 Spring IOC 容器）来保证。

**如果单例类并没有后续扩展的需求，并且不依赖外部系统，那设计成单例类就没有太大问题**

## **如何理解单例模式中的唯一性？**

定义中提到：一个类只允许创建唯一一个对象，那么对象的唯一性的作用范围是什么呢？

单例模式创建的对象是进程唯一的。所有代码最终会编译、链接、组织在一起，形成一个可执行文件；当在执行时，操作系统会启动一个进程将其加载到内存。

## **如何实现线程唯一的单例**

利用ConcurrentHashMap，键为线程ID，值为对象，这样即不同的线程对应不同的对象，同一个线程对应于同一个对象。ThreadLocal 工具类实现更加简单，但也是基于HashMap

```
public class IdGenerator {
  private AtomicLong id = new AtomicLong(0);

  private static final ConcurrentHashMap<Long, IdGenerator> instances
          = new ConcurrentHashMap<>();

  private IdGenerator() {}

  public static IdGenerator getInstance() {
    Long currentThreadId = Thread.currentThread().getId();
    instances.putIfAbsent(currentThreadId, new IdGenerator());
    return instances.get(currentThreadId);
  }

  public long getId() {
    return id.incrementAndGet();
  }
}
```

## **如何实现集群环境下的单例？**

将这个单例对象序列化并存储到外部共享存储区（如文件），进程在使用时，都需先去文件中取出对象，使用完后再存储到外部共享存储区；

为了保证进程间，即集群环境下只有一个实例，一个进程获取到对象后，需要对对象加锁，避免其他进程再次获取；在使用完后，需要从内存中删除对象，并解锁

```
public class IdGenerator {
  private AtomicLong id = new AtomicLong(0);
  private static IdGenerator instance;
  private static SharedObjectStorage storage = FileSharedObjectStorage(/*入参省略，比如文件地址*/);
  private static DistributedLock lock = new DistributedLock();
  
  private IdGenerator() {}

  public synchronized static IdGenerator getInstance() 
    if (instance == null) {
      lock.lock();
      instance = storage.load(IdGenerator.class);
    }
    return instance;
  }
  
  public synchroinzed void freeInstance() {
    storage.save(this, IdGeneator.class);
    instance = null; //释放对象
    lock.unlock();
  }
  
  public long getId() { 
    return id.incrementAndGet();
  }
}

// IdGenerator使用举例
IdGenerator idGeneator = IdGenerator.getInstance();
long id = idGenerator.getId();
IdGenerator.freeInstance();
```

## **如何实现一个多例模式？**

“多例”指的就是一个类可以创建多个对象，但是个数是有限制的，比如只能创建 3 个对象。跟线程实现唯一单例差不多，都是通过一个map来控制对象的个数

```
public class BackendServer {
  private long serverNo;
  private String serverAddress;

  private static final int SERVER_COUNT = 3;
  private static final Map<Long, BackendServer> serverInstances = new HashMap<>();

  static {
    serverInstances.put(1L, new BackendServer(1L, "192.134.22.138:8080"));
    serverInstances.put(2L, new BackendServer(2L, "192.134.22.139:8080"));
    serverInstances.put(3L, new BackendServer(3L, "192.134.22.140:8080"));
  }

  private BackendServer(long serverNo, String serverAddress) {
    this.serverNo = serverNo;
    this.serverAddress = serverAddress;
  }

  public BackendServer getInstance(long serverNo) {
    return serverInstances.get(serverNo);
  }

  public BackendServer getRandomInstance() {
    Random r = new Random();
    int no = r.nextInt(SERVER_COUNT)+1;
    return serverInstances.get(no);
  }
}
```

这种模式跟工厂模式有点类似，但**多例模式创建的对象都是同一个类的对象，而工厂模式创建的是不同子类的对象**

## **真实案例**

- [java.lang.Runtime#getRuntime()](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime())
- [java.awt.Desktop#getDesktop()](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
- [java.lang.System#getSecurityManager()](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

### 工厂模式

### 建造者模式

### 原型模式

## 结构型

## 行为型