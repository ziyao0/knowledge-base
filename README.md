# doc

#### 介绍

精品文章、技术组件安装文档、日常笔记、学习资料等。

# 更新日志

| 版本     | 更新说明                   | 日期        | 更新人 | 备注  |
|--------|------------------------|-----------|-----|-----|
| v1.0.0 | 初始化                    | 2021/4/6  | 张子尧 ||
| v1.0.1 | mysql安装                | 2021/4/6  | 张子尧 ||
| v1.0.2 | MQ搭建文档                 | 2021/4/6  | 张子尧 ||
| v1.0.3 | ELK搭建文档                | 2021/4/7  | 张子尧 ||
| v1.0.4 | zookeeper环境搭建文档        | 2021/4/7  | 张子尧 ||
| v1.0.5 | 分布式消息中间件-rabbitmq介绍及使用 | 2021/4/8  | 张子尧 ||
| v1.0.6 | linux路由相关操作            | 2021/4/12 | 张子尧 ||
| v1.1.0 | IO模型详解                 | 2021/7/6  | 张子尧 ||
| v1.1.1 | Netty从入门到精通            | 2021/7/7  | 张子尧 ||
| v1.2.0 | MySQL事务隔离级别            | 2021/3/24 | 张子尧 ||
| v1.2.1 | 深入理解MVCC               | 2021/3/25 | 张子尧 ||

# 工程结构

① Doc工程清单

| 工程名 | 描述 |
|-|-|
| ![](resources/images/direction_south.png) doc | 父模块 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) resources | 收藏数据、资源文件、附件等 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 数据结构与算法 | 数据结构与算法学习相关文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) JVM | JVM研究相关文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 源码研究 | 源码研究相关文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 分布式组件 | 分布式组件相关使用文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) Netty详解 | 分布式消息架构详解 |
| &nbsp;&nbsp;![](resources/images/direction_south.png) 微服务组件 | 微服务组件相关文档 |
| &nbsp;&nbsp;&nbsp;&nbsp;![](resources/images/direction_west.png) 注册中心 | 注册中心解决方案 |
| &nbsp;&nbsp;&nbsp;&nbsp;![](resources/images/direction_west.png) 分布式事务 | 分布式事务解决方案 |
| &nbsp;&nbsp;&nbsp;&nbsp;![](resources/images/direction_west.png) 限流容错降级 | 限流容错降级 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 组件安装 | 相关组件安装文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 服务器常用操作 | 服务器处理相关操作文档 |
| &nbsp;&nbsp;![](resources/images/direction_west.png) 工具使用 | 工具使用 |

# 目录

- [数据结构与算法](数据结构与算法)
    - [基础排序算法详解](数据结构与算法/基础排序算法详解.md)
    - [贪心算法与动态规划](数据结构与算法/贪心算法与动态规划.md)
    - [并查集算法详解](数据结构与算法/并查集算法详解.md)
    - [限流算法详解](数据结构与算法/限流算法详解.md)
    - [数论](数据结构与算法/数论.md)
    - [树](数据结构与算法/树.md)
    - [图论](数据结构与算法/图论.md)
- [JVM](JVM)
    - [JVM内存模型](JVM/JVM内存模型.md)
- [源码研究](源码研究)
    - [Spring Ioc设计与实现原理](源码研究/SpringIoc设计与实现原理.md)
- [MYSQL详解](MYSQL详解)
    - [MySQL事务隔离级别](MYSQL详解/MySQL事务隔离级别.md)
    - [深入理解MVCC](MYSQL详解/深入理解MVCC.md)
- [分布式组件](分布式组件)
    - [Redis常用的数据结构说明](分布式组件/缓存数据库/Redis常用的数据结构说明.md)
    - [分布式消息组件-Rabbitmq](分布式组件/分布式消息组件-Rabbitmq.md)
- [Netty详解](Netty详解)
    - [BIO、NIO、AIO详解](Netty详解/BIO、NIO、AIO详解.md)
    - [Netty入门](Netty详解/Netty入门.md)
    - [深入理解Netty架构](Netty详解/深入理解Netty架构.md)
- [微服务组件](微服务组件)
    - [注册中心](微服务组件/注册中心)
        - [Nacos基本使用](微服务组件/注册中心/Nacos基本使用.md)
    - [分布式事务](微服务组件/分布式事务)
        - [分布式事务解决方案](微服务组件/分布式事务/分布式事务解决方案.md)
    - [限流容错降级](微服务组件/限流容错降级)
        - [微服务限流容错降级Sentinel入门](微服务组件/限流容错降级/微服务限流容错降级Sentinel入门.md)
        - [降级规则详解](微服务组件/限流容错降级/降级规则详解.md)
        - [流量控制](微服务组件/限流容错降级/流量控制.md)
        - [热点规则详解](微服务组件/限流容错降级/热点规则详解.md)
        - [系统规则详解](微服务组件/限流容错降级/热点规则详解.md)
- [面试题汇总](面试题汇总/面试题总结.md)
- [组件安装](组件安装)
    - [ELK搭建文档](组件安装/ELK搭建文档.md)
    - [Kafka安装文档](组件安装/Kafka安装文档.md)
    - [Mysql安装文档](组件安装/Mysql安装文档.md)
    - [RabbitMq搭建文档](组件安装/RabbitMq搭建文档.md)
    - [Zookeeper环境搭建](组件安装/Zookeeper环境搭建.md)
- [服务器相关](服务器相关)
    - [KVM启动异常解决方案](服务器相关/KVM启动异常解决方案.md)
    - [防火墙与selinux](服务器相关/防火墙与selinux.md)
    - [Linux添加路由](服务器相关/Linux添加路由.md)
- [工具使用](工具使用)
    - [Git操作命令](工具使用/Git操作命令.md)
    - [Gradle中build文件详解](工具使用/Gradle/Gradle中build文件详解.md)