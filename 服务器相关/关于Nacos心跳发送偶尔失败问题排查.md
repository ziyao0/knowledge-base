# 关于Nacos心跳发送偶尔失败问题排查

## 一 问题介绍

​	不知道大家有没有遇到过nacos启动的时候不会出现错误，在启动成功之后偶尔会报心跳连接超时，出现`Socket connect timeout`，一开始没有太在意这个问题，以为只是服务器网络出现了波动，导致Nacos时常会出现超时现象。后来大范围的服务出现这种问题，导致服务状态处于不健康状态导致前端出现404，什么原因呢？

## 二 排查思路

### 1 从nacos入手排查

因为nacos是集群环境，以为是nacos某一个节点宕机，登录到节点之后返现节点状态都是`UP	`。

![image-20210812092622444](../resources/images/image-20210812092622444.png)

又到nacos的服务器查看了nameserver的日志，发现也正常，最后用了一个笨办法，起了三个服务，分别连接nacos的三个节点，服务运行了一段时间之后发现也没有出现心跳发送的问题，这时我意识到应该不是nacos本身的问题，所以着手开始从其他方面开始排查。

### 2 排查Nginx

我们nacos集群是通过nginx去负载的，众所周知nginx默认的工作连接数是`1024`，而这台nginx负载了rabbitmq集群、前端、Nacos集群，所以我怀疑是不是因为连接超过上限倒，导致多余连接被丢弃。所以我查了nginx的配置。

![image-20210812095035070](../resources/images/image-20210812095035070.png)

发现**`worker_connections`**被修改过，而我们的连接数不可能达到这个数值，所以nginx的因素也被排除到，那最后就只剩下是服务器本身的因素了。

### 3 排查服务器

大家所知linux的`open files`默认是`1024`，所以我先查了linux连接数。执行命令：

~~~sh
[root@localhost ~]# netstat -an | grep ESTABLISHED | wc -l
1237
~~~

看到有一千两百多的链接数之后我立马又查了linux默认连接数，

~~~sh
[root@localhost ~]# ulimit -a 
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15669
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15669
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
~~~

看到`open files (-n) 1024`之后，果然不出我所料，出现这种情况是linux本身默认的链接数是1024，而当先连接数已经到达1237，所以多余的连接被丢弃。所以我修改了默认最大连接数为4096。

~~~sh
sudo vim /etc/security/limits.conf
#在最后追加加上以下内容
* soft nofile 4096
* hard nofile 4096
~~~

然后重启验证

~~~sh
reboot
#验证
[root@localhost ~]# ulimit -a 
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 15669
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 4096
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 15669
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
~~~

在等待服务器重启之后，我以为这个问题终于解决了，所以到rancher上重新部署了服务，返现过段时间之后还是会出现连接超时。好吧！

在我绞尽脑汁去思考之后，还是没有想到问题的思路，最后灵机一动我去查了linux的运行日志：

~~~sh
[root@localhost ~]# tail -500f /var/log/messages
Aug 12 01:18:23 localhost kernel: net_ratelimit: 274 callbacks suppressed
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:23 localhost kernel: nf_conntrack: table full, dropping packet
Aug 12 01:18:27 localhost Keepalived_healthcheckers[1082]: TCP connection to [192.168.201.90]:22001 timeout.
~~~

看到这个日志之后我又查了两个东西

~~~sh
[root@localhost ~]# cat /proc/sys/net/netfilter/nf_conntrack_max
65535
[root@localhost ~]# cat /proc/sys/net/netfilter/nf_conntrack_count
67549
~~~

看到这两个值的时候我终于明白问题出在什么地方了，从日志里边看是linux内核本地跟踪连接的表已经达到上线了，所以后续的一些数据包被丢弃了，之后造成了TCP连接超时，所以我去修改了linux的nf_conntrack_max。

~~~sh
[root@localhost ~]# vi /etc/sysctl.conf
#在最后追加加上以下内容
net.netfilter.nf_conntrack_max = 655350
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
~~~

执行`sysctl -p`

~~~sh
[root@localhost ~]# sysctl -p
net.netfilter.nf_conntrack_max = 655350
net.netfilter.nf_conntrack_tcp_timeout_established = 1200
#查看确认是否修改成功
[root@localhost ~]# cat /proc/sys/net/netfilter/nf_conntrack_max
655350
~~~

确认之后我登录服务日志查看，果然不在出现超时问题了，这个问题到这也算是解决了。