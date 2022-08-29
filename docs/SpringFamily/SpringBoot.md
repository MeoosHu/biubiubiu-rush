**Spring Boot简述**



**SpringBoot自动装配实现及原理**

​		一般的SpringBoot应用程序都有一个主类：

```
@SpringBootApplication
public class SpringbootAutoConfigurationApplication {

    public static void main(String[] args) {
        SpringApplication.run(SpringbootAutoConfigurationApplication.class, args);
    }
}
```

​		由@SpringBootApplication进入可看到：

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

​		自动装配是通过@EnableAutoConfiguration来开启的

> @Enable注解主要的作用就是把相关组件的Bean装配到Ioc容器中，若以往基于JavaConfig的形式来完成Bean的装载，则必须使用@Configuration+@Bean，而@Enable本身就是这两者的组合注解

​		进去后会看到一个@Import({AutoConfigurationImportSelector.class})，在使用@Enable后，Spring会解析到@Import导入的配置类，从而根据配置类的描述来实现Bean的装配。而@AutoConfigurationPackage是把使用了该注解的类所在的包及其子包下的所有组件扫描进Spring IOC容器中。

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
```

​		而AutoConfigurationImportSelector实现了ImportSelector，ImportSelector只有一个selectImports抽象方法，返回一个String[]数组。在这个数组可以指定需要装配到Spring IOC容器中的类。即在@Import导入了一个ImportSelector的实现类之后，会把实现类中返回的Class名称都装载到IOC 容器中。

```
package org.springframework.context.annotation;

import java.util.function.Predicate;
import org.springframework.core.type.AnnotationMetadata;
import org.springframework.lang.Nullable;

public interface ImportSelector {
    String[] selectImports(AnnotationMetadata var1);

    @Nullable
    default Predicate<String> getExclusionFilter() {
        return null;
    }
}
```

​		**过程过程**：

- 通过@import（AutoConfigurationImportSelector）实现配置类的导入，但是这里并不是传统意义上的单个配置类装配
- AutoConfigurationImportSelector类实现了ImportSelector接口，重写了方法selectImports，它用于实现选择性批量配置类的装配
- 通过Spring提供的SpringFactoriesLoader机制，扫描classpath路径下的META-INF/Spring factories，读取需要实现自动装配的配置类
- 通过条件筛选的方式，把不符合条件的配置类移除，最终完成自动装配

**如何自定义一个Spring Boot Starter**



**Spring Boot启动流程**
