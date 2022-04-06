# [Linux添加路由](../README.md)

# 一、环境

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|  **centos 7.6**  | linux路由操作 | 2021/4/11 | 张子尧 |

# 二、route介绍

## 1.简介	

​	linux系统的route命令用于显示和操作IP路由表（show/manipulate the IP routing table）。要实现两个不同的子网之间的通信，需要连接两个网络的路由器，或者同时位于两个网络的网关来实现。设置路由是为了解决在一个局域网中只有一个网关可以访问网络，那么就需要将这台机器的ip地址设置为linux机器的默认路由。注意直接在命令行执行route命令来添加路由，只是临时添加，不会永久保存，当网卡重启，路由就会失效。设置永久路由需要在/etc/rc.local中添加toute命令。

## 2.命令介绍

​	`route`命令是用于操作基于内核ip路由表，它的主要作用是创建一个静态路由让指定一个主机或者一个网络通过一个网络接口，如eth0。当使用"add"或者"del"参数时，路由表被修改，如果没有参数，则显示路由表当前的内容。

**命令格式：**

~~~sh
route [-f] [-p] [Command [Destination] [mask Netmask] [Gateway] [metric Metric]] [if Interface]] 
~~~

`Command `指定您想运行的命令 (Add/Change/Delete/Print)。 

`Destination `指定该路由的网络目标。 

`mask Netmask` 指定与网络目标相关的网络掩码（也被称作子网掩码）。 

`Gateway `指定网络目标定义的地址集和子网掩码可以到达的前进或下一跃点 IP 地址。 

`metric Metric `为路由指定一个整数成本值标（从 1 至 9999），当在路由表(与转发的数据包目标地址最匹配)的多个路由中进行选择时可以使用。 

`if Interface `为可以访问目标的接口指定接口索引。若要获得一个接口列表和它们相应的接口索引，使用 `route print` 命令的显示功能。可以使用十进制或十六进制值进行接口索引。

**参数介绍：**

- -c：显示更多信息
- -n：不解析域名
- -v：显示详细的处理信息
- -F：显示发送的信息
- -C：显示路由缓存
- -f：清除所有网关入口的路由表
- -p：与add命令一起是哟时使路由具有永久性
- add：添加一条新路由
- del：删除一条路由
- -net：目标地址的一个网络
- netmask：子网掩码
- gw：路由数据包通过的网关。注意：指定的网关必须能到达
- -host：目标地址的一个主机
- -metric：设置路由跳板

# 三、route使用

### 1.查看路由表

linux查看路由信息，执行命令：route -n

~~~sh
[root@dns-master ~]# route 
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    100    0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.201.0   0.0.0.0         255.255.255.0   U     100    0        0 eth0
[root@dns-master ~]# 
~~~

**说明：**

Flags为路由标志，标记当前网络节点的状态。

Flags标志说明：

U Up表示此路由当前为启动状态

H Host，表示此网关为一主机

G Gateway，表示此网关为一路由器

R Reinstate Route，使用动态路由重新初始化的路由

D Dynamically,此路由是动态性地写入

M Modified，此路由是由路由守护程序或导向器动态修改

! 表示此路由当前为关闭状态

**备注：**

`route -n `(-n 表示不解析名字,列出速度会比route 快)

### 2.常用指令

**添加网关/设置网关：**

~~~sh
[root@beijing01 ~]# route add -net 224.0.0.0 netmask 240.0.0.0 dev eth0
[root@beijing01 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.21.0.0      0.0.0.0         255.255.240.0   U     0      0        0 eth0
224.0.0.0       0.0.0.0         240.0.0.0       U     0      0        0 eth0
[root@beijing01 ~]# 
~~~

**屏蔽一条路由：**

~~~sh
[root@beijing01 ~]# route add -net 224.0.0.0 netmask 240.0.0.0 reject
[root@beijing01 ~]# route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default         gateway         0.0.0.0         UG    0      0        0 eth0
link-local      0.0.0.0         255.255.0.0     U     1002   0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
172.21.0.0      0.0.0.0         255.255.240.0   U     0      0        0 eth0
224.0.0.0       -               240.0.0.0       !     0      -        0 -
224.0.0.0       0.0.0.0         240.0.0.0       U     0      0        0 eth0
[root@beijing01 ~]# 
~~~