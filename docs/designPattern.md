# 设计原则

## 单一职责原则

> Single Responsibility Principle，缩写SRP，意为一个类或者模块只负责完成一个职责或者功能。

**如何理解单一职责原则（SRP）？**

这个原则描述的对象有两个，即类和模块。定义也很简单，即不要设计大而全的类，要设计粒度小、功能单一的类。

**如何判定一个类的职责是否够单一？**

不同的应用场景、不同阶段的需求背景下，对同一个类的职责是否单一的判定，可能都是不一样的。在某种应用场景或者当下的需求背景下，一个类的设计可能已经满足单一职责原则了，但如果换个应用场景或着在未来的某个需求背景下，可能就不满足了，需要继续拆分成粒度更细的类。

例如一个UserInfo类，单从用户角度来看，可能满足单一职责的设计；但若需要支持物流，则需将地址相关信息抽离；若需要支持统一账户，则需要将身份认证相关的信息抽离。

一般可先写一个粗粒度的类，满足业务需求，随着业务的发展，如果粗粒度的类越来越庞大，代码越来越多，这个时候就可以考虑拆分，这就是**持续重构**。下面有几条判断原则可以参考：

- 类中的代码行数、函数或属性过多，影响了代码的**可读性**和可维护性
- 类依赖的其他类过多，或依赖类的其他类过多，不符合高内聚、低耦合的设计思想
- 私有方法过多，就需考虑能否将私有方法独立到新的类中，设置为public方法，供更多的类调用，提高代码的复用性
- 给类起名字比较难，很难用业务名词概括，或者只能用一些笼统的Manager、Context，这一般说明类的职责定义不够清晰
- 类中大量方法都是集中操作类的某几个属性，这时可以考虑拆分

**类的职责是否设计得越单一越好？**

单一职责原则通过避免设计大而全的类，避免将不相关的功能耦合在一起，来提高类的内聚性。但是，如果拆分得过细，实际上会适得其反，反倒会降低内聚性，也会影响代码的可维护性。

## 依赖倒转原则

TODO

# 设计模式

## 创建型

### 单例模式

> 单例模式（Single Design Pattern）：一个类只允许创建唯一一个对象（或者实例），那这个类就是一个单例类，这种设计模式就叫做单例设计模式

**为什么要使用单例**

&emsp;&emsp;**处理资源访问冲突**

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

&emsp;&emsp;此处FileWriter的write方法自带对象级别的锁，可这并不能解决在多线程环境下并发写入的问题。

- 采取类锁

- 分布式锁

- 并发队列（如Java中的BlockingQueue），即多个线程同时往并发线程里写入日志，一个单独的线程负责将并发队列中的数据写入到文件

&emsp;&emsp;而单例的处理思路则相对简单很多，并且不用频繁的创建对象，节省了内存空间和系统文件句柄。

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

&emsp;&emsp;**表示全局唯一类**

&emsp;&emsp;如配置信息类、唯一递增ID生成器

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

**如何实现一个单例**

设计一个单例的要点一般有以下几点：

1. 构造函数是否为private权限，避免外部通过new创造实例
2. 考虑对象创建时的线程安全问题
3. 考虑是否支持延迟加载
4. 考虑getInstance()性能是否高（是否加锁）

&emsp;&emsp;**饿汉式**

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

&emsp;&emsp;**懒汉式**

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

&emsp;&emsp;**双重检测**

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

&emsp;&emsp;可以给 instance 成员变量加上 **volatile** 关键字，禁止指令重排序就行了；其实高版本的 Java 已经在 JDK 内部实现中解决了这个问题（解决的方法很简单，只要把对象 new 操作和初始化操作设计为原子操作，就自然能禁止重排序）。

&emsp;&emsp;**静态内部类**

&emsp;&emsp;类似饿汉式，但又做到了延迟加载。

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

&emsp;&emsp;外部类被加载时，并不会创建IdGenerator对象，只有调用getInstance方法时，实例才会被加载。instance的唯一性，创建过程的线程安全性，都由JVM保证，既做到了延迟加载，又保证了线程安全。

&emsp;&emsp;**枚举**

```
public enum IdGenerator {
  INSTANCE;
  private AtomicLong id = new AtomicLong(0);
 
  public long getId() { 
    return id.incrementAndGet();
  }
}
```

