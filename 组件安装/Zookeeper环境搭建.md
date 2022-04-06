# [Zookeeper环境搭建](../README.md)

- [Zookeeper环境搭建](#[Zookeeper环境搭建](../README.md))
    - [一 搭建环境](#一-搭建环境)
      - [1 前期准备](#1-前期准备)
      - [2 下载解压资源](#2-下载解压资源)
      - [3 修改配置文件](#3-修改配置文件)
      - [4 启动](#4-启动)
      - [5 检查是否启动成功](#5-检查是否启动成功)


# 一 搭建环境

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|  **zookeeper3.7.0**  | 编写zookeeper搭建文档 | 2021/4/7 | 张子尧 |

### 1 前期准备

安装`jdk`环境，详情请查看`jdk安装文档`。

```sh
[root@kiss ~]# java -version 
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)

```

### 2 下载解压资源

```sh
wget https://downloads.apache.org/zookeeper/zookeeper-3.7.0/apache-zookeeper-3.7.0-bin.tar.gz
tar -zxvf apache-zookeeper-3.7.0-bin.tar.gz
cd zookeeper-3.7.0
```

### 3 修改配置文件

```sh
cd conf
cp zoo_sample.cfg zoo.cfg
```

`zoo.cfg`

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes. 数据持久化目录
dataDir=/data/zookeeper
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1

```

### 4 启动

~~~sh
bin/zkServer.sh start conf/zoo.cfg
~~~

### 5 检查是否启动成功

~~~sh
echo stat | nc 192.168.0.1
#连接服务器
bin/zkCli.sh -server 192.168.0.1:2181

ls /
~~~

