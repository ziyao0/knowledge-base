# Linux常用命令

查看磁盘大小

~~~sh
df -h 
~~~

查看文件夹或文件大小

~~~sh
du -h --max-depth=1 /usr
~~~

查看内存

~~~sh
free
~~~

查看cpu

~~~sh
top
~~~

字符串切割判断循环赋值脚本

~~~shell
#!/usr/bin/env bash
web_addr=$1
if [ ! ${web_addr} ]; then
    echo "Please specify args eg：sh xxx.sh 192.168.0.1 192.168.0.2 192.168.0.3"
    exit
fi
regex="\b(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[1-9])\.(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[0-9])\.(25[0-5]|2[0-4][0-9]|1[0-9][0-9]|[1-9][0-9]|[1-9])\b"
for i in "$@"; do
    echo "current upload addr ${i}"
      port=22
      ip=127.0.0.1
      user=root
      array=(${i//#/ })
      for var in ${array[@]}; do
         if [ `echo $var | egrep $regex | wc -l` -ne 0 ]; then
            ip=$var
         elif [ `expr match $var "[0-9][0-9]*$"` -gt 0 ]; then
            port=$var
         else
            user=$var
         fi
      done
done
~~~

docker创建数据库

~~~sh
docker run -d \
--restart=unless-stopped \
-e MYSQL_ROOT_PASSWORD=mysql-kuang -e TZ=Asia/Shanghai \
-v /data/mysql/docker-mysql/rancher/data:/var/lib/mysql \
-p 22031:3306 mysql:5.7 \
--max-connections=4096 \
--max-user-connections=4096 --lower-case-table-names=1 \
--sql-mode=STRICT_TRANS_TABLES,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION \
--character-set-server=utf8mb4 --collation-server=utf8mb4_general_ci
~~~

