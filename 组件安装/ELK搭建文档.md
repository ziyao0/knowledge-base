
# [ELK搭建文档](../README.md)

- [ELK搭建文档](#ELK搭建文档)
    - [搭建环境](#搭建环境)
    - [一 Elasticsearch安装](#一-Elasticsearch安装)
      - [1 创建普通用户](#1-创建普通用户)
      - [2 上传并解压](#2-上传并解压)
      - [3 修改配置文件](#3-修改配置文件)
      - [4 修改JVM参数](#4-修改JVM参数)
      - [5 启动服务](#5-启动服务)
    - [二 启动异常排查](#二-启动异常排查)
      - [1 普通用户打开文件最大数限制](#1-普通用户打开文件最大数限制)
      - [2 普通用户启动线程数限制](#2-普通用户启动线程数限制)
      - [3 普通用户调大虚拟内存](#3-普通用户调大虚拟内存)
    - [三 Kibana安装](#三-Kibana安装)
      - [1 解压修改配置](#1-解压修改配置)
      - [2 启动kibana](#2-启动kibana)
    - [四 安装IK分词器](#四-安装IK分词器)
      - [1 创建目录](#1-创建目录)
      - [2 上传资源并解压](#2-上传资源并解压)
      - [3 重启服务](#3-重启服务)

# 搭建环境

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|   **Elasticsearch7.6.1**  | 编写elasticsearch搭建文档 | 2021/4/6 | 张子尧 |
| v1.0.0 | **kibana-7.6.1** | 编写kibana搭建文档 | 2021/4/6 | 张子尧 |

## 一 Elasticsearch安装

### 1 创建普通用户

由于安全因素`Elasticsearch`是不能用普通用户来安装的。`必须`创建一个普通用户来进行安装造作。

~~~sh
#使用root权限创建用户
#1 创建es用户组
groupadd es
#2 创建用户 设置密码
useradd zhangsan
passwd zhangsan
#3 创建文件夹
mkdir -p /user/local/es
#4 授权
usermod -G es zhangsan
chown -R zhangsan /user/local/es/elasticsearch7.6.1
#5 设置sudo权限
#为了让普通用户有更大的操作权限，给普通用户设置sudo权限。
visudo
#在root ALL=(ALL) ALL 下添加:
zhangsan ALL=(ALL) ALL
#6 切换用户进行安装操作    设置密码 bin/elasticsearch-setup-passwords interactive
su zhangsan 
~~~

### 2 上传并解压

`注意：`以下操作都要在`zhangsan`用户下操作。

~~~sh
#相关安装包自行下载或见附件
tar -zxvf elasticsearch-7.6.1-linux-x86_64.tar.gz
~~~

### 3 修改配置文件

~~~sh
cd /usr/local/es/elasticsearch-7.6.1
mkdir -p  /usr/local/es/elasticsearch-7.6.1/data
mkdir -p  /usr/local/es/elasticsearch-7.6.1/log
#修改配置文件
vim config/elasticsearch.yml
~~~

`elasticsearch.yml`

```yaml
cluster.name: es-cluster
node.name: es1
path.data: /usr/local/es/elasticsearch-7.6.1/data
path.logs: /usr/local/es/elasticsearch-7.6.1/log
network.host: 0.0.0.0
http.port: 9200
discovery.seed_hosts: ["127.0.0.1"]#填写自己服务器名称
cluster.initial_master_nodes: ["es1"]#节点名称
bootstrap.system_call_filter: false
bootstrap.memory_lock: false
http.cors.enabled: true
http.cors.allow-origin: "*"
```

### 4 修改JVM参数

~~~sh
vim config/jvm.option
##根据自己的实际情况调整jvm的大小
-Xms2g
-Xmx2g
~~~

### 5 启动服务

~~~shell
#启动方式一
nohup /usr/local/es/elasticsearch-7.6.1/bin/elasticsearch 2>&1 &
#启动方式二
bin/elasticsearch -d
~~~

启动成功后可以用jps查看es的进程

访问`http://192.168.0.1:9200/?pretty`验证

## 二 启动异常排查

### 1 普通用户打开文件最大数限制

```wiki
问题错误信息描述：

max file descriptors [4096] for elasticsearch process likely too low, increase to at least [65536]
```

es因为需要大量的创建索引文件，需要大量的打开系统的文件，所有我们要接触linux系统当中打开文件最大数目的限制，否则es启动会报错

```sh
sudo vim /etc/security/limits.conf
```

`limits.conf`

```sh
##添加如下内容
* soft nofile 65536
* hard nofile 131072
* soft nproc 2048
* hard nproc 4094
```

### 2 普通用户启动线程数限制

```wiki
问题错误信息描述
max number of threads [1024] for user [es] likely too low, increase to at least [4096]
```

`[4096]`无法创建本地线程问题，用户最大可创建线程数太小。

```
#修改用户最大可创建线程数大小
### centos 6
sudo vim /etc/security/limits.d/90-nproc.conf
### centos 7
sudo vi /etc/security/limits.d/20-nproc.conf
```

找到`* soft nproc 1024`修改为`* soft nproc 4096`，可以根据自己硬件实际情况来修改。

### 3 普通用户调大虚拟内存

```wiki
错误信息描述：
max virtual memory areas vm.max_map_count [65530] likely too low, increase to at least [262144]
```

`[262144]`最大虚拟内存太小

~~~sh
vim /etc/sysctl.conf
#追加如下配置
vm.max_map_count=262144
##保存执行
sysctl -p
~~~

以上问题修改之后退出普通用户，断开`xshell`连接，然后重连切换`zhangsan`用户执行启动命令即可。

## 三 Kibana安装

kibana是es的一款主流的客户端，提供es图形界面客户端和代码客户端的功能。

### 1 解压修改配置

~~~sh
tar -zxvf kibana-7.6.1-linux-x86_64.tar.gz

cd kibana-7.6.1/config
#编辑配置文件
vim kibana.yml
~~~

`kibana.yml`

```yaml
server.port: 5601
#当前服务器的地址
server.host: 127.0.0.1
#es的地址 多个可以用，分隔
elasticsearch.hosts:["http://192.168.0.1:9200"] 
```

### 2 启动kibana

```sh
nohup  ./kibana &
```

访问`http://ip:5601/app/kibana`验证。

## 四 安装IK分词器

### 1 创建目录

~~~sh
mkdir -p /usr/local/es/elasticsearch-7.6.1/plugins/ik
~~~

### 2 上传资源并解压

~~~sh
cd /usr/local/es/elasticsearch-7.6.1/plugins/ik
unzip  elasticsearch-analysis-ik-7.6.1.zip 
~~~

### 3 重启服务

~~~sh
bin/elasticsearch -d
~~~





























