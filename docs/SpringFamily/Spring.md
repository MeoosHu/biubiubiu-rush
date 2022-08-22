# Spring基础概念

Spring是一个轻量级的、非侵入式的控制反转(IOC)和面向切面(AOP)的框架；而我们常说的Spring是指Spring Framework，它是很多模块的集合，如核心容器、数据访问/集成、Web、AOP、工具、消息和测试模块。比如：Core Container中的Core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。

Spring的6个特征：

- **核心技术**：依赖注入（DI），AOP，事件（events），资源，i18n，验证，数据绑定，类型转换
- **测试**：模拟对象，TestContext框架，Spring MVC测试，WebTestClient
- **数据访问**：事务，DAO支持，JDBC，ORM，编组XML
- **Web支持**：Spring MVC和Spring WebFlux Web框架
- **集成**：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存
- **语言**：Kotlin，Groovy，动态语言

## **核心模块**

- **Spring Core**：Spring核心包，是框架最基础的部分，提供IOC和依赖注入DI特性。
- **Spring Context**：Spring上下文容器，它是BeanFactory功能加强的一个子接口。
- **Spring Web**：它提供web应用开发的支持。
- **Spring MVC**：它针对web应用中MVC思想的实现。
- **Spring DAO**：提供对JDBC抽象层，简化了JDBC编码，同时，编码更具有健壮性。
- **Spring ORM**：它支持用于流行的ORM框架的整合，比如Spring+Hibernate、Spring+ibatis等
- **Spring AOP**：即面向切面编程。

## 常用注解

### **web相关**

**@Controller**

多个request请求进来如何保证线程安全

**@RestController**

**@RequestMapping**

@GetingMapping、@PostMapping、@PutMapping、@DeleteMapping

**@ResponseBody**

**@RequestBody**

**@PathVariable**

### **容器**

**@Component与@Bean**

@Bean的加载过程是怎么样的？

**@Service与@Repository**

**@Autowired与@Qualifer**

**@Comfiguration与@Value**

**@Scope**

### **AOP**

@Aspect(@After、@Before、@Around、@PointCut)

### **事务**

@Transational

## **设计模式在Spring中的应用**

### 工厂模式

Spring容器本质上是一个大工厂，使用工厂模式通过BeanFactory、ApplicationContext创建Bean对象。

### 代理模式

Spring AOP功能就是通过代理模式实现的，分为动态代理和静态代理。

### 单例模式

Spring中的Bean默认都是单例的，有利于容器对Bean的管理。

### 模板模式

Spring中JdbcTemplate、RestTemplate等以Template结尾的对数据库、网络等进行操作的模板类

### 观察者模式

Spring事件驱动模型就是观察者模式的一个经典应用。

### 适配器模式

Spring AOP的增强或通知（Advice）使用到了适配器模式、Spring MVC中也是用到了适配器模式适配Controller。

### 策略模式

Spring中有一个Resource接口，它的不同实现类，会根据不同的策略去访问资源。

# IOC

&emsp;&emsp;IOC（Inspection of Control：控制反转）是一种设计思想，即由Spring IOC容器来负责对象的生命周期和对象之间的关系。

&emsp;&emsp;DI（Dependency Injection）指的是容器在实例化对象的时候把它依赖的类注入给它。

## **Spring容器启动阶段实现原理**   

&emsp;&emsp;容器启动阶段

- 加载配置

- 分析配置信息

- 装配到BeanDefinition

- 其他后处理

  Bean实例化阶段

- 实例化对象

- 装配依赖

- 生命周期回调

- 对象其他处理

- 注册回调接口

[Spring IOC容器源码分析]: https://javadoop.com/post/spring-ioc

## 简易版Spring IOC容器实现

todo

## BeanFactory与ApplicationContext

## 循环依赖

# AOP

&emsp;&emsp;AOP（Aspect-Oriented Programming：面向切面编程）主要解决一些系统层面上的问题，如日志，事务，权限等，即将那些与业务无关，却为业务模块所共同调用的逻辑和责任封装起来，便于减少系统的重复代码，降低模块之间的耦合度，提高了可操作性和可维护性。

**核心概念**

- 切面：类是对物体特征的抽象，切面就是对横切关注点的抽象
- 连接点：被拦截到的点，因为Spring只支持方法类型的连接点，所以在Spring中连接点指的是被拦截到的方法，实际上连接点还可以是字段或者构造器
- 切点：对连接点进行拦截的定位
- 通知：所谓通知指的就是拦截到连接点之后要执行的代码，也可以称为增强
- 目标对象：代理的目标对象
- 织入：织入是将增强添加到目标类的具体连接点上的过程
  - 编译器织入：切面在目标类编译时被织入
  - 类加载期织入：切面在目标类加载到JVM时被织入。一般情况下，在织入切面时，AOP容器会为目标对象动态地创建一个代理对象。Spring AOP就是以这种方式，Spring采用运行期织入，而AspectJ采用编译期织入和类加载器织入
