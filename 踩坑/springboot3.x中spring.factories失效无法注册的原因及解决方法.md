### 记录SpringBoot3.x中spring.factories失效无法注册的原因及解决方法

#### 1 什么是spring.factories？

 在SpringBoot中`@ComponentScan`注解的作用是扫描`@SpringBootApplication`所在的 Application类所在的包下所有的`@component`
注解（或拓展了`@component`的注解）标记的 bean，并注册到spring容器中。

 那么在SpringBoot项目中，如果被spring管理的bean不在SpringBoot包扫描路径下该怎么处理？

springbooot对于这种问题提供了两种解决方案:

+ 在SpringBoot主类上使用`@Import`注解
+ 使用`spring.factories`文件

#### 2 失效原因

 排除程序自身原因（忘记加注解、`spring.factories`中类路径配置错误、书写错误等等），在[SpringBoot发行说明](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.7-Release-Notes#changes-to-auto-configuration)从
SpringBoot 2.7开始不推荐使用`/META-INF/spring.factories`w文件，并且在SpringBoot 3将异常对`/META-INF/spring.factories`的支持。

**引用官网发行说明：**

> Changes to Auto-configuration
> Auto-configuration Registration
> If you have created your own auto-configurations, you should move the registration from spring.factories under the org.springframework.boot.autoconfigure.EnableAutoConfiguration key to a new file named META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports. Rather than a single comma-separate list, each line contains the fully qualified name of an auto-configuration class. See the included auto-configurations for an example.
>
> For backwards compatibility, entries in spring.factories will still be honored.
>
> New @AutoConfiguration Annotation
> A new @AutoConfiguration annotation has been introduced. It should be used to annotate top-level auto-configuration classes that are listed in the new META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports file, replacing @Configuration. Configuration classes that are nested within or imported by an @AutoConfiguration class should continue to use @Configuration as before.
>
> For your convenience, @AutoConfiguration also supports auto-configuration ordering via the after, afterNames, before and beforeNames attributes. This can be used as a replacement for @AutoConfigureAfter and @AutoConfigureBefore.

##### 解决方案

> 对于SpringBoot3后换了一个新的写法:
>
> ```sh
> META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
> ```
>
> > 原来写法：META-INF/spring.factories
>


![](../resources/images/springboot3.x中spring.factories失效无法注册的原因及解决方法.png)