&emsp;&emsp;由枚举本身的特性（每一个枚举类型极其定义的枚举变量在JVM中都是唯一的），保证了实例创建的线程安全性和实例的唯一性。

**单例存在的问题**

- **单例对OOP特性的支持不友好**

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

&emsp;&emsp;单例这种设计模式对于其中的抽象、继承、多态都支持得不好，如：IdGenerator的使用违背了面向接口而非实现的原则，即**违背了抽象特性**，如假如未来希望针对不同的业务实现不同的ID生成算法，则所有用到IdGenerator的地方都需要修改。

&emsp;&emsp;单例对继承、多态特性的支持也不友好（而非“完全不支持”），单例类也可以被继承、也可以实现多态，只是实现起来会非常奇怪，会导致代码的可读性变差。一旦你选择将某个类设计成到单例类，也就意味着放弃了继承和多态这两个强有力的面向对象特性，也就相当于损失了可以应对未来需求变化的扩展性。

- **单例会隐藏类之间的依赖关系**

&emsp;&emsp;单例类不需要显示创建、不需要传递参数依赖，直接调用即可。若代码较复杂，这种调用关系就会比较隐蔽，导致**可读性变差**。

- **单例对代码的扩展性不友好**

&emsp;&emsp;众所周知，单例类只有一个实例对象，假如有一天需要创建多个对象呢？

&emsp;&emsp;如数据库连接池，若有些SQL语句在执行时非常慢，我们希望将慢SQL与其他SQL隔离，即建立两个数据库连接池，这样会影响代码的**扩展性、灵活性**，实际上，开源的数据库连接池、线程池也确实没有设计成单例类。

- **单例对代码的可测试性不友好**

&emsp;&emsp;如果单例类依赖比较重的外部资源，比如 DB，我们在写单元测试的时候，希望能通过 mock 的方式将它替换掉。而单例类这种硬编码式的使用方式，导致无法实现 mock 替换。

&emsp;&emsp;如果单例类持有成员变量（比如 IdGenerator 中的 id 成员变量），那它实际上相当于一种全局变量，被所有的代码共享。如果这个全局变量是一个可变全局变量，也就是说，它的成员变量是可以被修改的，那我们在编写单元测试的时候，还需要注意不同测试用例之间，修改了单例类中的同一个成员变量的值，从而导致测试结果互相影响的问题。

- **单例不支持有参数的构造函数**

&emsp;&emsp;比如我们创建一个连接池的单例对象，我们没法通过参数来指定连接池的大小，可通过以下方式解决：

&emsp;&emsp;**Example 1：**

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

&emsp;&emsp;**Example 2：**

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

&emsp;&emsp;**Example 3：**

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

&emsp;&emsp;将参数放到另外一个全局变量中。具体的代码实现如下。Config 是一个存储了 paramA 和 paramB 值的全局变量。里面的值既可以像下面的代码那样通过静态常量来定义，也可以从配置文件中加载得到。实际上，这种方式是最值得推荐的。

**单例有什么替代方案**

- 静态方法

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

- 另外一种使用单例的方式

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

&emsp;&emsp;这**只能解决单例隐藏类之间依赖关系的问题**。不过，对于单例存在的其他问题，比如对 OOP 特性、扩展性、可测性不友好等问题，还是无法解决。

**总结**

&emsp;&emsp;实际上，类的全局唯一性还可以通过**工厂模式**、**IOC 容器**（比如 Spring IOC 容器）来保证。

**如果单例类并没有后续扩展的需求，并且不依赖外部系统，那设计成单例类就没有太大问题**

**如何理解单例模式中的唯一性**？

定义中提到：一个类只允许创建唯一一个对象，那么对象的唯一性的作用范围是什么呢？

单例模式创建的对象是进程唯一的。所有代码最终会编译、链接、组织在一起，形成一个可执行文件；当在执行时，操作系统会启动一个进程将其加载到内存。

**如何实现线程唯一的单例**

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

**如何实现集群环境下的单例**？

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

**如何实现一个多例模式**？

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

**真实案例**