- 引介：引介是一种特殊的增强，可以动态地为类添加一些属性和方法

**环绕方式**

- 前置通知（@Before）
- 返回通知（@AfterReturning）
- 异常通知（@AfterThrowing）
- 后置通知（@After）
- 环绕通知（@Around）

&emsp;&emsp;多个切面的情况下，可以通过@Order指定先后顺序，数字越小，优先级越高

**手写AOP**

todo

**JDK动态代理和CGLIB代理**

&emsp;&emsp;**JDK动态代理**：一般通过实现**invocationHandler**接口，定义横切逻辑，再通过反射机制（invoke）调用目标类的代码，在此过程，可能包装luo'ji，对目标方法进行前置后置处理；**Proxy**利用**invocationHandler**动态创建一个符合目标类实现的接口实例，生成目标类的代理对象。

&emsp;&emsp;**Cglib代理**：JDK动态代理只能为接口创建代理实例，Cglib代理没有限制。Cglib代理通过使用字节码处理框架**ASM**,即通过字节码技术为一个类创建子类，并在子类中采用方法拦截的技术拦截所有父类方法的调用，顺势织入横切逻辑。

&emsp;&emsp;Cglib代理创建的动态代理对象性能较高，但消耗的时间也多。所以对于单例对象，无需频繁创建，使用Cglib合适，否则使用JDK代理。 同时，Cglib无法对final方法进行代理，因为它是采用动态创建子类的方式。

**example**  todo

**Spring AOP与AspectJ AOP**

Spring AOP属于**运行时增强**，基于动态代理实现，默认接口使用JDK动态代理，方法使用Cglib动态代理；需要依赖**IOC容器**管理，并且只能作用于Spring容器，java代码实现；性能上，由于基于动态代理，需生成代理实例，在方法调用上也会增加栈的深度，性能不如AspectJ；它主要解决企业级开发中最普遍的AOP（方法织入）。

AspectJ属于**编译时增强**，可以单独使用，也可以整合到其它框架中，需用到单独的编译器ajc；它属于**静态织入**，通过修改代码实现，在运行前就完成了织入，所以生成的类没有额外开销，一般有以下几个织入时机：

- 编译器织入：如类A使用AspectJ添加了一个属性，类B引用了它，这个场景就需要编译器的时候就进行织入，否则无法编译类B。
- 编译后织入：即生成了.class文件，或已经打成jar包了，这种情况我们需要增强处理，需用此方式。
- 类加载后织入：即在加载类的时候进行织入，要实现这个时期的织入，有几种常见的方法

具体对比如下图   todo

# Spring Bean

**Spring中bean的作用域**

- singleton：唯一bean实例，Spring中的bean默认都是单例的
- prototype：每次请求都会创建一个新的bean实例
- request：每一次Http请求都会产生一个新的bean，该bean仅在当前Http request内有效
- session：每一次Http请求都会产生一个新的bean，该bean仅在当前Http session内有效
- globalsession：全局session作用域，仅仅在基于protlet的web应用中才有意义，Spring5已经取消。

**Spring Bean的单例Bean的线程安全问题**

&emsp;&emsp;当多个线程同时操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

&emsp;&emsp;但如果这个Bean是无状态的，如Spring MVC中的Controller、Service、Dao等，线程中的操作无法对Bean中的成员变量执行查询以外的操作，那么这个单例Bean是线程安全的。

&emsp;&emsp;常见解决办法：

1. 将Bean定义为多例，但这样容器不好管理
2. 在Bean对象中尽量笔迷那定义可变的成员变量（不现实）
3. 在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐）

**@Component和@Bean的区别**

1. 作用对象不同：@Component注解作用于类，@Bean作用于方法
2. @Component通常是通过类路径扫描来自动检测以及自动装配到Spring的Bean容器中；@Bean注解通常是我们在标有该注解的方法中定义产生这个bean，@Bean告诉了Spring这是某个类的示例。
3. @Bean注解比@Component注解的自定义性更强，有些地方只能用@Bean，比如引入第三方库的类需要装配到Spring容器时。

@Bean注解使用示例：

```
@Configuration
public class AppConfig{
	@Bean
	public TransferService transferService(){
		return new TransferServiceImpl();
	}
}
```

