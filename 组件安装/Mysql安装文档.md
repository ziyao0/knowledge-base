# [Mysql安装及常见问题处理](../README.md)
- [Mysql安装及常见问题处理](#Mysql安装及常见问题处理)
    - [搭建环境](#搭建环境)
    - [一 Mysql安装](#一-Mysql安装)
      - [1 创建用户](#1-创建用户)
      - [2 创建相应目录](#2-创建相应目录)
      - [3 修改my.cnf](#3-修改my.cnf)
      - [4 解压安装包(安装包见附件)](#4-解压安装包(安装包见附件))
      - [5 初始化](#5-初始化)
      - [6 查看密码](#6-查看密码)
      - [6 建立软连接](#7-建立软连接)
    - [二 重置密码](#二-重置密码)
        - [1  修改配置文件](#1-修改配置文件)
        - [2 重启服务](#2-重启服务)
        - [3 登录修改密码](#3-登录修改密码)
        - [4 重启服务](#4-重启服务)

# 搭建环境

| 文档版本|   版本   |  更新说明  |更新时间 | 更新人 |
| ---------|-------|-------|-------|------------ |
| v1.0.0|   **mysql5.7**  | 编写mysql搭建文档 | 2021/4/6 | 张子尧 |

## 一 Mysql安装

### 1 创建用户

创建`mysql`用户组和用户并修改权限（注：如果使用root用户安装则跳过该步骤）

```sh
groupadd mysql
useradd -r -g mysql mysql
```

### 2 创建相应目录

```sh
#创建数据存储目录
mkdir -p /data/mysql
#给mysql用户赋予该文件夹操作权限
chown mysql:mysql -R /data/mysql
```

### 3 修改my.cnf

```sh
vi /etc/my.cnf
```

`my.cnf`

```sh
[mysqld]
bind-address=0.0.0.0
port=23306
user=mysql
socket=/var/lib/mysql/mysql.sock
#symbolic-links=0
basedir=/usr/local/mysql5.7
datadir=/data/mysql
#socket=/tmp/mysql.sock
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid
character_set_server=utf8mb4
symbolic-links=0
explicit_defaults_for_timestamp=true
#skip-grant-tables
#附加参数
wait_timeout=86400
max_connections=4096
max_user_connections=4096
lower_case_table_names=1
sql_mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION
#character_set_server=utf8mb4
collation_server=utf8mb4_general_ci

[mysqld_safe]
#log-error=/var/log/mariadb/mariadb.log
pid-file=/data/mysql/mysql.pid
[client]
socket=/var/lib/mysql/mysql.sock

[mysqldump]
quick
quote-names
max_allowed_packet      = 16M

!includedir /etc/mysql/conf.d/
```

### 4 解压安装包(安装包见附件)

```sh
tar -zxvf mysql-5.7.32-linux-glibc2.12-x86_64.tar.gz
```

### 5 初始化

``` sh
#进入解压目录
cd mysql5.7
#执行初始化命令
./mysqld --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```

### 6 查看密码

~~~sh
cat /data/mysql/mysql.err
~~~

### 7 建立软连接

~~~sh
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
mkdir -p /var/lib/mysql
chmod 777 /var/lib/mysql
~~~

## 二 重置密码

### 1 修改配置文件

修改`/etc/my.cnf`

~~~sh
vim /etc/my.cnf
# 在[mysqld] 下任意地方添加 skip-grant-tables
~~~

### 2 重启服务

~~~sh
/etc/init.d/mysql restart 
~~~

### 3 登录修改密码

```sh
mysql -uroot -p原始密码
alter user 'root'@'localhost' identified by '1qaz@WSX#EDC';
#use mysql;
#update user set password=password("新密码") where user='root';
flush privileges;
quit
```

### 4 重启服务

~~~sh
/etc/init.d/mysql restart 
~~~