- [java.lang.Runtime#getRuntime()](http://docs.oracle.com/javase/8/docs/api/java/lang/Runtime.html#getRuntime())
- [java.awt.Desktop#getDesktop()](http://docs.oracle.com/javase/8/docs/api/java/awt/Desktop.html#getDesktop--)
- [java.lang.System#getSecurityManager()](http://docs.oracle.com/javase/8/docs/api/java/lang/System.html#getSecurityManager--)

### 工厂模式

> 

**应用场景**

当创建逻辑比较复杂，是一个“大工程”的时候，我们就考虑使用工厂模式，封装对象的创建过程，**将对象的创建和使用相分离**。

1. 代码中存在大量if-else，动态地根据不同的类型创建不同的对象。针对这种情况，我们就考虑使用工厂模式，将这一大坨 if-else 创建对象的代码抽离出来，放到工厂类中，**推荐使用简单工厂模式**
2. 单个对象本身的创建过程比较复杂，比如前面提到的要组合其他类对象，做各种初始化操作。在这种情况下，我们也可以考虑使用工厂模式，将对象的创建过程封装到工厂类中，建议使用工厂方法模式、
3. 若创建对象的逻辑并不复杂，则直接创建对象即可。

判断是否使用工厂模式的参考标准：

- 封装变化：创建逻辑有可能变化，封装成工厂类之后，创建逻辑的变更对调用者透明
- 代码复用：创建代码抽离到独立的工厂类之后可以复用
- 隔离复杂性：封装复杂的创建逻辑，调用者无需了解如何创建对象
- 控制复杂度：将创建代码抽离出来，让原本的函数或类职责更单一，代码更简洁

**简单工厂**

对于众多的if/else，可将其中的代码逻辑抽离到一个类中，这个类就被称为工厂类，大部分工厂类以Factory结尾，但也有例外，如DateFormat、Calender；另外工厂类中创建对象的方法一般以create开头，但有的也命名为 getInstance()、createInstance()、newInstance()，有的甚至命名为 valueOf()（比如 Java String 类的 valueOf() 函数）。

如案例1：

```
public class RuleConfigSource {
  public RuleConfig load(String ruleConfigFilePath) {
    String ruleConfigFileExtension = getFileExtension(ruleConfigFilePath);
    IRuleConfigParser parser = null;
    if ("json".equalsIgnoreCase(ruleConfigFileExtension)) {
      parser = new JsonRuleConfigParser();
    } else if ("xml".equalsIgnoreCase(ruleConfigFileExtension)) {
      parser = new XmlRuleConfigParser();
    } else if ("yaml".equalsIgnoreCase(ruleConfigFileExtension)) {
      parser = new YamlRuleConfigParser();
    } else if ("properties".equalsIgnoreCase(ruleConfigFileExtension)) {
      parser = new PropertiesRuleConfigParser();
    } else {
      throw new InvalidRuleConfigException(
             "Rule config file format is not supported: " + ruleConfigFilePath);
    }

    String configText = "";
    //从ruleConfigFilePath文件中读取配置文本到configText中
    RuleConfig ruleConfig = parser.parse(configText);
    return ruleConfig;
  }

  private String getFileExtension(String filePath) {
    //...解析文件名获取扩展名，比如rule.json，返回json
    return "json";
  }
}
```

若应用简单工厂的话就可以重构为

```
public class RuleConfigSource {
  public RuleConfig load(String ruleConfigFilePath) {
    String ruleConfigFileExtension = getFileExtension(ruleConfigFilePath);
    IRuleConfigParser parser = RuleConfigParserFactory.createParser(ruleConfigFileExtension);
    if (parser == null) {
      throw new InvalidRuleConfigException(
              "Rule config file format is not supported: " + ruleConfigFilePath);
    }

    String configText = "";
    //从ruleConfigFilePath文件中读取配置文本到configText中
    RuleConfig ruleConfig = parser.parse(configText);
    return ruleConfig;
  }

  private String getFileExtension(String filePath) {
    //...解析文件名获取扩展名，比如rule.json，返回json
    return "json";
  }
}

public class RuleConfigParserFactory {
  public static IRuleConfigParser createParser(String configFormat) {
    IRuleConfigParser parser = null;
    if ("json".equalsIgnoreCase(configFormat)) {
      parser = new JsonRuleConfigParser();
    } else if ("xml".equalsIgnoreCase(configFormat)) {
      parser = new XmlRuleConfigParser();
    } else if ("yaml".equalsIgnoreCase(configFormat)) {
      parser = new YamlRuleConfigParser();
    } else if ("properties".equalsIgnoreCase(configFormat)) {
      parser = new PropertiesRuleConfigParser();
    }
    return parser;
  }
}
```

在以上代码实现中，每次调用createParser时，都需要创建一个新的parser。为了节省内存和对象创建的时间，这里可以考虑将parser进行复用，可以将 parser 事先创建好缓存起来，算是**单例和简单工厂的结合**。

```
public class RuleConfigParserFactory {
  private static final Map<String, RuleConfigParser> cachedParsers = new HashMap<>();

  static {
    cachedParsers.put("json", new JsonRuleConfigParser());
    cachedParsers.put("xml", new XmlRuleConfigParser());
    cachedParsers.put("yaml", new YamlRuleConfigParser());
    cachedParsers.put("properties", new PropertiesRuleConfigParser());
  }

  public static IRuleConfigParser createParser(String configFormat) {
    if (configFormat == null || configFormat.isEmpty()) {
      return null;//返回null还是IllegalArgumentException全凭你自己说了算
    }
    IRuleConfigParser parser = cachedParsers.get(configFormat.toLowerCase());
    return parser;
  }
}
```

上述其实不大符合**开闭原则**，但若不是频繁的修改，也是能够接受的。

总结：就是根据不同类型创建不同的对象，将其代码逻辑抽象为一个工厂类，负责创建对象；为了节省内存和对象创建的时间，可将创建的对象缓存到map中，算是单例和简单工厂的结合。其实如果if/else过多，也可以考虑利用多态或其他设计模式来替代，可以提高代码的可扩展性，更加符合开闭原则；但也增加了类的个数，牺牲了代码的可读性。

**工厂方法**

在简单工厂的基础上，利用多态的特性面向接口编程，即每个实现类负责一种对象的创建，**更加符合开闭原则**。重构后的代码如下：

```
public interface IRuleConfigParserFactory {
  IRuleConfigParser createParser();
}

public class JsonRuleConfigParserFactory implements IRuleConfigParserFactory {
  @Override
  public IRuleConfigParser createParser() {
    return new JsonRuleConfigParser();
  }
}

public class XmlRuleConfigParserFactory implements IRuleConfigParserFactory {
  @Override
  public IRuleConfigParser createParser() {
    return new XmlRuleConfigParser();
  }
}

public class YamlRuleConfigParserFactory implements IRuleConfigParserFactory {
  @Override
  public IRuleConfigParser createParser() {
    return new YamlRuleConfigParser();
  }
}

public class PropertiesRuleConfigParserFactory implements IRuleConfigParserFactory {
  @Override
  public IRuleConfigParser createParser() {
    return new PropertiesRuleConfigParser();
  }
}
```

但其实代码的**创建仍然耦合在RuleConfigSource类的load()方法中**，我们可以**为工厂类再创建一个简单工厂**，

```
public class RuleConfigSource {
  public RuleConfig load(String ruleConfigFilePath) {
    String ruleConfigFileExtension = getFileExtension(ruleConfigFilePath);

    IRuleConfigParserFactory parserFactory = RuleConfigParserFactoryMap.getParserFactory(ruleConfigFileExtension);
    if (parserFactory == null) {
      throw new InvalidRuleConfigException("Rule config file format is not supported: " + ruleConfigFilePath);
    }
    IRuleConfigParser parser = parserFactory.createParser();

    String configText = "";
    //从ruleConfigFilePath文件中读取配置文本到configText中
    RuleConfig ruleConfig = parser.parse(configText);
    return ruleConfig;
  }

  private String getFileExtension(String filePath) {
    //...解析文件名获取扩展名，比如rule.json，返回json
    return "json";
  }
}

//因为工厂类只包含方法，不包含成员变量，完全可以复用，
//不需要每次都创建新的工厂类对象，所以，简单工厂模式的第二种实现思路更加合适。
public class RuleConfigParserFactoryMap { //工厂的工厂
  private static final Map<String, IRuleConfigParserFactory> cachedFactories = new HashMap<>();

  static {
    cachedFactories.put("json", new JsonRuleConfigParserFactory());
    cachedFactories.put("xml", new XmlRuleConfigParserFactory());
    cachedFactories.put("yaml", new YamlRuleConfigParserFactory());
    cachedFactories.put("properties", new PropertiesRuleConfigParserFactory());
  }

  public static IRuleConfigParserFactory getParserFactory(String type) {
    if (type == null || type.isEmpty()) {
      return null;
    }
    IRuleConfigParserFactory parserFactory = cachedFactories.get(type.toLowerCase());
    return parserFactory;
  }
}
```

其实在这个应用场景下，由于每个Factory只是做简单的创建操作，没必要设计成单独的类。简单工厂反而更加合适。

**什么时候使用工厂方法，而不是简单工厂？**

- 当代码块比较复杂，可以将它拆分成单独的函数或者类，应用简单工厂。
- 当对象的创建逻辑比较复杂，而是要组合其他类对象，做各种初始化操作，推荐使用工厂方法。想避免烦人的if/else，也可以使用工厂方法。

**真实案例**

- [java.util.Calendar](http://docs.oracle.com/javase/8/docs/api/java/util/Calendar.html#getInstance--)
- [java.util.ResourceBundle](http://docs.oracle.com/javase/8/docs/api/java/util/ResourceBundle.html#getBundle-java.lang.String-)
- [java.text.NumberFormat](http://docs.oracle.com/javase/8/docs/api/java/text/NumberFormat.html#getInstance--)
- [java.nio.charset.Charset](http://docs.oracle.com/javase/8/docs/api/java/nio/charset/Charset.html#forName-java.lang.String-)
- [java.net.URLStreamHandlerFactory](http://docs.oracle.com/javase/8/docs/api/java/net/URLStreamHandlerFactory.html#createURLStreamHandler-java.lang.String-)
- [java.util.EnumSet](https://docs.oracle.com/javase/8/docs/api/java/util/EnumSet.html#of-E-)
- [javax.xml.bind.JAXBContext](https://docs.oracle.com/javase/8/docs/api/javax/xml/bind/JAXBContext.html#createMarshaller--)



//1：用bean注入的方式

//2：用Function函数式接口来替代类，可以避免程序的臃肿

**抽象工厂**

即让一个工厂负责多种不同类型对象的创建，例如上述案例可以按配置文件格式（Json、Xml、Yaml……）来分类，也可以按照解析的对象（Rule 规则配置还是 System 系统配置）来分类：

```
针对规则配置的解析器：基于接口IRuleConfigParser
JsonRuleConfigParser
XmlRuleConfigParser
YamlRuleConfigParser
PropertiesRuleConfigParser

针对系统配置的解析器：基于接口ISystemConfigParser
JsonSystemConfigParser
XmlSystemConfigParser
YamlSystemConfigParser
PropertiesSystemConfigParser
```

针对这种场景，就需要编写8个工厂类，如若后续继续增加需求，会导致类的数量过多难以维护。我们这时就可以应用抽象工厂：

```
public interface IConfigParserFactory {
  IRuleConfigParser createRuleParser();
  ISystemConfigParser createSystemParser();
  //此处可以扩展新的parser类型，比如IBizConfigParser
}

public class JsonConfigParserFactory implements IConfigParserFactory {
  @Override
  public IRuleConfigParser createRuleParser() {
    return new JsonRuleConfigParser();
  }

  @Override
  public ISystemConfigParser createSystemParser() {
    return new JsonSystemConfigParser();
  }
}

public class XmlConfigParserFactory implements IConfigParserFactory {
  @Override
  public IRuleConfigParser createRuleParser() {
    return new XmlRuleConfigParser();
  }

  @Override
  public ISystemConfigParser createSystemParser() {
    return new XmlSystemConfigParser();
  }
}

// 省略YamlConfigParserFactory和PropertiesConfigParserFactory代码
```

**真实案例**

- [javax.xml.parsers.DocumentBuilderFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/parsers/DocumentBuilderFactory.html)
- [javax.xml.transform.TransformerFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/transform/TransformerFactory.html#newInstance--)
- [javax.xml.xpath.XPathFactory](http://docs.oracle.com/javase/8/docs/api/javax/xml/xpath/XPathFactory.html#newInstance--)

**工厂模式和DI容器有何区别？**

DI容器底层的设计思想就是基于工厂模式，其实就是一个大的工厂类，在程序启动时，根据配置创建好对象，使用时直接从容器中获得即可

DI容器的复杂性更高，负责的是整个应用中对象的创建，同时还负责配置的解析，对象生命周期的管理等

**DI容器的核心功能**

- 配置解析

  容器通过读取配置文件，根据配置文件提供的信息来创建对象

- 对象创建

  所有类的对象都放到了一个工厂类，通过反射机制，可以在程序运行中，动态的加载类、创建类

- 对象生命周期管理

  简单工厂有两种创建方式，一种是每次返回新的对象，另一种时每次都返回事先创建好的对象；如在Spring中，可配置scope属性，若为prototype则返回新创建的对象，若为singleton则返回单例对象

  配置懒加载同上，属性lazy-init表示是否需要懒加载

  对象的init-method和destroy-method还能在对象创建好和销毁前进行对象的初始化和清理工作

**如何实现一个简单的DI容器（案例待解决-----反射）**

核心逻辑包括两个部分：配置文件解析、根据配置文件通过“反射”语法来创建对象

1. 最小原型设计
2. 提供执行入口
3. 配置文件解析
4. 核心工厂类设计   

### 建造者模式

> 建造者模式（Builder Pattern）：将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

**产生背景**

在日常开发中，创建一个对象最普遍的方式是使用**new**关键字，那什么情况下这种方法就不适用了？

即随着可配置参数项的增多，若继续使用**new**创建对象，构造函数的参数列表会越来越长，影响代码的可读性和易用性；甚至可能会因为搞错顺序传进错误的参数值，从而导致隐藏bug的产生

```
public ResourcePoolConfig(String name, Integer maxTotal, Integer maxIdle, Integer minIdle，...) {
        if (StringUtils.isBlank(name)) {
            throw new IllegalArgumentException("name should not be empty.");
        }
        this.name = name;

        if (maxTotal != null) {
            if (maxTotal <= 0) {
                throw new IllegalArgumentException("maxTotal should be positive.");
            }
            this.maxTotal = maxTotal;
        }
        
        .......
}
```

**第一次改进：**

我们可以通过用set方法来进行赋值，从而替代冗长的构造函数。一般我们在**set()方法中设置选填参数**，**构造函数设置必填项**

```
public ResourcePoolConfig1(String name){  //构造函数设置必填项
        if (StringUtils.isBlank(name)){
            throw new IllegalArgumentException("name should not be empty.");
        }
        this.name = name;
    }

    public void setMaxTotal(Integer maxTotal) {  //set方法设置选填参数
        if (maxTotal<=0){
            throw new IllegalArgumentException("maxTotal should be positive.");
        }
        this.maxTotal = maxTotal;
    }
}
```

这样虽说解决了冗长的参数列表的问题，但有些问题仍然无法解决：

- 若必填参数过多，又会出现构造函数参数过多的问题；假若必填项放置set()方法处理，那校验必填参数是否填写的逻辑又无处安放。
- 若参数间有一定的依赖关系，例如由参数1、参数2、参数3，设置了一个就必须显式地设置另外两个；或者有一定的约束条件，如参数1小于参数2；校验的逻辑将无处安放。
- 若希望类对象是一个不可变对象，也就不能暴露set方法。

**建造者模式**

为了解决上述问题，建造者模式就可以用上了

```
public final class ResourcePoolConfig {

    //资源名称
    private String name;
    //最大总资源数量
    private int maxTotal;
    //最大空闲资源数量
    private int maxIdle;
    //最小空闲资源数量
    private int minIdle;

    private ResourcePoolConfig(Builder builder) {
        this.name = builder.name;
        this.maxTotal = builder.maxTotal;
        this.maxIdle = builder.maxIdle;
        this.minIdle = builder.minIdle;
    }

    /**
     * 可以Builder设计成内部类，也可将Builder设置成独立的非内部类
     */
    public static class Builder {

        private static final int DEFAULT_MAX_TOTAL = 8;
        private static final int DEFAULT_MAX_IDLE = 8;
        private static final int DEFAULT_MIN_IDLE = 0;

        private String name;
        private int maxTotal = DEFAULT_MAX_TOTAL;
        private int maxIdle = DEFAULT_MAX_IDLE;
        private int minIdle = DEFAULT_MIN_IDLE;

        public ResourcePoolConfig build() {
            //校验逻辑放至这里，包括必填项校验、依赖关系、约束关系等
            if (StringUtils.isBlank(name)) {
                throw new IllegalArgumentException("...");
            }
            if (maxIdle > maxTotal) {
                throw new IllegalArgumentException("...");
            }
            if (minIdle > maxTotal || minIdle > maxIdle) {
                throw new IllegalArgumentException("...");
            }
            return new ResourcePoolConfig(this);
        }

        public Builder setName(String name) {
            if (StringUtils.isBlank(name)) {
                throw new IllegalArgumentException("...");
            }
            this.name = name;
            return this;
        }

        public Builder setMaxTotal(int maxTotal) {
            if (maxTotal <= 0) {
                throw new IllegalArgumentException("...");
            }
            this.maxTotal = maxTotal;
            return this;
        }
        
        //...省略setMaxIdle、setMinIdle...
        
    }

    //...省略getter函数...
    
}
```

建造者模式能够将对象的创建与表示分离，即

1. 将校验逻辑放置Builder类中，通过建造者的set()方法设置建造者的变量值。
2. 在使用build()方法创建对象之前，做集中的校验，校验之后才会生成对象。
3. 所创建类的构造方法为private私有权限，且没有提供set()方法，这样创建出来的也就是不可变对象。

同时建造者模式还能避免对象处于无效状态，如

```
Rectangle r = new Rectange(); // r is invalid
r.setWidth(2); // r is invalid
r.setHeight(3); // r is valid
```

为了避免这种无效状态的存在，我们就需要使用构造函数一次性初始化好所有的成员变量。如果构造函数参数过多，我们就需要考虑使用建造者模式，先设置建造者的变量，然后再一次性地创建对象，让对象一直处于有效状态。

若不关心对象是否处于无效状态或可变的，直接使用set也行，毕竟Builder中的成员变量属于**重复性代码**。

**建造者模式会带来什么问题**

建造者模式一般用来创建具有共同点的对象，若差异过大，则不太适合，使用场景有一定限制

其次若对象内部变化复杂，可能需要很多的建造者来实现这种变化

**构建者模式与工厂模式的区别**

工厂模式:用来创建不同但是**相关类型**的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象。

建造者模式:是用来创建**一种类型**的复杂对象，通过设置不同的可选参数，“定制化”地创建不同的对象。

**真实案例**

- [java.lang.StringBuilder](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuilder.html)
- [java.nio.ByteBuffer](http://docs.oracle.com/javase/8/docs/api/java/nio/ByteBuffer.html#put-byte-) as well as similar buffers such as FloatBuffer, IntBuffer and so on.
- [java.lang.StringBuffer](http://docs.oracle.com/javase/8/docs/api/java/lang/StringBuffer.html#append-boolean-)
- All implementations of [java.lang.Appendable](http://docs.oracle.com/javase/8/docs/api/java/lang/Appendable.html)
- [Apache Camel builders](https://github.com/apache/camel/tree/0e195428ee04531be27a0b659005e3aa8d159d23/camel-core/src/main/java/org/apache/camel/builder)
- [Apache Commons Option.Builder](https://commons.apache.org/proper/commons-cli/apidocs/org/apache/commons/cli/Option.Builder.html)

### 原型模式

> 如果对象的创建成本比较大，而同一个类的不同对象之间差别不大（大部分字段都相同），在这种情况下，我们可以利用对已有对象（原型）进行复制（拷贝）的方式来创建新对象，以达到节省创建时间的目的。这种基于原型来创建对象的方式就叫做原型设计模式，简称原型模式。

对象创建成本大，例如对象中的数据需经过复杂计算、需要从RPC、网络、数据库、文件系统等慢IO中读取，即可考虑原型模式

**原型模式的实现方式：深拷贝和浅拷贝**

浅拷贝只会复制对象中基本数据类型数据和引用对象的内存地址，不会递归地复制引用对象，以及引用对象的引用对象

深拷贝得到的是一份完完全全独立的对象,实现有两种方式：

1. 递归拷贝对象、对象的引用对象及引用对象的引用对象，直到要拷贝的数据只包含基本数据类型数据，没有引用对象为止**（案例待解决）**
2. 先将对象序列化，然后再反序列化成新的对象

若要拷贝的对象是不可变对象，使用浅拷贝没问题；但对于可变对象，浅拷贝和原始对象会共享部分数据，可能导致数据被修改，此时应该使用深拷贝

## 结构型

TODO

## 行为型

TODO