> 只要是被@Component修饰的类或注解都可以定义@Bean，都可以注册进Spring容器里面，如@Repository、@Service、@Controller @Configuration

**Spring Bean的生命周期**

实例化->属性赋值->（前置处理）->初始化->（后置处理）->销毁

https://www.jianshu.com/p/1dec08d290c1

1. **实例化Bean**：对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

2. **设置对象属性（依赖注入）**：实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。紧接着，Spring根据BeanDefinition中的信息进行依赖注入。并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

3. **注入Aware接口**：Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

4. **BeanPostProcessor**：经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。

   该接口提供了两个函数：

   - postProcessBeforeInitialzation( Object bean, String beanName ) 当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 这个函数会先于InitialzationBean执行，因此称为**前置处理**。 所有Aware接口的注入就是在这一步完成的。
   - postProcessAfterInitialzation( Object bean, String beanName ) 
     当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
     这个函数会在InitialzationBean完成后执行，因此称为**后置处理**。

5. **InitializingBean与init-method**：当BeanPostProcessor的前置处理完成后就会进入本阶段。 InitializingBean接口只有一个函数：afterPropertiesSet()。

   ​		这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
   若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

   ​		当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

6. **DisposableBean和destroy-method**：通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。

**Spring Bean的依赖注入**

构造方法注入

属性注入

工厂方法注入（静态工厂、非静态工厂）

**自动装配及原理**

- byName
- byType
- constructor
- autodetect

# Spring事务

​		Spring事务分为编程式事务（硬编码，不推荐）和声明式事务（配置文件配置，推荐），声明式事务可用XML或注解进行配置

（1）**事务的特性**

- **原子性（Atomicity）**：事务是一个原子操作，由一系列动作组成。事务的原子性确保动作要么全部完成，要么完全不起作用。
- **一致性（Consistency）：**一旦事务完成（不管成功还是失败），系统必须确保它所建模的业务处于一致的状态，而不会是部分完成部分失败。在现实中的数据不应该被破坏。
- **隔离性（Isolation）**：可能有许多事务会同时处理相同的数据，因此每个事务都应该与其他事务隔离开来，防止数据损坏。
- **持久性（Durability）**：一旦事务完成，无论发生什么系统错误，它的结果都不应该受到影响，这样就能从任何系统崩溃中恢复过来。通常情况下，事务的结果被写到持久化存储器中。

（2）**事务传播特性**

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。
- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。
- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。
- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作。
- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。
- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。
- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作。它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。内部事务的回滚不会对外部事务造成影响。它只对DataSourceTransactionManager事务管理器生效。

（3）**事务隔离级别**

​		TransactionDefinition接口中定义了5个表示隔离级别的常量：

- TransactionDefinition.ISOLATION_DEFAULT:使用后端数据库默认的隔离级别，Mysql默认采用的REPEATABLE_READ隔离级别，Oracle默认采用的READ_COMMITTED隔离级别
- TransactionDefinition.ISOLATION_READ_UNCOMMITTED:读未提交，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**。
- TransactionDefinition.ISOLATION_READ_COMMITTED:读已提交，允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**。
- TransactionDefinition.ISOLATION_REPEATABLE_READ:对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生**。
- TransactionDefinition.ISOLATION_SERIALIZABLE:可序列化，最高的隔离级别，完全符合ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，该级别可以**防止脏读、幻读和不可重复读**，但严重影响程序性能，通常不会使用。

（4）**@Transactional（rollbackFor=Exception.class）**

​		Exception分为运行时异常和非运行时异常，当@Transactional注解作用于类上，该类的所有**public**方法都将具有该类型的事务属性，同时，在方法加此注解可以覆盖类级别的定义。如果类或者方法上加了这个注解，那么这个类里面的方法抛出异常，就会回滚，数据库里面的数据也会回滚。

​		在@Transactional注解中如果不配置rollbackFor属性，那么事务只会在遇到Rxception的时候才会回滚，加上rollbackFor=Exception.class，可以让事务在遇到非运行时异常时也回滚。

​		@Transactional注解的属性信息

- name:当在配置文件中有多个TransactionManager，可以用该属性指定选择哪个事务管理器
- propagation：事务的传播行为，默认值未REQUIRED
- isolation:事务的隔离度，默认值采用DEFAULT
- timeout:事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务
- read-only：指定事务是否为只读事务，默认为false；为了忽略那些不需要事务的方法，比如读取事务，可以设置read-only为true
- rollback-for：用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔
- no-rollback-for：抛出no-rollback-for指定的异常类型，不回滚事务

# Q&A























