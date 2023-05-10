### 记录springboot3.x中spring.factories失效无法注册的原因及解决方法

#### 1 什么是spring.factories？

​ 在springboot中`@ComponentScan`注解的作用是扫描`@SpringBootApplication`所在的 Application类所在的包下所有的`@component`
注解（或拓展了`@component`的注解）标记的 bean，并注册到spring容器中。

​ 那么在springboot项目中，如果被spring管理的bean不在springboot包扫描路径下该怎么处理？

springbooot对于这种问题提供了两种解决方案:

+ 在springboot主类上使用`@Import`注解
+ 使用`spring.factories`文件

#### 2 失效原因

​ 排除程序自身原因（忘记加注解、`spring.factories`中类路径配置错误、书写错误等等），在springboot官网中从
springboot2.7开始不推荐使用`/META-INF/spring.factories`w文件，并且在springboot3将异常对`/META-INF/spring.factories`的支持。

##### 解决方案

> 对于springboot3后换了一个新的写法:
>
> > META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
>
>
>
> > 原来写法：
> > META-INF/spring.factories
>


![](../resources/images/springboot3.x中spring.factories失效无法注册的原因及解决方法.png)
