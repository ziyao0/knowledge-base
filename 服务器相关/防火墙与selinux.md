# [防火墙与selinux](../README.md)

- [防火墙与selinux](#[防火墙与selinux](../README.md))
    - [1 关闭防火墙](#1-关闭防火墙)
    - [2 关闭selinux](#2-关闭selinux)
    - [3 Iptables](#3-Iptables)


# 环境

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|  **centos 7.6**  | linux常见命令 | 2021/4/7 | 张子尧 |

### 1 关闭防火墙

由于contos 安装会自带防火墙`firewall`，所以第一步先禁用该防火墙。

~~~sh
#停止firewall
systemctl stop firewalld.service
#禁止开机自启动
systemctl disable firewalld.service
~~~

### 2 关闭selinux

~~~sh
[root@kiss ~]# setenforce 0
setenforce: SELinux is disabled
[root@kiss ~]# vim /etc/sysconfig/selinux
##SELINUX的值改为disable，SELINUX=disabled
~~~

### 3 Iptables

有些特定的环境需要用到防火墙，这里我们使用`iptables`作为安全组件。

~~~sh
#启用IPtables，并配置开机自启动
systemctl start iptables.service && systemctl enable iptables.service
~~~

只要安装好了iptable这个防火墙，这个防火墙的配置文件在`/etc/sysconfig/iptabls`。

例如：

~~~sh
 vi /etc/sysconfig/iptables   
 # 追加如下开启端口
-A INPUT -m state --state NEW -m tcp -p tcp --dport 80 -j ACCEPT

~~~

