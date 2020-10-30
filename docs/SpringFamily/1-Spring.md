### 1、Spring IOC & AOP

#### IOC

​		IOC（Inspection of Control：控制反转）是一种设计思想，就是将原本在程序中手动创建对象的控制权，交由Spring框架来管理。IOC在其他语言中也有应用，并非Spring特有。IOC容器是Spring用来实现IOC的载体，IOC容器实际上就是个Map，Map中存放的是各种对象。

​		将对象之间的相互依赖关系交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。在实际项目中一个Service类可能有几百甚至上千个类作为它的底层，加入我们需要实例化这个Service，你可能要每次都要搞清楚这个Service所有底层类的构造函数，这可能会把人逼疯。如果利用IOC的话，你只需要配置好，然后在需要的地方引用就行了，这大大增加了项目的可维护性且降低了开发难度。

​		Spring时代我们一般通过xml文件来配置Bean，后来开发人员觉得xml文件来配置不太好，于是SpringBoot注解配置就慢慢开始流行起来

​		控制反转是依赖倒置原则的一种代码设计的思路，采用的方法就是依赖注入。

​		所谓依赖注入：就是把底层类作为参数传入上层类，实现上层类对下层类的“控制”。

​		**spring IOC如何实现，以及如何动态定义初始化，使用了什么设计模式**（需要解决）---Spring是如何使用IOC的

#### AOP

​		AOP（Aspect-Oriented Programming：面向切面编程）能够将那些与业务无关的，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

​		Spring AOP就是基于动态代理的，如果要代理的对象，实现了某个接口，那么Spring AOP会使用JDK Proxy，去创建代理对象，而对于没有实现接口的对象，就无法使用JDK Proxy去进行代理了，这时候Spring AOP会使用Cglib，这时候Spring AOP会使用Cglib生成一个被代理对象的子类来作为代理

​		Spring AOP属于运行时增强，AspectJ是编译时增强。前者基于代理，后者基于字节码操作。Spring AOP已经集成了AspectJ，切面少时用Spring AOP比较好，若切面太多，最好使用AspectJ。

### 2、Spring事务如何使用，以及什么情况下会失效？

1. **spring的事务注解@Transactional只能放在public修饰的方法上才起作用，如果放在其他非public**
2. **如果采用spring+spring mvc，则context:component-scan重复扫描问题可能会引起事务失败**
3. **如使用mysql且引擎是MyISAM，则事务会不起作用，原因是MyISAM不支持事务，可以改成InnoDB引擎**
4. **@Transactional 注解开启配置，必须放到listener里加载，如果放到[DispatcherServlet](https://www.baidu.com/s?wd=DispatcherServlet&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)的配置里，事务也是不起作用的**
5. **Spring团队建议在具体的类（或类的方法）上使用 @Transactional 注解**
6. **在业务代码中如果抛出RuntimeException异常，事务回滚；但是抛出Exception，事务不回滚**
7. **如果在加有事务的方法内，使用了try...catch..语句块对异常进行了捕获，而catch语句块没有throw new RuntimeExecption异常，事务也不会回滚**
8. **在类A里面有方法a 和方法b， 然后方法b上面用 @Transactional加了方法级别的事务，在方法a里面 调用了方法b， 方法b里面的事务不会生效。原因是在同一个类之中，方法互相调用，切面无效 ，而不仅仅是事务。这里事务之所以无效，是因为spring的事务是通过aop实现的**

### 3、Bean的生命周期

![img](https://pic1.zhimg.com/80/v2-baaf7d50702f6d0935820b9415ff364c_720w.jpg)

#### 1.实例化Bean

​		对于BeanFactory容器，当客户向容器请求一个尚未初始化的bean时，或初始化bean的时候需要注入另一个尚未初始化的依赖时，容器就会调用createBean进行实例化。 
对于ApplicationContext容器，当容器启动结束后，便实例化所有的bean。 
容器通过获取BeanDefinition对象中的信息进行实例化。并且这一步仅仅是简单的实例化，并未进行依赖注入。 
实例化对象被包装在BeanWrapper对象中，BeanWrapper提供了设置对象属性的接口，从而避免了使用反射机制设置属性。

#### 2.设置对象属性（依赖注入）

​		实例化后的对象被封装在BeanWrapper对象中，并且此时对象仍然是一个原生的状态，并没有进行依赖注入。 
紧接着，Spring根据BeanDefinition中的信息进行依赖注入。 
并且通过BeanWrapper提供的设置属性的接口完成依赖注入。

#### 3. 注入Aware接口

​		紧接着，Spring会检测该对象是否实现了xxxAware接口，并将相关的xxxAware实例注入给bean。

#### 4. BeanPostProcessor

​		当经过上述几个步骤后，bean对象已经被正确构造，但如果你想要对象被使用前再进行一些自定义的处理，就可以通过BeanPostProcessor接口实现。 
该接口提供了两个函数：

- postProcessBeforeInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会先于InitialzationBean执行，因此称为前置处理。 
  所有Aware接口的注入就是在这一步完成的。
- postProcessAfterInitialzation( Object bean, String beanName ) 
  当前正在初始化的bean对象会被传递进来，我们就可以对这个bean作任何处理。 
  这个函数会在InitialzationBean完成后执行，因此称为后置处理。

#### 5. InitializingBean与init-method

​		当BeanPostProcessor的前置处理完成后就会进入本阶段。 
​		InitializingBean接口只有一个函数：

- afterPropertiesSet()

  ​		这一阶段也可以在bean正式构造完成前增加我们自定义的逻辑，但它与前置处理不同，由于该函数并不会把当前bean对象传进来，因此在这一步没办法处理对象本身，只能增加一些额外的逻辑。 
  若要使用它，我们需要让bean实现该接口，并把要增加的逻辑写在该函数中。然后Spring会在前置处理完成后检测当前bean是否实现了该接口，并执行afterPropertiesSet函数。

  ​		当然，Spring为了降低对客户代码的侵入性，给bean的配置提供了init-method属性，该属性指定了在这一阶段需要执行的函数名。Spring便会在初始化阶段执行我们设置的函数。init-method本质上仍然使用了InitializingBean接口。

#### 6. DisposableBean和destroy-method

​		和init-method一样，通过给destroy-method指定函数，就可以在bean销毁前执行指定的逻辑。