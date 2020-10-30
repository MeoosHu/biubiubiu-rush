### 1、SpringBoot如何实现自动配置？

​		自动配置指的是SpringBoot会自动将一些配置类的bean注册进IOC容器，我们可以直接使用@autowired、@resource等注解来使用

​		SpringBoot的自动配置实际上是依赖@Conditional来实现的，@Conditional是一个条件注解，是Spring4提供的新特性，用于根据特定条件来控制Bean的创建行为。@Conditional的实现里只有一个value属性，是一个Condition类型的数组；在Condition类中有一个方法matches，若返回true则会将标注的类注册到容器中。方法里面有两个参数，AnnotatedTypeMetadata注解元数据类，可以判断一个类是否为注解，或获取一个注解类的所有属性以及对应的值。ConditionContext则是专门为Condition服务的一个接口，可以从中获取到Spring容器的一些对象信息。当一个 Bean 被 Conditional 注解修饰时，Spring容器会对数组中所有 Condition 接口的 matches() 方法进行判断，当其中所有 Condition 接口的matches()方法都为 ture 时，才会创建 Bean

​		**自己动手实现SpringBoot自动配置**

### 2、Springboot启动时做了什么

​		SpringBoot通常有一个名为xxxxxApplication的入口类，包含了@SpringBootApplication这个核心注解，它其实是一个组合注解，可以用@Configuration、@EnableAutoConfiguration、@ConponentScan替换；

​		@Configuration:是做类似于spring xml 工作的注解 标注在类上,类似与以前的**.xml配置文件。

​		@EnableAutoConfiguration:spring boot自动配置时需要的注解,会让Spring Boot根据类路径中的jar包依赖为当前项目进行自动配置。同时,它也是一个组合注解。

		在@EnableAutoConfiguration中用了@Import注解导入EnableAutoConfigurationImportSelector类,而EnableAutoConfigurationImportSelector就是自动配置关键。

@Import:Spring4.2之前只支持导入配置类
        Spring4.2之后支持导入普通的java类,并将其声明成一个bean

@ComponentScan:告诉Spring 哪个packages 的用注解标识的类 会被spring自动扫描并且装入bean容器。

SpringBoot的自动配置:SpringBoot的一大特色就是自动配置,例如:添加了spring-boot-starter-web依赖,会自动添加Tomcat和SpringMVC的依赖,SpringBoot会对Tomcat和SpringMVC进行自动配置。
又例如:添加了spring-boot-starter-data-jpa依赖,SpringBoot会自动进行JPA相关的配置。

SpringBoot会自动扫描@SpringBootApplication所在类的同级包以及下级包的Bean(如果为JPA项目还可以扫描标注@Entity的实体类),所以建议入口类放置在最外层包下。